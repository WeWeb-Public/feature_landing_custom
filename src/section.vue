<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div :style="customStyle">
        <!-- wwManager:start -->
        <wwSectionEditMenu :sectionCtrl="sectionCtrl" :options="openOptions"></wwSectionEditMenu>
        <!-- wwManager:end -->
        <wwObject class="background" :ww-object="section.data.bg" ww-category="background"></wwObject>

        <wwLayoutColumn tag="div" ww-default="ww-text" :ww-list="section.data.titles" class="content feature-title" @ww-add="add(section.data.titles, $event)" @ww-remove="remove(section.data.titles, $event)">
            <wwObject tag="div" v-for="title in section.data.titles" :key="title.uniqueId" :ww-object="title"></wwObject>
        </wwLayoutColumn>
        <div class="feature-landing">
            <div class="content-wrapper">
                <div class="feature-content">
                    <div v-for="feature in section.data.rootFeatures" :key="feature.uniqueId">
                        <transition name="fade">
                            <div v-if="feature.uniqueId === activeFeature.uniqueId ">
                                <wwObject :ww-object="feature.content"></wwObject>
                            </div>
                        </transition>
                    </div>
                </div>

                <div class="feature-title-wrapper">
                    <div class="feature-title">
                        <div class="title-wrapper" v-for="(feature, index) in section.data.rootFeatures" :key="feature.uniqueId">
                            <!-- wwManager:start -->
                            <wwContextMenu v-if="editMode" class="ww-orange-button" tag="div" :options="elemOptions" @remove="removeFeature(index)" @addBefore="addFeature(index, 'before')" @addAfter="addFeature(index, 'after')">
                                <wwOrangeButton></wwOrangeButton>
                            </wwContextMenu>
                            <!-- wwManager:end -->
                            <div class="title" :class="{ selected: feature.uniqueId === activeFeature.uniqueId }">
                                <wwObject tag="div" :ww-object="feature.title" @click="selectActiveFeature(feature)"></wwObject>
                            </div>
                        </div>
                    </div>
                    <div class="see-doc">
                        <wwObject tag="div" :ww-object="section.data.seeDoc"></wwObject>
                    </div>
                </div>
            </div>

            <div class="mobile-content-wrapper">
                <div class="mobile-content">
                    <div class="feature-wrapper" v-for="(feature, index) in section.data.rootFeatures" :key="feature.uniqueId">
                        <!-- wwManager:start -->
                        <wwContextMenu v-if="editMode" class="ww-orange-button" tag="div" :options="elemOptions" @remove="removeFeature(index)" @addBefore="addFeature(index, 'before')" @addAfter="addFeature(index, 'after')">
                            <wwOrangeButton></wwOrangeButton>
                        </wwContextMenu>
                        <!-- wwManager:end -->
                        <div class="feature-title">
                            <wwObject tag="div" :ww-object="feature.title"></wwObject>
                        </div>

                        <div class="feature-content">
                            <wwObject tag="div" :ww-object="feature.content"></wwObject>
                        </div>
                    </div>
                </div>
                <div class="see-doc">
                    <wwObject tag="div" :ww-object="section.data.seeDoc"></wwObject>
                </div>
            </div>
        </div>
        <!--BOTTOM WWOBJS-->
        <div class="bottom-ww-objs">
            <wwLayoutColumn tag="div" ww-default="ww-image" :ww-list="section.data.bottomWwObjs" class="top-ww-obj" @ww-add="add(section.data.bottomWwObjs, $event)" @ww-remove="remove(section.data.bottomWwObjs, $event)">
                <wwObject v-for="bottomWwObj in section.data.bottomWwObjs" :key="bottomWwObj.uniqueId" :ww-object="bottomWwObj"></wwObject>
            </wwLayoutColumn>
        </div>
    </div>
</template>

<!-- This is your Javascript -->
<!-- ✨ Here comes the magic ✨ -->
<script>

