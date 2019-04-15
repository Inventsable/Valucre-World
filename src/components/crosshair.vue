<template>
    <div class="mapCenter" :style="getStyle()">
            <div id="crossHair" class="mapCenterMark">
                <svg id="crossCheck" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20">
                    <g v-if="crosshair == 'default'">
                        <line class="crossHair" x1="10" x2="10" y2="20"/>
                        <line class="crossHair" x1="20" y1="10" y2="10"/>
                    </g>
                    <g v-if="crosshair == 'marker'">
                        <path class="marker-icon" d="M10,18.4,5.13,10.09A5.63,5.63,0,0,1,6,3.25H6a5.65,5.65,0,0,1,8,0h0a5.66,5.66,0,0,1,.88,6.84ZM10,6.33a1,1,0,0,1,0,2,1,1,0,0,1,0-2Z" transform="translate(0)"/>
                    </g>
                </svg>
            </div>
        </div>
</template>

<script>
export default {
    name: 'crosshair',
    computed: {
        app() {
            return this.$root.$children[0];
        },
        crosshair() {
            if (this.app.isLoaded)
                return this.app.mainmap.crosshairType;
            else
                return 'default';
        }
    },
    mounted() {
        // this.app.mainmap.crossElt = this.$el;
    },
    methods: {
        getStyle() {
            if (this.app.isLoaded) {
                return `
                    top: ${this.app.mainmap.isAdding ? -8 : 0}px;
                `
            }
        }
    }
}
</script>

<style>
    .mapCenter {
        pointer-events: none;
        top: 0px;
        left: 0px;
        position: absolute;
        width: 100%;
        height: calc(100vh - 48px);
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .mapCenterMark {
        /* border: 2px solid white; */
        width: 20px;
        height: 20px;
        /* border-radius: 20px; */
    }
    .crossHair {
        stroke: rgba(0,0,0,0.125);
        stroke-miterlimit: 10;
        stroke-width: 3px;
    }
    .marker-icon {
        fill: none;
        stroke: #000;
        stroke-linecap: round;
        stroke-linejoin: round;
        stroke-width: 2px;
      }
</style>
