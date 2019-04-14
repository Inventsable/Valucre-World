<template>
    <div 
        v-click-outside="onClickOutside" 
        class="marker" 
        :style="getMarkerOrigin()" 
        v-show="checkStatus()"
        >
        <markeranno ref="anno" :marker="marker" :hover="hover" :active="active" />
        <div
            @click="clickOn"
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
    name: 'marker-city',
    components: {
        markeranno,
    },
    props: {
        map: Object,
        align: String,
        marker: Object,
    },
    data: () => ({
        opacity: .05,
        point: null,
        active: false,
        hover: false,
    }),
    computed: {
        app() {
            return this.$root.$children[0];
        },
    },
    mounted() {
        this.setUp();
        this.$on('checkOverlap', this.checkForCollision)
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
        checkStatus() {
            let result = false;
            if (this.marker.active) {
                // console.log(`${this.marker.title} passes from selection`)
                return true;
            } else {
                if (this.app.mainmap.zoom < +this.marker.minZoom) {
                    if (/xs|sm/.test(this.$vuetify.breakpoint.name)) {
                        if (this.app.mainmap.zoom >= +this.marker.minZoom - 1) {
                            if (this.marker.hide) {
                                // console.log(`${this.marker.title} failed because hidden`);
                            } else {
                                // console.log(`${this.marker.title} matches mobile ${+this.marker.minZoom - 1} of current ${this.app.mainmap.zoom}`);
                                result = true;
                            }
                        } else {
                            // console.log(`${this.marker.title} fails mobile ${+this.marker.minZoom - 1} of current ${this.app.mainmap.zoom}`)
                            if (!this.marker.hide) {
                                result = false;
                            }
                        }
                    } else {
                        result = false;
                        // console.log(`${this.marker.title} needs ${this.marker.minZoom} of current ${this.app.mainmap.zoom}`);
                    }
                } else if (this.marker.hide) {
                    // console.log(`${this.marker.title} is hidden`);
                    result = false;
                } else {
                    // console.log(`${this.marker.title} is shown by default`)
                    result = true;
                }
            }
            return result;
        },
        onClickOutside (e, el) {
            // console.log('clickedOutside');
            if (this.active) {
                if (!this.app.mainmap.wasScrolling) {
                    // console.log('Was not scrolling')
                    if (this.app.mainmap.inMap) {
                        this.active = false;
                        this.marker.active = false;
                        // console.log(e);
                        // console.log(`Deselect ${this.marker.title}`)
                        this.app.mainmap.resetHovers();
                        this.app.mainmap.resetAllScales();
                    } else {
                        // console.log(`Click wasn't in map`)
                    }
                }
            }
            // console.log('click heard outside element:', el);
            // console.log('element clicked:', e.target);
            // console.log('event:', e);
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
        resetDistance() {
            this.$refs.anno.calculatingDistance = false;
        },
        checkForCollision() {
            let rect1 = this.$el.getBoundingClientRect();
            let rect2 = this.app.mainmap.crossElt.getBoundingClientRect();
            let check = !(rect1.right < rect2.left || 
                rect1.left > rect2.right || 
                rect1.bottom < rect2.top || 
                rect1.top > rect2.bottom)  
            if (check) {
                this.hover = true;
                this.marker.hover = true;
            } else {
                this.hover = false;
                this.marker.hover = false;
            }
            this.checkIfDistance();
        },
        checkForCollisionOfAnnoBox() {
            try {

                if (this.active || this.hover) {
                    let rect1 = this.marker.annoElt.getBoundingClientRect();
                    let mapBounds = this.app.mainmap.map.getBounds();
                    // let markersWithinView = this.app.mainmap.markers.filter(marker => {
                    //     return mapBounds.contains(marker.latlng) && marker !== this.marker;
                    // })
                    // console.log(markersWithinView);
                    // markersWithinView.forEach(target => {
                    this.app.mainmap.markers.forEach(target => {
                        // console.log(`Checking ${target.title} for overlap hover`)
                        let rect2 = target.labelElt.getBoundingClientRect();
                        let check = !(rect1.right < rect2.left || 
                            rect1.left > rect2.right || 
                            rect1.bottom < rect2.top || 
                            rect1.top > rect2.bottom)  
                        if (check) {
                            // console.log(`Hiding ${target.title} marker`)
                            target.hide = true;
                        } else {
                            target.hide = false;
                        }
                    })
                }
            } catch(err) {
                console.log(`Marker doesn't exist yet`)
            }
        },
        getMarkerOrigin() {
            // z-index: 1;
            return `
                transform-origin: 50% calc(100% - ${(this.marker.type.toLowerCase() == 'city') ? 5 : 3.5}px);
            `
        },
        hoverOnMarker(marker) {
            let res = this.app.mainmap.markers.find(mark => {
                return mark.title == marker.title
            })
            this.hover = true;
            res.hover = true;
            this.app.clickingOnMarker = true;
            this.checkIfDistance();
            const self = this;
            setTimeout(() => {
                self.checkForCollisionOfAnnoBox();
            }, 100);
        },
        hoverOffMarker(marker) {
            let res = this.app.mainmap.markers.find(mark => {
                return mark.title == marker.title
            })
            this.app.mainmap.resetHovers();
            this.hover = false;
            res.hover = false;
            this.app.clickingOnMarker = false;
            this.app.mainmap.resetHidden();
            this.checkIfDistance();
        },
        checkIfDistance() {
            if (this.hover) {
                // console.log(`${this.marker.title} is Hovered`)
                let res = this.app.mainmap.markers.find(mark => {
                    return mark.title == this.marker.title
                })
                // console.log(this.app.mainmap.selectedMarker)
                if ((this.app.mainmap.selectedMarker)
                    && this.app.mainmap.selectedMarker.title !== this.marker.title) {
                    this.$refs.anno.calculatingDistance = true;
                } else {
                    this.$refs.anno.calculatingDistance = false;
                    this.app.mainmap.resetAllCalcDistance();
                }
                

            } else {
                // console.log('Reset all distances');
                // this.app.mainmap.resetAllCalcDistance();
            }

        },
        clickOn() {
            // console.log(this.$el)
            let res = this.app.mainmap.markers.find(mark => {
                return mark.title == this.marker.title
            })
            // if (this.app.mainmap.selectedMarker) {
                // }
            res.hover = false;
            res.active = !res.active;
            if (res.active) {
                this.app.mainmap.resetActivesExcept(this.marker.dbref);
                res.active = true;
            }

            // console.log(this.active)
            // console.log(this.active)
            this.active = res.active;
            if (res.active) {
                this.$el.classList.toggle('marker-grow', !this.$el.classList.contains('marker-grow'));
                this.$el.classList.remove('marker-shrink');
                this.app.mainmap.panToSelectedMarker();
                this.active = true;
            } else {
                this.$el.classList.replace('marker-grow', 'marker-shrink')
                this.app.mainmap.resetAllCalcDistance();
                this.checkForCollisionOfAnnoBox();
                this.active = false;
                // this.$el.classList.toggle()
            }


        },
        getTextSize() {
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

.marker {
    box-sizing: border-box;
    /* border: 2px solid red; */
    box-sizing: border-box;
    display: flex;
    justify-content: flex-start;
    flex-direction: column;
    align-items: center;
    
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
