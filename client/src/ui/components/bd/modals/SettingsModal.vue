/**
 * BetterDiscord Settings Modal Component
 * Copyright (c) 2015-present Jiiks/JsSucks - https://github.com/Jiiks / https://github.com/JsSucks
 * All rights reserved.
 * https://betterdiscord.net
 *
 * This source code is licensed under the MIT license found in the
 * LICENSE file in the root directory of this source tree.
*/

<template>
    <div class="bd-settings-modal" :class="{'bd-edited': changed}">
        <Modal :headerText="modal.headertext" :close="modal.close" :class="{'bd-modal-out': modal.closing}">
            <SettingsPanel :settings="configCache" :schemes="modal.schemes" :change="settingChange" slot="body" class="bd-settings-modal-body" />
            <div slot="footer" class="bd-footer-alert" :class="{'bd-active': changed, 'bd-warn': warnclose}">
                <div class="bd-footer-alert-text">Unsaved changes</div>
                <div class="bd-button bd-reset-button bd-tp" :class="{'bd-disabled': saving}" @click="resetSettings">Reset</div>
                <div class="bd-button bd-ok" :class="{'bd-disabled': saving}" @click="saveSettings">
                    <div v-if="saving" class="bd-spinner-7"></div>
                    <template v-else>Save Changes</template>
                </div>
            </div>
        </Modal>
    </div>
</template>
<script>
    // Imports
    import Vue from 'vue';
    import { Modal } from '../../common';
    import SettingsPanel from '../SettingsPanel.vue';
    import { Utils } from 'common';

    export default {
        props: ['modal'],
        data() {
            return {
                changed: false,
                warnclose: false,
                configCache: [],
                closing: false,
                saving: false
            }
        },
        components: {
            Modal,
            SettingsPanel
        },
        methods: {
            checkForChanges() {
                let changed = false;
                for (let category of this.configCache) {
                    const cat = this.modal.settings.find(c => c.category === category.category);
                    for (let setting of category.settings) {
                        if (!Utils.compare(cat.settings.find(s => s.id === setting.id).value, setting.value)) {
                            changed = true;
                            Vue.set(setting, 'changed', true);
                        } else {
                            Vue.set(setting, 'changed', false);
                        }
                    }
                }
                return changed;
            },
            settingChange(category_id, setting_id, value) {
                const category = this.configCache.find(c => c.category === category_id);
                if (!category) return;

                const setting = category.settings.find(s => s.id === setting_id);
                if (!setting) return;

                setting.value = value;

                this.changed = this.checkForChanges();
                this.$forceUpdate();
            },
            async saveSettings() {
                if (this.saving) return;
                this.saving = true;
                try {
                    await this.modal.saveSettings(this.configCache);
                    this.configCache = JSON.parse(JSON.stringify(this.modal.settings));
                    this.changed = false;
                } catch (err) {
                    // TODO Display error that settings failed to save
                    console.log(err);
                }
                this.saving = false;
            },
            resetSettings() {
                if (this.saving) return;
                this.configCache = JSON.parse(JSON.stringify(this.modal.settings));
                this.changed = false;
                this.$forceUpdate();
            }
        },
        created() {
            this.modal.beforeClose = force => {
                if (this.changed && !force) {
                    this.warnclose = true;
                    setTimeout(() => this.warnclose = false, 400);
                    throw {message: 'Settings have been changed'};
                }
            }
        },
        beforeMount() {
            this.configCache = JSON.parse(JSON.stringify(this.modal.settings));
            this.changed = this.checkForChanges();
        }
    }
</script>