<template>
    <div 
        class="markerFake" 
        :style="getMarkerOrigin()"
        id="fake" 
        >
        <markeranno :style="getFakeHoverStyle()" ref="anno" :marker="marker" :hover="true" :active="active" />
        <div
            style="display:flex;justify-content:center;flex-wrap:wrap;cursor:pointer;" 
            @mouseenter="hoverOnMarker(marker)"
            @mouseleave="hoverOffMarker(marker)">
            <span :class="(marker.active ? 'marker-title-active' : 'marker-title-idle')" :style="getTextSize()">{{marker.title}}</span>
            <div class="placeholder">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 40 40">
                    <g v-if="/city/i.test(marker.type)">
                        <path :class="marker.active ? 'city-ring-active' : 'city-ring-idle'" d="M20,5A15,15,0,1,1,5,20,15,15,0,0,1,20,5m0-5A20,20,0,1,0,40,20,20,20,0,0,0,20,0Z"/>
                        <circle :class="marker.active ? 'city-pin-active' : 'city-pin-idle'" cx="20" cy="20" r="3"/>
                    </g>
                    <g v-if="/town/i.test(marker.type)">
                        <circle :class="marker.active ? 'town-pin-active' : 'town-pin-idle'" cx="20" cy="20" r="17.5"/>
                        <path :class="marker.active ? 'town-ring-active' : 'town-ring-idle'" d="M20,5A15,15,0,1,1,5,20,15,15,0,0,1,20,5m0-5A20,20,0,1,0,40,20,20,20,0,0,0,20,0Z"/>
                    </g>
                    <g v-if="/land/i.test(marker.type)">
                        <rect :class="marker.active ? 'land-pin-active' : 'land-pin-idle'" x="2.5" y="2.5" width="35" height="35"/>
                        <!-- :rx="(marker.active) ? 40 : 0" :ry="(marker.active) ? 40 : 0" -->
                    </g>
                </svg>
            </div>
        </div>
    </div>
</template>

<script>
import markeranno from './markerAnno.vue'

export default {
    name: 'markerpreview',
    components: {
        markeranno,
    },
    props: {
        align: String,
        marker: Object,
    },
    data: () => ({
        opacity: .05,
        point: null,
        active: false,
        hover: true,
    }),
    computed: {
        app() {
            return this.$root.$children[0];
        },
    },
    mounted() {
        console.log('Fake marker loaded')
        console.log(this.marker);
        // this.setUp();
        // this.$on('checkOverlap', this.checkForCollision)
    },
    computed: {
        app() {
            return this.$root.$children[0];
        },
        lat () {
            return parseFloat(this.marker.lat || this.marker.latitude)
        },
        lng () {
            return parseFloat(this.marker.lng || this.marker.longitude)
        },
        position () {
            const self = this
            return {
                lat () {
                    return self.lat
                },
                lng () {
                    return self.lng
                }
            }
        }
    },
    // watch: {
    //     zoom() {
    //         this.getTextSize();
    //     }
    // },
    methods: {
        getFakeFullStyle() {
            if (/xs|sm/.test(this.$vuetify.breakpoint.name)) {
                return `
                `
            } else {
                return `
                `
            }
        },
        getFakeHoverStyle() {
            // if (/xs|sm/.test(this.$vuetify.breakpoint.name)) {
            //     return `
            //         border: 2px solid blue;
            //     `
            // } else {
            //     return `
            //         border: 2px solid green;
            //     `
            // }
        },
        assignAnnoElt() {
            this.$refs.anno.init();
            this.marker.labelElt = this.$el;
        },
        resetActive() {
            this.active = false;
            if (!this.active)
                this.$el.classList.replace('marker-grow', 'marker-shrink');
        },
        resetScale() {
            // console.log(this.marker.title);
            // console.log(this.$el.classList);
            this.$el.classList.replace('marker-grow', 'marker-shrink');
        },
        resetHover() {
            this.hover = false;
            // this.$el.classList.replace('marker-grow', 'marker-shrink');
        },
        
        getMarkerOrigin() {
            try {
                return `
                    transform-origin: 50% calc(100% - ${(this.marker.type.toLowerCase() == 'city') ? 5 : 3.5}px);
                `
            } catch(err) {
                return `
                     transform-origin: 50% calc(100% - 5px);
                `
            }
        },
        hoverOnMarker(marker) {
            // let res = this.app.mainmap.markers.find(mark => {
            //     return mark.title == marker.title
            // })
            this.hover = true;
        },
        hoverOffMarker(marker) {
            this.hover = false;
        },
        clickOn() {
            console.log(this.$el)

        },
        getTextSize() {
                return `
                    width: 100%;
                    font-weight: ${this.marker.type.toLowerCase() == 'city' ? 700 : 500};
                    font-size: ${this.marker.type.toLowerCase() == 'city' ? 1 : .875 }rem;
                ` 
            // }
        },
    }
}
</script>