export default {
    name: "__COMPONENT_NAME__",
    props: {
        // The section controller object is passed to you.
        sectionCtrl: Object
    },
    data() {
        return {
            activeFeature: {},

            shadowValue: {},
            selectedFeatureIndex: 0,
            currentIndex: 0,
            elemOptions: {
                items: [
                    {
                        text: {
                            en: 'Before',
                            fr: 'Avant'
                        },
                        icon: 'wwi wwi-add',
                        action: 'addBefore'
                    },
                    {
                        text: {
                            en: 'After',
                            fr: 'Apres'
                        },
                        icon: 'wwi wwi-add',
                        action: 'addAfter'
                    },
                    {
                        text: {
                            en: 'Delete',
                            fr: 'Supprimer'
                        },
                        icon: 'wwi wwi-delete',
                        action: 'remove'
                    }
                ]
            }
        }
    },
    computed: {
        //Get the section object here !
        // It contains all the info and data about the section
        // Use it has you like !
        section() {
            return this.sectionCtrl.get();
        },
        editMode() {
            return this.sectionCtrl.getEditMode() == 'CONTENT'
        },

        customStyle() {
            return {
                '--shadowConfig': this.section.data.shadowConfig,
                '--borderColor': this.section.data.borderColor,
            }
        }


    },
    created() {

        let needUpdate = false

        //Initialize section data
        this.section.data = this.section.data || {};


        this.section.data.shadowValue = this.section.data.shadowValue || {
            x: 0, y: 0, s: 0, b: 10, c: 'rgba(50, 50, 50, 0.75)'
        }
        this.section.data.shadowConfig =
            this.section.data.shadowConfig || '0 1px 23px -5px rgba(0, 0, 0, 0.2)';

        this.section.data.borderColor =
            this.section.data.borderColor || '#2E85C2';

        if (!this.section.data.bg) {
            this.section.data.bg = wwLib.wwObject.getDefault({
                type: 'ww-color'
            });
            needUpdate = true
        }
        if (!this.section.data.seeDoc) {
            this.section.data.seeDoc = wwLib.wwObject.getDefault({ type: 'ww-text' });
            needUpdate = true
        }

        if (!this.section.data.titles) {
            this.section.data.titles = []
            needUpdate = true
        }
        if (!this.section.data.rootFeatures) {
            this.section.data.rootFeatures = [this.getNewFeature()]
            needUpdate = true
        }


        if (_.isEmpty(this.activeFeature)) {
            this.activeFeature = this.section.data.rootFeatures[0] || this.getFeature()

        }
        if (_.isEmpty(this.section.data.bottomWwObjs)) {
            this.section.data.bottomWwObjs = [];
            needUpdate = true;
        }
        if (needUpdate) {
            this.sectionCtrl.update(this.section);
        }
    },
    mounted() {

    },
    methods: {
        /* wwManager:start */
        add(list, options) {
            list.splice(options.index, 0, options.wwObject);
            this.sectionCtrl.update(this.section);
        },
        /* add root element that contain a new section */
        addFeature(_index, where) {
            try {
                const up = (where == 'after') ? parseInt(1) : 0;
                const index = _index + up

                // const newRootFeature = this.getNewFeature()
                const newRootFeature = this.getFeature()
                this.section.data.rootFeatures.splice(index, 0, newRootFeature);
                this.sectionCtrl.update(this.section);
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', err);
            }

        },
        getFeature() {
            try {
                let _feature = {}
                if (this.section.data.rootFeatures.length > 0) {
                    _feature = JSON.parse(JSON.stringify(this.section.data.rootFeatures[0]))
                    wwLib.wwUtils.changeUniqueIds(_feature)

                    _feature.uniqueId = wwLib.wwUtils.getUniqueId()
                } else
                    _feature = this.getNewFeature()

                return _feature
            } catch (error) {
                console.error(error)
            }
        },
        getNewFeature() {
            try {
                const feature = {
                    title: wwLib.wwObject.getDefault({ type: 'ww-row' }),
                    content: wwLib.wwObject.getDefault({
                        type: 'ww-image'
                    }),
                    uniqueId: wwLib.wwUtils.getUniqueId()
                };
                return feature;
            } catch (error) {
                console.error(error)
            }

        },

        async openOptions() {
            try {
                wwLib.wwPopups.addStory('WWFEATURE_CUSTOM', {
                    title: {
                        en: 'Fill in code',
                        fr: 'Inserer le code'
                    },
                    type: 'wwPopupForm',
                    storyData: {
                        fields: [
                            {
                                label: {
                                    en: 'card shadow config:',
                                    fr: 'Configuration de l\'ombre de la carte :'
                                },
                                type: 'shadow',
                                key: 'shadowConfig',
                                valueData: 'section.data.shadowValue',
                                desc: {
                                    en: 'Box-shadow of the banner',
                                    fr: 'L\'ombre de la bannière'
                                }
                            },
                            {
                                label: {
                                    en: 'Border color config:',
                                    fr: 'Configuration de la couleur de la bordure :'
                                },
                                type: 'color',
                                key: 'borderColor',
                                valueData: 'section.data.borderColor',
                                desc: {
                                    en: 'Border color',
                                    fr: 'Couleur de la bordure'
                                }
                            },
                        ]
                    },
                    buttons: {
                        NEXT: {
                            text: {
                                en: 'Finish',
                                fr: 'Terminer'
                            },
                            next: null
                        }
                    }
                })
                let options = {
                    firstPage: 'WWFEATURE_CUSTOM',
                    data: {
                        section: this.section,
                    },
                }

                const result = await wwLib.wwPopups.open(options)

                let needUpdate = false
                if (typeof (result) != 'undefined') {

                    if (typeof (result.shadowConfig) != 'undefined') {
                        console.log('result.shadowConfig:', result.shadowConfig)
                        this.section.data.shadowValue = result.shadowConfig
                        const _shadow = `${this.shadowValue.x}px ${this.shadowValue.y}px ${this.shadowValue.b}px ${this.shadowValue.s}px ${this.shadowValue.c}`;
                        this.section.data.shadowConfig = _shadow
                        needUpdate = true;

                    } if (typeof (result.borderColor) != 'undefined') {
                        this.section.data.borderColor = result.borderColor
                        needUpdate = true;
                    }

                    if (needUpdate) {
                        this.sectionCtrl.update(this.section);
                        this.$forceUpdate();

                    }
                }
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },

        remove(list, options) {
            try {
                list.splice(options.index, 1);
                this.sectionCtrl.update(this.section);
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', err);
            }

        },

        removeFeature(index) {
            try {
                this.section.data.rootFeatures.splice(index, 1);
                if (!this.section.data.rootFeatures.length) {
                    this.addFeature(0, 'after');
                }
                this.sectionCtrl.update(this.section);
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', err);
            }
        },
        /* wwManager:end */

        /* update current selected element */

        selectActiveFeature(feature) {
            try {
                this.activeFeature = feature
            } catch (error) {
                console.error(error)
            }


        },
    }
};
</script>

<!-- This is your CSS -->
<!-- Add "scoped" attribute to limit CSS to this component only -->
<!-- Add lang="scss" or others to use computed CSS -->
<style lang='scss' scoped>
$ww-green-strong: #19947c;
$ww-blue-strong: #1763a9;
.feature-landing {
    width: 100%;
    justify-content: center;
    display: flex;
    margin-bottom: 30px;

    .content-wrapper {
        width: 80%;
        display: none;
        font-family: "Montserrat", sans-serif;
        @media (min-width: 1025px) {
            display: flex;
        }
        .feature-title-wrapper {
            flex-basis: 40%;
            display: flex;
            flex-direction: column;
            padding: 10px 10px 10px 30px;
            @media (max-width: 768px) {
                padding: 10px;
            }

            height: max-content;
            .feature-title {
                border-bottom: 1px solid;
                border-bottom-color: var(--borderColor);
                width: fit-content;
                .title-wrapper {
                    display: flex;
                    margin-bottom: 10px;
                    padding-right: 5px;
                    .title {
                        width: 100%;
                        cursor: pointer;
                        &.selected {
                            box-shadow: var(--shadowConfig);
                        }
                    }
                }
            }
        }

        .feature-content {
            position: relative;
            flex-basis: 60%;
        }
    }

    .mobile-content-wrapper {
        font-family: "Montserrat", sans-serif;
        display: block;
        width: 95%;
        @media (min-width: 1025px) {
            display: none;
        }
        .see-doc {
            border-top: 1px solid;
            border-top-color: var(--borderColor);
        }
    }
}

.fade-enter-active {
    transition: opacity 0.4s ease-in;
}

.fade-leave-active {
    transition: opacity 0.4s ease-out;
}
.fade-enter,
.fade-leave-to {
    opacity: 0;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}

.background {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
}
.feature-background {
    position: absolute;
    top: 0;
    min-height: 30px;
    left: 0;
    height: 100%;
    width: 100%;
}

.root-features {
    @media (min-width: 768px) {
        width: 80%;
        margin-left: 10%;
    }
}

/* wwManager:start */

.feature-wrapper,
.feature-title {
    .ww-orange-button {
        position: absolute;
        transform: translate(-50%, -50%);
        z-index: 2;
    }
}

/* wwManager:end */
</style>