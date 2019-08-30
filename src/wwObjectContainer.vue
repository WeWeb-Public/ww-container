<template>
    <div class="ww-container" :style="c_style">
        <!-- wwManager:start -->
        <div class="ww-container-tab">
            <span class="wwi wwi-article"></span>
        </div>
        <div class="ww-container-borders"></div>
        <!-- wwManager:end -->
        <wwObject class="ww-container-bg" :ww-object="wwObject.data.background" ww-category="background" ww-inside-ww-object="ww-container"></wwObject>
        <wwLayoutColumn tag="div" ww-default="ww-image" :ww-list="wwObject.data.wwObjects" class="ww-contaiter-layout" :ww-align="c_style.alignItems" @ww-add="wwAdd(wwObject.data.wwObjects, $event)" @ww-remove="wwRemove(wwObject.data.wwObjects, $event)">
            <wwObject v-for="wwObj in wwObject.data.wwObjects" :key="wwObj.uniqueId" :ww-object="wwObj" ww-inside-ww-object="ww-container"></wwObject>
        </wwLayoutColumn>
    </div>
</template>


<script>
export default {
    name: "__COMPONENT_NAME__",
    props: {
        wwObjectCtrl: Object,
        wwAttrs: Object
    },
    data() {
        return {
        };
    },
    computed: {
        wwObject() {
            return this.wwObjectCtrl.get();
        },
        c_height() {
            if (this.wwAttrs.wwInsideWwObject && this.wwAttrs.wwInsideWwObject == 'ww-grid') {
                return '';
            }

            let height = 60;
            let unit = 'px'

            let defaultHeight = '60px';

            // if (this.wwAttrs && this.wwAttrs.wwContainerDefaultHeight) {
            //     height = this.wwAttrs.wwContainerDefaultHeight;
            // }

            try {
                height = this.wwObject.style.height.get({ original: true }).v;
                unit = this.wwObject.style.height.get({ original: true }).u;
            } catch (error) {
                // height = 'auto';
                // unit = px
            }

            if (height == 'auto' || height == 0 || height == '0') {
                return defaultHeight;
            }

            if (unit == '%') {
                if (window.CSS && window.CSS.supports && window.CSS.supports('--fake-var', 0)) {
                    return 'calc(var(--vh, 1vh) * ' + height + ')'
                }
                else {
                    return height + 'vh'
                }
            }
            else {
                return height + unit;
            }
        },
        c_style() {
            const style = {};

            style.minHeight = this.c_height;
            style.alignItems = this.wwObject.style.alignItems || 'flex-start';

            return style;
        }

    },
    watch: {
    },
    methods: {
        init() {
        },

        /* wwManager:start */
        wwAdd(list, options) {
            list.splice(options.index, 0, options.wwObject);
            this.wwObjectCtrl.update(this.wwObject);
        },
        wwRemove(list, options) {
            list.splice(options.index, 1);
            this.wwObjectCtrl.update(this.wwObject);
        },
        /* wwManager:end */

        /*=============================================m_ÔÔ_m=============================================\
          UTILS
        \================================================================================================*/

        /* wwManager:start */
        async edit() {
            wwLib.wwObjectHover.setLock(this);

            let editList = {
                VERTICAL_ALIGN: {
                    separator: {
                        en: 'Configuration',
                        fr: 'Configuration'
                    },
                    title: {
                        en: 'Vertical align',
                        fr: 'Alignement vertical'
                    },
                    desc: {
                        en: 'Set content at the top, center or bottom of the container',
                        fr: 'Placer le contenu en haut, au centre ou en bas du conteneur'
                    },
                    icon: 'wwi wwi-config',
                    shortcut: 'c',
                    next: 'WWCONTAINER_EDIT'
                }
            }

            wwLib.wwPopups.addStory('WWGRID_EDIT', {
                title: {
                    en: 'Edit Grid',
                    fr: 'Editer la grille'
                },
                type: 'wwPopupEditWwObject',
                buttons: null,
                storyData: {
                    list: editList
                }
            })


            wwLib.wwPopups.addStory('WWCONTAINER_EDIT', {
                title: {
                    en: 'Edit container',
                    fr: 'Editer leconteneur'
                },
                type: 'wwPopupForm',
                storyData: {
                    fields: [
                        {
                            label: {
                                en: 'Vertical align :',
                                fr: 'Alignement vertical'
                            },
                            desc: {
                                en: 'Choose if content must stick to the top, the middle or the bottom of the container',
                                fr: 'Choisir si le contenu doit être en haut, au milieu ou en bas du conteneur.'
                            },
                            type: 'select',
                            key: 'alignItems',
                            valueData: 'wwObject.style.alignItems',
                            options: {
                                wwObject: {},
                                values: [
                                    {
                                        value: 'flex-start',
                                        default: true,
                                        text: {
                                            en: 'Top',
                                            fr: 'Haut'
                                        }
                                    },
                                    {
                                        value: 'center',
                                        text: {
                                            en: 'Middle',
                                            fr: 'Milieu'
                                        }
                                    },
                                    {
                                        value: 'flex-end',
                                        text: {
                                            en: 'Bottom',
                                            fr: 'Bas'
                                        }
                                    }
                                ]
                            }
                        }
                    ]
                },
                buttons: {
                    OK: {
                        text: {
                            en: 'Ok',
                            fr: 'Ok'
                        }
                    }
                }
            });

            let options = {
                firstPage: 'WWGRID_EDIT',
                data: {
                    wwObject: this.wwObject
                }
            }

            try {
                const result = await wwLib.wwPopups.open(options);

                console.log(result.alignItems);
                /*=============================================m_ÔÔ_m=============================================\
                  STYLE
                \================================================================================================*/
                if (typeof (result.alignItems) != 'undefined') {
                    //this.wwObject.style = this.wwObject.style || {};
                    this.wwObject.data._s.alignItems = result.alignItems;
                }

                // console.log(this.wwObject);

                this.wwObjectCtrl.update(this.wwObject);

                this.wwObjectCtrl.globalEdit(result);
            } catch (error) {
                console.log(error);
            }

            wwLib.wwObjectHover.removeLock();
        },
        getMinHeight() {
            return this.wwObject.style.height ? this.wwObject.style.height.value : new wwLib.wwTypes.ValueUnit('60px');
        },
        setMinHeight(value, screens) {
            if (!this.wwObject.style.height) {
                this.wwObject.style.height = new wwLib.wwTypes.ResponsiveDevice(new wwLib.wwTypes.ValueUnit('60px'));
            }

            for (let screen of screens) {
                const oldValue = this.wwObject.style.height.get({ screen });
                const newValue = new wwLib.wwTypes.ValueUnit('60px')
                newValue.value = value;
                this.wwObject.style.height.set(newValue, { screen });
            }

            this.wwObjectCtrl.update(this.wwObject);
        }
        /* wwManager:end */
    },
    created() {
        if (!this.wwObject.data.background) {
            this.wwObject.data.background = wwLib.wwObject.getDefault({ type: 'ww-color' });
            this.wwObjectCtrl.update(this.wwObject);
        }
    },
    mounted() {
        this.init();
        this.$emit('ww-loaded', this);
    }
};
</script>

<style scoped lang="scss">
.ww-container {
    display: flex;
    position: relative;
    //min-height: 20px;

    .ww-container-bg {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
    }

    .ww-contaiter-layout {
        width: 100%;
        position: relative;
        pointer-events: none;
    }
}
/* wwManager:start */
.ww-container-borders {
    display: none;
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    border: 1px solid #03a9f457;
}
.ww-editing .ww-container-borders {
    display: block;
}

.ww-container-tab {
    display: none;
    position: absolute;
    top: 0;
    left: 0;
    border-radius: 0 0 20px 20px;
    background-color: #d02e7c;
    z-index: 51;
    color: white;
    height: 20px;
    width: 20px;
    justify-content: center;
    align-items: center;
    font-size: 12px;
    pointer-events: all;
}
.ww-editing .ww-container-tab {
    display: flex;
}
/* wwManager:end */
</style>

<style>
/* wwManager:start */
.ww-container-hover {
    background-color: #2ec6ba30;
    background: repeating-linear-gradient(
        -45deg,
        #2ec6ba30,
        #2ec6ba30 10px,
        #2ec6ba50 10px,
        #2ec6ba50 11px
    );
    border-width: 5px !important;
}
/* wwManager:end */
</style>