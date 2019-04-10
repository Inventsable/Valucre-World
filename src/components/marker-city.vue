<template>
    <div class="marker" :style="getMarkerOrigin()">
        <div class="annobox" :style="getAnnoBoxStyle()">

        </div>
        <div @click="clickOn"
        style="display:flex;justify-content:center;flex-wrap:wrap;" 
        @mouseenter="hoverOnMarker(marker)"
        @mouseleave="hoverOffMarker(marker)">
            <span :class="(marker.active ? 'marker-title-active' : 'marker-title-idle')" :style="getTextSize()">{{marker.title}}</span>
            <div class="placeholder">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 40 40">
                    <g v-if="marker.type.toLowerCase() == 'city'">
                        <path :class="marker.active ? 'city-ring-active' : 'city-ring-idle'" d="M20,5A15,15,0,1,1,5,20,15,15,0,0,1,20,5m0-5A20,20,0,1,0,40,20,20,20,0,0,0,20,0Z"/>
                        <circle :class="marker.active ? 'city-pin-active' : 'city-pin-idle'" cx="20" cy="20" r="3"/>
                    </g>
                    <g v-if="marker.type.toLowerCase() == 'town'">
                        <circle :class="marker.active ? 'town-pin-active' : 'town-pin-idle'" cx="20" cy="20" r="17.5"/>
                        <path :class="marker.active ? 'town-ring-active' : 'town-ring-idle'" d="M20,5A15,15,0,1,1,5,20,15,15,0,0,1,20,5m0-5A20,20,0,1,0,40,20,20,20,0,0,0,20,0Z"/>
                    </g>
                    <g v-if="marker.type.toLowerCase() == 'land'">
                        <rect :class="marker.active ? 'land-pin-active' : 'land-pin-idle'" x="2.5" y="2.5" width="35" height="35"/>
                        <!-- :rx="(marker.active) ? 40 : 0" :ry="(marker.active) ? 40 : 0" -->
                    </g>
                </svg>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'marker-city',
    props: {
        map: Object,
        align: String,
        marker: Object,
    },
    data: () => ({
        opacity: .05,
        point: null,
        hover: false,
    }),
    computed: {
        app() {
            return this.$root.$children[0];
        },
    },
    mounted() {
        this.setUp();
    },
    destroyed () {
        this.$overlay.setMap(null)
        this.$overlay = undefined
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
        getAnnoBoxStyle() {
            return `
                position: absolute;
                bottom: 26px;
                opacity: ${this.marker.active ? 1 : 0};
                transition: width 200ms var(--quart) 20ms;
                
                padding: ${this.marker.active ? '4px 10px' : '4px 10px'};
                width: ${this.marker.active ? 150 : 150}px;
                height: 50px;
                background-color: rgba(255,255,255,.95);
                z-index: 8;
            
            `
        },
        getMarkerOrigin() {
            return `
                z-index: 1;
                transform-origin: 50% calc(100% - ${(this.marker.type.toLowerCase() == 'city') ? 5 : 3.5}px);
            `
        },
        hoverOnMarker(marker) {
            // console.log();
            // this.hover = true;
            let res = this.app.mainmap.markers.find(mark => {
                return mark.title == marker.title
            })
            // this.app.clickingOnMarker = true;
            // this.$parent.temp = this.point;
            // // console.log(this.$parent.temp)
            res.hover = true;
        },
        hoverOffMarker(marker) {
            let res = this.app.mainmap.markers.find(mark => {
                return mark.title == marker.title
            })
            res.hover = false;
        },
        clickOn() {

            console.log(this.$el)
            let res = this.app.mainmap.markers.find(mark => {
                return mark.title == this.marker.title
            })
            res.hover = false;
            res.active = !res.active;
            if (res.active) {
                this.$el.classList.toggle('marker-grow', !this.$el.classList.contains('marker-grow'));
                this.$el.classList.remove('marker-shrink');
                this.app.mainmap.panToSelectedMarker();
            } else {
                this.$el.classList.replace('marker-grow', 'marker-shrink')
                // this.$el.classList.toggle()
            }

            
            // console.log('Marker was clicked')
            // console.log(this.marker)
            // let coords = {
            //     lat: this.marker.lat,
            //     lng: this.marker.lng,
            // }
            // this.marker.hover = false;
            // this.$parent.$parent.setLatLng(coords);
            // this.$parent.$parent.loreRef = this.marker.dbref;
            // this.$parent.$parent.selectedMarker = this.marker;
            // this.$parent.$parent.showInfo = true;
        },
        getTextSize() {
            // if (this.marker.active) {
            //     return `
            //         font-weight: ${this.marker.type.toLowerCase() == 'city' ? 700 : 500};
            //         font-size: ${this.marker.type.toLowerCase() == 'city' ? 1.25 : 1 }rem;
            //     ` 
            // } else {
                return `
                    width: 100%;
                    font-weight: ${this.marker.type.toLowerCase() == 'city' ? 700 : 500};
                    font-size: ${this.marker.type.toLowerCase() == 'city' ? 1 : .875 }rem;
                ` 
            // }
        },
        setUp() {
            const self = this;
            class Overlay extends google.maps.OverlayView {
                constructor () {
                    super()
                    this.setMap(self.map)
                    // this.draw = () => this.repaint()
                    this.setPosition = () => this.repaint()
                }
                draw () {
                    const div = self.$el
                    const projection = this.getProjection()

                    const point = this.getProjection().fromLatLngToDivPixel(self.marker.latlng);
                    self.point = point;
                    if (point) {
                        if (/top/.test(self.align))
                            div.style.top = `${point.y}px`;
                        else if (/bottom/.test(self.align))
                            div.style.top = `${point.y - div.clientHeight}px`;
                        else 
                            div.style.top = `${point.y - (div.clientHeight / 2)}px`;

                        if (/left/.test(self.align))
                            div.style.left = `${point.x}px`;
                        else if (/right/.test(self.align))
                            div.style.left = `${point.x - div.clientWidth}px`;
                        else
                            div.style.left = `${point.x - (div.clientWidth / 2)}px`;
                    } 
                }
                onAdd () {
                    const div = self.$el
                    const panes = this.getPanes()
                    div.style.position = 'absolute'
                    panes.overlayLayer.appendChild(div)
                    panes.overlayMouseTarget.appendChild(div)
                }
                onRemove () {
                    self.$el.remove()
                }
            }
            this.$overlay = new Overlay(map)
        },
        
    }
}
</script>

<style>

.annobox {
    display: flex;
    justify-content: center;
    z-index: 7;
}

.marker {
    box-sizing: border-box;
    display: flex;
    flex-direction: column;
    align-items: center;
    cursor: pointer;
    transition: transform 200ms var(--quart) 20ms;
    transform-style: preserve-3D;
    z-index: 3;
}
.marker-grow {
    transform: scale(2);
    z-index: 3;
}
.marker-shrink {
    transform: scale(1);
    z-index: 3;
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
