<template>
    <div class="banner-start" :class="{'banner-start_hidden': hidden}">
        <div
            v-for="user in users.list"
            class="banner-start__user"
            :class="{'banner-start__user_hidden': user.isHidden}"
            :style="{backgroundImage: `url(${user.ava})`, left: `${user.left}px`, top: `${user.top}px`, width: `${user.height}px`, height: `${user.width}px`}"
        ></div>

        <div
            class="banner-start__button button"
            ref="startButton"
            @click="$emit('nextState')"
            :style="{top: `${windowHeight/2 - buttonHeight/2}px`}"
        >
            {{$t('find_match')}}
        </div>

        <div class="banner-start-pointer">
            <pointer/>
        </div>
    </div>
</template>

<script>
    import {mapState, mapGetters} from 'vuex';
    import Pointer from "@/assets/svg/cursor.svg"
    import {usersStart} from "@/store/start/users"
    import {shuffle} from "@/lib/shuffle"

    export default {
        props: ['hidden'],
        name: "start",
        components: {Pointer},
        data(){
            return {
                windowWidth: 0,
                windowHeight: 0,
                buttonHeight: 0,
                users: {
                    list: [],
                    shuffled: shuffle(usersStart)
                },
                isLoading: false
            }
        },
        mounted() {
            this.buttonHeight = this.$refs.startButton ? this.$refs.startButton.getBoundingClientRect().height : 0;

            this.windowWidth = window.innerWidth || document.documentElement.clientWidth ||
                document.body.clientWidth;
            this.windowHeight = window.innerHeight || document.documentElement.clientHeight ||
                document.body.clientHeight;

            if(window.attachEvent) {
                window.attachEvent('onresize', this.redraw);
            } else if(window.addEventListener) {
                window.addEventListener('resize', this.redraw, true);
            }

            this.startFillUsers()
        },
        destroyed() {
            if(window.detachEvent) {
                window.detachEvent('onresize', this.redraw);
            } else if(window.removeEventListener) {
                window.removeEventListener('resize', this.redraw, true);
            }
        },
        computed: {

        },
        methods: {
            redraw(){
                setTimeout(()=>{
                    let id = window.setTimeout(() => {}, 0);

                    while (id--) {
                        window.clearTimeout(id);
                    }

                    this.windowWidth = window.innerWidth || document.documentElement.clientWidth ||
                        document.body.clientWidth;
                    this.windowHeight = window.innerHeight|| document.documentElement.clientHeight||
                        document.body.clientHeight;

                    this.buttonHeight = this.$refs.startButton.getBoundingClientRect().height

                    this.users.list = [];
                    this.startFillUsers()
                }, 150)
            },
            startFillUsers() {
                let startX = 0; // переменная для хранение сдвигов по x для ячеек
                let startY = 0; // переменная для хранение сдвигов по y для ячеек
                let countUsersInRow = 3; // сколько пользователей полностью должно влезть точно в экран
                let spaceForAva = 7 * 2; // отступы для фоток
                let possibleHeight = ((this.windowHeight - this.buttonHeight)/2 - 32)/2.5; // мы вычисляем возможную высоту авы
                let possibleWidth = (this.windowWidth - spaceForAva * countUsersInRow) / countUsersInRow; // мы вычисляем возможную длинну авы

                let size = possibleHeight // размер фотки выбираем по минимальным значениям высоты или длинны
                let row = 0 // храним строку, которую заполняем
                let indexInRow = 0 // какой индекс у фотки в строке
                let indexInShuffledUsers = 0 // индекс для получения пользователя в перемешанных пользователях
                let spaceForPaddingButton = 32 * 2

                startY = -size / 2
                startX = -size / 2

                while (row < 6){
                    let startForRow = (row % 2 === 0 ? startX : 0) + 15 // 15 - смещение для отступов
                    let left = startForRow + (size + spaceForAva) * indexInRow

                    if (left > this.windowWidth) {
                        row ++;
                        indexInRow = 1;
                        left = (row % 2 === 0 ? startX : 0) + 15
                    } else {
                        indexInRow++
                    }

                    if (indexInShuffledUsers >= this.users.shuffled.length) {
                        indexInShuffledUsers = 0
                    }

                    if (row < 6){
                        this.users.list.push({
                            left: left,
                            top: (row > 2 ? spaceForPaddingButton + this.buttonHeight + startY : startY) + row * size,
                            ava: this.users.shuffled[indexInShuffledUsers].avatar,
                            height: size,
                            width: size,
                            id: this.users.length,
                            isHidden: true
                        })

                        indexInShuffledUsers++
                    }
                }

                this.startShowUsers()
            },
            startShowUsers(){
                let duration = 1200
                let step = duration / this.users.list.length

                this.users.list.forEach((user, index) => {
                    setTimeout(()=> {
                        this.users.list[index].isHidden = false
                    }, step * index)
                })

                setTimeout(this.rotateUsers, duration + 500)
            },
            rotateUsers(){
                let countRotate = 5
                let rotateIndices = []
                let rotateToIndices = []

                for (let i=0; i<countRotate; i++) {
                    let index = Math.floor(Math.random() * this.users.list.length)

                    // переделываем рандом, если пользователь уже был в массиве на ротейт
                    if (rotateIndices.indexOf(index) !== -1){
                        index = Math.floor(Math.random() * this.users.list.length)
                    }

                    rotateIndices.push(index)
                }

                for (let i=0; i<countRotate; i++) {
                    let index = Math.floor(Math.random() * this.users.shuffled.length)

                    // переделываем рандом, если пользователь уже был в массиве на ротейт
                    if (rotateToIndices.indexOf(index) !== -1){
                        index = Math.floor(Math.random() * this.users.shuffled.length)
                    }

                    rotateToIndices.push(index)
                }

                rotateIndices.forEach((indexUser, i) => {
                    setTimeout(()=> {
                        this.users.list[indexUser].isHidden = true

                        setTimeout(()=>{
                            this.users.list[indexUser].ava = this.users.shuffled[rotateToIndices[i]]
                            this.users.list[indexUser].isHidden = false
                        }, 500)
                    }, i * 100)
                })

                setTimeout(this.rotateUsers, 1000 + Math.random() + 1000)
            }
        }
    }
</script>
