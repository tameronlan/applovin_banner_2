<template>
    <div class="banner-search" ref="formBox" :class="{'banner-search_hidden': hidden}" :style="styleForForm">
        <div class="banner-search-field">
            <div class="banner-search-field__title">{{$t("search.gender_title")}}</div>
            <div class="banner-search-genders">
                <div
                    class="banner-search-gender-btn banner-search-gender-btn_woman"
                    :class="{'banner-search-gender-btn_activated': searchGenders.indexOf('w') !== -1}"
                    @click="choiceGender('w')"
                >
                    <div class="banner-search-gender-btn__icn">
                        <woman-dance/>
                    </div>
                    <div class="banner-search-gender-btn__title">{{$t("search.gender_woman")}}</div>
                </div>

                <div
                    class="banner-search-gender-btn banner-search-gender-btn_man"
                    :class="{'banner-search-gender-btn_activated': searchGenders.indexOf('m') !== -1}"
                    @click="choiceGender('m')"
                >
                    <div class="banner-search-gender-btn__icn">
                        <man-dance/>
                    </div>
                    <div class="banner-search-gender-btn__title">{{$t("search.gender_man")}}</div>
                </div>
            </div>
        </div>

        <div class="banner-search-field">
            <div class="banner-search-field__title">{{$t("search.age_title")}}</div>
            <vue-slider
                v-model="valueAge"
                :max="65"
                :min="18"
                :height="20"
                :tooltip="'always'"
            />
        </div>

        <div class="banner-search-field">
            <div class="banner-search-field__title">{{$t("search.distance_title")}}</div>
            <vue-slider
                v-model="valueDistance"
                :max="20"
                :min="0"
                :height="20"
                :tooltip="'always'"
                :tooltip-formatter="'{value} km'"
            />
        </div>

        <div class="banner-search-slogan" :class="{'banner-search-slogan_animated': isButtonActivated}">
            <div>{{$t("search.make_choice")}}</div>
            <div class="banner-title-underline">
                <underline/>
            </div>
        </div>

        <div></div>

        <div class="banner-search-actions" ref="actionBox">
            <div
                class="banner-search-btn button"
                :class="{'button_disabled': !isButtonActivated}"
                @click="submitForm"
            >
                {{$t("search.btn_act")}}
            </div>

            <div class="banner-search-actions__pointer" v-if="isButtonActivated">
                <pointer/>
            </div>
        </div>
    </div>
</template>

<script>
    import VueSlider from "vue-slider-component"
    import Underline from "@/assets/svg/underline.svg"
    import ManDance from "@/assets/svg/man-dance.svg"
    import WomanDance from "@/assets/svg/woman-dance.svg"
    import Pointer from "@/assets/svg/cursor.svg"

    export default {
        props: ['hidden'],
        name: "Search",
        components: {VueSlider, Underline, ManDance, WomanDance, Pointer},
        data(){
            return {
                searchGenders: [],
                valueAge: [18, 65],
                valueDistance: 20,
                isButtonActivated: false,
                styleForActions: {},
                styleForForm: {}
            }
        },
        mounted() {
            console.log(this.$refs)
            this.redraw()

            if(window.attachEvent) {
                window.attachEvent('onresize', this.redraw);
            } else if(window.addEventListener) {
                window.addEventListener('resize', this.redraw, true);
            }
        },
        computed: {
            pointerStyle(){
                return {display: "none"}
            },
        },
        methods: {
            redraw(){
                let paddingBottom = this.$refs.actionBox.clientHeight + 20

                this.styleForForm =  {
                    paddingBottom: `${paddingBottom}px`
                }
            },
            choiceGender(gender){
                if (this.searchGenders.indexOf(gender) !== -1){
                    this.searchGenders = this.searchGenders.filter(itemGender => gender !== itemGender)
                } else {
                    this.searchGenders.push(gender)
                }

                this.$emit('changeSearchGenders', this.searchGenders)

                this.isButtonActivated = this.searchGenders.length > 0
            },
            submitForm(){
                if (this.searchGenders.length <= 0) { return }

                this.$emit("nextState")
            }
        }
    }
</script>
