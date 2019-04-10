<template>
    <div class="fakescreen">
        <div class="loading-icon" ></div>
    </div>
</template>

<script>
import loading from '../assets/boxLock.json'
// import loadingUnlocked from '../assets/boxLockRev.json'
import * as lottie from 'lottie-web'

export default {
    name: 'loadingicon',
    data: () => ({
        speed: 1,
        elt: {},
        autoplay: true,
    }),
    computed: {
        app() {
            return this.$root.$children[0]
        }
    }, 
    mounted() {
        this.elt = this.$el.children[0];
        this.init();
    },
    methods: {
        init() {
            this.animData = this.buildAnimation();
            this.animData.addEventListener('complete', this.isDone);
            this.animData.play();
        },
        isDone() {
            this.$emit('done');
        },
        play() {
            // console.log('Loading should trigger');
            // this.animData.play();
        },
        buildAnimation() {
            const self = this;
            const animData = {
                wrapper: self.elt,
                animType: 'svg',
                loop: false,
                prerender: true,
                autoplay: true,
            };
            animData.animationData = loading;
            return lottie.loadAnimation(animData);
        },
    }
}
</script>

<style>
svg {
    width: 100%;
}

.fakescreen {
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    position: absolute;
}

.anim-lock-main {
    fill: var(--color-invert);
    stroke: var(--color-invert);
}

.anim-lock-mask {
    fill: var(--color-bg);
    stroke: var(--color-bg);
}

.anim-bg {
    fill: transparent;
    stroke: transparent;
}
.loading-icon {
    width: 300px;
}
</style>