<style>



#fake > .annoBox {
    position: relative;
    bottom: 0px;
    /* border: 2px solid blue; */
}

.markerFake {
    /* padding-top: 2rem; */
    box-sizing: border-box;
    /* border: 2px solid red; */
    box-sizing: border-box;
    display: flex;
    justify-content: flex-start;
    flex-direction: column;
    align-items: center;
    width: 100%;
    /* height: auto; */
    flex-wrap: nowrap;
    transition: 
        transform 200ms var(--quart) 20ms,
        z-index 200ms var(--quart) 20ms;
    transform-style: preserve-3D;
    z-index: 100;
}
.marker-grow {
    transform: scale(1.25);
    /* z-index: 20; */
}
.marker-shrink {
    transform: scale(1);
    /* z-index: 3; */
}

.marker-title-idle {
    color: var(--text-titles-color);
    text-shadow:
        -1px -1px 0 var(--text-titles-stroke),  
        1px -1px 0 var(--text-titles-stroke),
        -1px 1px 0 var(--text-titles-stroke),
        1px 1px 0 var(--text-titles-stroke);
}

.marker-title-active {
    color: var(--color-active);
    
    text-shadow:
        -0.5px -0.5px 0 var(--text-titles-stroke),  
        0.5px -0.5px 0 var(--text-titles-stroke),
        -0.5px 0.5px 0 var(--text-titles-stroke),
        0.5px 0.5px 0 var(--text-titles-stroke);
}

[class^="marker-title-"] {
    display: flex;
    justify-content: center;
    font-family: 'Roboto';
    letter-spacing: .0875rem;    
    transition: font-size 140ms var(--quint) 20ms;
}

svg {
    width: 100%;
}

.placeholder {
    box-sizing: border-box;
    width: var(--marker-city-width);
    height: var(--marker-city-width);
    transition: 
        font-size 140ms var(--quint) 20ms,
        text-shadow 140ms var(--quint) 20ms,
        width 140ms var(--quint) 20ms,
        height 140ms var(--quint) 20ms;
    display: flex;
    justify-content: center;
    align-items: center;

}

[class^="city-pin-"] {
    stroke-miterlimit: 10;
    
}
.city-pin-active {
    stroke: var(--color-active);
    fill: var(--color-active);
    stroke-width: 10px;
}
.city-pin-idle {
    stroke: var(--color-idle);
    stroke-width: 8px;
}

.city-ring-active, .town-ring-active {
    stroke: var(--color-active);
    fill: var(--color-active);
}
.city-ring-idle, .town-ring-idle {
    stroke: var(--color-idle);
    fill: var(--color-idle);
    /* stroke-width: 1px; */
}

[class^="town-pin-"] {
    
}

.town-pin-active {
    fill: #fff;
    opacity: 0.2;
}

.town-pin-idle {
    fill: #fff;
    opacity: 0.2;
}

[class^="land-pin-"] {
    stroke-width: 8px;
}

.land-pin-active {
    stroke: var(--color-active);
    fill: var(--color-active);
    
}
.land-pin-idle {
    stroke: var(--color-idle);
    fill: var(--color-idle);
}

</style>
