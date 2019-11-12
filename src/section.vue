<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div :style="customStyle">
        <!-- wwManager:start -->
        <wwSectionEditMenu :sectionCtrl="sectionCtrl" :options="openOptions"></wwSectionEditMenu>
        <!-- wwManager:end -->
        <!-- <wwObject class="background" :ww-object="section.data.bg" ww-category="background"></wwObject> -->

        <wwLayoutColumn tag="div" ww-default="ww-text" :ww-list="section.data.titles" class="content feature-title" @ww-add="add(section.data.titles, $event)" @ww-remove="remove(section.data.titles, $event)">
            <wwObject tag="div" v-for="title in section.data.titles" :key="title.uniqueId" :ww-object="title"></wwObject>
        </wwLayoutColumn>
        <div class="feature-landing">
            <div class="content-wrapper">
                <!-- <div class="feature-content">
                    <wwObject :ww-object="activeFeature.content"></wwObject>
                </div>-->
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
            activeFeature: [],
            animate: true,
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
        animated() {
            return this.animate
        },
        customStyle() {
            return {
                '--shadowConfig': this.section.data.shadowConfig,
            }
        }


    },
    created() {

        let needUpdate = false

        //Initialize section data
        this.section.data = this.section.data || {};
        this.section.data.shadowConfig =
            this.section.data.shadowConfig || '0 1px 23px -5px rgba(0, 0, 0, 0.2)';

        if (!this.section.data.bg) {
            needUpdate = true
            this.section.data.bg = wwLib.wwObject.getDefault({
                type: 'ww-color'
            });
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
            this.activeFeature = this.getFeature()
            needUpdate = true
        }


        if (_.isEmpty(this.activeFeature)) {
            this.activeFeature = this.getFeature()
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
                                type: 'text', //shadow dosen't work
                                key: 'shadowConfig',
                                valueData: 'section.data.shadowConfig',
                                desc: {
                                    en: 'Box-shadow of the banner: offset-x | offset-y | blur-radius | spread-radius | color',
                                    fr: 'L\'ombre de la bannière : offset-x | offset-y | blur-radius | spread-radius | color'
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
                        this.section.data.shadowConfig = result.shadowConfig;
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
        /* create a popup forum */
        async edit() {
            wwLib.wwPopups.addStory('WWDOT_CUSTOM', {
                title: {
                    en: 'Color picker',
                    fr: 'Choisir une couleur'
                },
                type: 'wwPopupForm',
                storyData: {
                    fields: [
                        {
                            label: {
                                en: 'Border Color:',
                                fr: 'Couleur de la bordure :'
                            },
                            type: 'color',
                            key: 'borderColor',
                            value: "#42b983",
                            valueData: 'borderColor',
                            desc: {
                                en: 'Choose a border color',
                                fr: 'Changer la couleur de la bordure'
                            }
                        },
                    ]
                },
                buttons: {

                    NEXT: {
                        text: {
                            en: 'Ok',
                            fr: 'Ok'
                        },
                        next: false
                    }
                }
            })

            let options = {
                firstPage: 'WWDOT_CUSTOM',
                data: {
                    wwObject: this.wwObject,
                }
            }
            return options
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
            this.animate = false
            this.activeFeature = feature
            this.animate = true
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
        display: flex;
        font-family: "Montserrat", sans-serif;
        @media (max-width: 768px) {
            width: 90%;
            flex-direction: column;
        }

        .feature-title-wrapper {
            flex-basis: 30%;
            display: flex;
            flex-direction: column;
            padding: 10px 10px 10px 30px;
            @media (max-width: 768px) {
                padding: 10px;
            }
            height: max-content;
            .feature-title {
                border-bottom: grey 1px solid;
                width: fit-content;
                .title-wrapper {
                    display: flex;
                    .title {
                        width: 100%;
                        cursor: pointer;
                        &.selected {
                            box-shadow: var(--shadowConfig);
                        }
                    }
                }
                .title-wrapper:last-child {
                    margin-bottom: 15px;
                }
            }
        }

        .feature-content {
            flex-basis: 60%;
        }
    }
}
// Vuejs transition
// .fade-enter-active {
//     transition: opacity 5s ease 0.5s;
// }

// .fade-leave-active {
//     transition: opacity 10s;
// }

.fade-enter-active {
    transition: 1s opacity 5s;
}
}
.fade-leave-active {
    transition: 1s opacity 7s;
}
.fade-enter,
.fade-leave-to {
    opacity: 0;
    position: absolute;
    top: 0;
    left: 0;
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

.contextmenu {
    position: absolute;
    top: 0;
    left: 30px;
    transform: translate(-50%, -50%);
    width: 30px;
    height: 30px;
    color: white;
    background-color: #ef811a;
    border-radius: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 1.2rem;
    cursor: pointer;
    z-index: 1;
}

.feature-title {
    .ww-orange-button {
        position: absolute;
        // top: 0;
        // left: 0;
        transform: translate(-50%, -50%);
        z-index: 2;
    }
}

/* wwManager:end */
</style>