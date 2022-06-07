<template>
    <div class="app">
        <transition name="fade-slide">
            <start
                v-if="state===STATE_START"
                @nextState="changeState(STATE_SEARCH)"
            />
        </transition>
        <transition name="fade-slide">
            <search
                v-if="state===STATE_SEARCH"
                @nextState="changeState(STATE_FINISH)"
                @changeSearchGenders="changeSearchGenders"
            />
        </transition>
        <transition name="fade-slide">
            <finish
                v-if="state===STATE_FINISH"
                :searchGenders="searchGenders"
                @openStore="openStore"
            />
        </transition>
    </div>
</template>

<script>
    import Start from "@/components/Start";
    import Search from "@/components/Search";
    import Finish from "@/components/Finish";
    import {androidLinkSM, iosLinkSM} from "@/settings/sm/links";
    import {androidLinkEM, iosLinkEM} from "@/settings/em/links";
    import {androidLinkIH, iosLinkIH} from "@/settings/ih/links";

    let STATE_START     = "start"
    let STATE_SEARCH    = "search"
    let STATE_FINISH    = "finish"

    export default {
        name: 'app',
        components: {
            Finish,
            Search,
            Start,
        },
        data(){
            return {
                STATE_START: STATE_START,
                STATE_SEARCH: STATE_SEARCH,
                STATE_FINISH: STATE_FINISH,
                state: STATE_START,
                searchGenders: []
            }
        },
        computed: {
        },
        methods: {
            openStore(){
                let androidLink, iosLink;

                if (process.env.VUE_APP_NAME === "sm"){
                    androidLink = androidLinkSM;
                    iosLink = iosLinkSM;
                } else if (process.env.VUE_APP_NAME === "ih") {
                    androidLink = androidLinkIH;
                    iosLink = iosLinkIH;
                } else if (process.env.VUE_APP_NAME === "em") {
                    androidLink = androidLinkEM;
                    iosLink = iosLinkEM;
                }

                let url = /android/i.test(navigator.userAgent) ? androidLink : iosLink;

                if (window.mraid){
                    mraid.open(url);
                } else {
                    this.$toasted.show(`MRAD is not initialized::${url}`)
                }
            },
            changeState(state){
                this.state = state
            },
            changeSearchGenders(genders){
                this.searchGenders = genders
            }
        }
    };
</script>
