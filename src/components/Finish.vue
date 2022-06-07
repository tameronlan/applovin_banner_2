<template>
    <div class="banner-finish">
        <div class="banner-finish__top">
            <div class="banner-finish__title">
                <div>{{$t('matches_found')}}</div>
                <div class="banner-title-underline">
                    <underline/>
                </div>
            </div>

            <div class="banner-finish__counter" :class="{'banner-finish__counter_animated': counterAnimation}">{{counter}}</div>
        </div>

        <div class="banner-finish-map" ref="map">
            <div
                v-for="pointer in pointers" class="banner-finish-map__pointer"
                :style="{height: `${pointerSize}px`, width: `${pointerSize}px`, left: `${pointer.left}%`, top: `${pointer.top}%`}"
            >
                <div class="banner-finish-map__ava" :style="{backgroundImage: `url(${pointer.ava})`}"></div>
            </div>
        </div>

        <div class="banner-finish__bottom">
            <div
                class="banner-finish__btn button"
                :class="{'banner-finish__btn_animated': counterAnimation}"
                @click="$emit('openStore')"
            >
                {{$t('look_matches')}}
            </div>

            <div class="banner-finish-pointer">
                <pointer/>
            </div>
        </div>
    </div>
</template>

<script>
    import Underline from "@/assets/svg/underline.svg"
    import Pointer from "@/assets/svg/cursor.svg"
    // import {usersFinish} from "@/store/finish/users"
    import {shuffle} from "@/lib/shuffle"
    import {usersStart} from "@/store/start/users"

    export default {
        props: ['searchGenders'],
        name: "Finish",
        components: {Underline, Pointer},
        data(){
            return {
                counterDuration: 2000,
                counterInterval: null,
                counterAnimation: false,
                counterTo: 0,
                counter: 0,
                pointerSize: 0,
                pointers: [],
                maxPointers: 4,
                mapWidth: 0,
                mapHeight: 0,
                shuffledUsers: []
            }
        },
        created() {
            console.log("genders", this.searchGenders)

            this.shuffledUsers = shuffle(usersStart.filter(user => this.searchGenders.indexOf(user.gender) !== -1))
        },
        mounted() {
            if (this.$refs.map) {
                this.mapWidth = this.$refs.map.offsetWidth
                this.mapHeight = this.$refs.map.offsetHeight

                let width = this.mapHeight < this.mapWidth ? this.mapHeight : this.mapWidth

                this.pointerSize = width * 0.27
            }

            if(window.attachEvent) {
                window.attachEvent('onresize', this.redraw);
            } else if(window.addEventListener) {
                window.addEventListener('resize', this.redraw, true);
            }

            this.startCounter()
            this.showPointers()
        },
        methods: {
            redraw(){
                this.mapWidth = this.$refs.map.offsetWidth
                this.mapHeight = this.$refs.map.offsetHeight

                let width = this.mapHeight < this.mapWidth ? this.mapHeight : this.mapWidth

                this.pointerSize = width * 0.27
            },
            startCounter(){
                this.counterTo = 500 + Math.floor(Math.random() * 500)

                this.counterInterval = setInterval(this.updateCounter, 70)
            },

            updateCounter(){
                if (this.counter > this.counterTo) {
                    this.counterAnimation = true

                    clearInterval(this.counterInterval)
                } else {
                    this.counter += Math.floor(Math.random() * 45)
                }
            },

            showPointers(){
                let index = this.pointers.length
                let {left, top} = this.getPositionForPointer(index)

                this.pointers.push({
                    left,
                    top,
                    ava: this.shuffledUsers[index].avatar
                })

                if (this.pointers.length < this.maxPointers) {
                    setTimeout(this.showPointers, 300 + Math.random()*500)
                }
            },
            getPositionForPointer(index){
                let startX = 0
                let startY = 0

                switch (index) {
                    case 1:
                        startX = this.mapWidth / 2
                        break;
                    case 2:
                        startY = this.mapHeight / 2
                        break;
                    case 3:
                        startY = this.mapHeight / 2
                        startX = this.mapWidth / 2
                }

                let left = startX + Math.random() * this.mapWidth / 2 - this.pointerSize / 2
                let top = startY + Math.random() * this.mapHeight / 2 - this.pointerSize / 2

                if (top + this.pointerSize + 20 > this.mapHeight) {
                    top = top - this.pointerSize
                } else if (top < 0){
                    top = 0
                }

                if (left + this.pointerSize + 20 > this.mapWidth) {
                    left = left - this.pointerSize
                } else if (left < 0) {
                    left = 0
                }

                left = left * 100 / this.mapWidth
                top = top * 100 / this.mapHeight

                return {left, top}
            }
        }
    }
</script>