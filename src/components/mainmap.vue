<template>
    <div class="content">
        <div id="map"></div>
        <div v-if="hasMap">
            <v-tooltip
                v-model="marker.hover" bottom
                v-for="marker in citymarkers"
                :key="marker.dbref">
                <template v-slot:activator="{ on }">
                    <markercity
                        v-on="on"
                        :ref="marker.dbref"
                        v-show="showCities"
                        :marker="marker"
                        :map="map"
                        align="bottom"
                        />
                </template>
                <span v-html="marker.tooltip"></span>
            </v-tooltip>
            <v-tooltip
                v-model="marker.hover" bottom
                v-for="marker in townmarkers"
                :key="marker.dbref">
                <template v-slot:activator="{ on }">
                    <markercity
                        v-on="on"
                        :ref="marker.dbref"
                        v-show="showTowns"
                        :marker="marker"
                        :map="map"
                        align="bottom"
                        />
                </template>
                <span v-html="marker.tooltip"></span>
            </v-tooltip>
            <v-tooltip
                v-model="marker.hover" bottom
                v-for="marker in landmarkers"
                :key="marker.dbref">
                <template v-slot:activator="{ on }">
                    <markercity
                        v-on="on"
                        :ref="marker.dbref"
                        v-show="showLands"
                        :marker="marker"
                        :map="map"
                        align="bottom"
                        />
                </template>
                <span v-html="marker.tooltip"></span>
            </v-tooltip>
        </div>
    </div>
</template>

<script>
import db from '@/firebase/init'
import toolbar from './toolbar.vue'
import drawer from './drawer.vue'
import markercity from './marker-city.vue'


export default {
    name: 'mainmap',
    components: {
        toolbar,
        drawer,
        markercity,
    },
    data: () => ({
        drawer: false,
        map: null,
        zoom: 2,
        hasRestriction: false,
        hasMap: false,
        maxZoom: 5,
        minZoom: 1,
        lat: 20,
        lng: 125,
        mapType: 'Valucre',
        isHarsh: false,
        limitBounds: {
            north: 80,
            east: -140,
            south: -60,
            west: 30,
        },
        defBounds: {
            north: 80,
            east: -140,
            south: -60,
            west: 30,
        },
        center: {lat: 20, lng: 125},
        markers: [],
    }),
    computed: {
        app() {
            return this.$root.$children[0];
        },
        selectedMarker() {
            return this.markers.find((marker) => {
                return marker.active;
            });
        },
        showCities() {
            if (/xs|sm/.test(this.$vuetify.breakpoint.name)) {
                if (this.zoom >= 2)
                    return this.app.$refs.toolbar.showmarkers.includes(1);
                else
                    return false;
            } else {
                if (this.zoom > 2)
                    return this.app.$refs.toolbar.showmarkers.includes(1);
                else
                    return false;
            }
        },
        citymarkers() {
            return this.markers.filter((marker) => {
                return /city/i.test(marker.type);
            })
        },
        showTowns() {
            if (/xs|sm/.test(this.$vuetify.breakpoint.name)) {
                if (this.zoom >= 3)
                    return this.app.$refs.toolbar.showmarkers.includes(2);
                else
                    return false;
            } else {
                if (this.zoom > 3)
                    return this.app.$refs.toolbar.showmarkers.includes(2);
                else
                    return false;
            }
            
        },
        townmarkers() {
            return this.markers.filter((marker) => {
                return /town/i.test(marker.type);
            })
        },
        showLands() {
            if (/xs|sm/.test(this.$vuetify.breakpoint.name)) {
                if (this.zoom >= 3)
                    return this.app.$refs.toolbar.showmarkers.includes(3);
                else
                    return false;
            } else {
                if (this.zoom > 3)
                    return this.app.$refs.toolbar.showmarkers.includes(3);
                else
                    return false;
            }
        },
        landmarkers() {
            return this.markers.filter((marker) => {
                return /land/i.test(marker.type);
            })
        },
    },
    created() {
        db.collection('Terrenus').get()
        .then(snapshot => {
            console.log(snapshot);
            snapshot.docs.forEach(doc => {
                let marker = doc.data();
                this.markers.push({
                    board: marker.board,
                    dbref: marker.dbref,
                    desc: marker.desc,
                    lat: marker.lat,
                    lng: marker.lng,
                    latlng: new google.maps.LatLng({
                        lat: marker.lat,
                        lng: marker.lng,
                    }),
                    link: marker.link,
                    tags: marker.tags,
                    title: marker.title,
                    tooltip: marker.tooltip,
                    type: marker.type,
                    hover: false,
                    active: false,
                })
            })
            console.log(this.markers)
        })
    },
    mounted() {
        // console.log('Hello world');
        this.initMap();
        window.addEventListener('resize', this.automateBounds);
        this.automateBounds();
    },
    methods: {
        panToSelectedMarker() {
            this.map.panTo({
                lat: this.selectedMarker.lat,
                lng: this.selectedMarker.lng,
            })
        },
        resetBounds() {
            if (this.hasRestriction) {
                this.map.setOptions({
                    restriction: {
                        latLngBounds: this.limitBounds,
                        strictBounds: this.isHarsh,
                    },
                })
            }
        },
        automateBounds() {
            if (/sm|xs/.test(this.$vuetify.breakpoint.name)) {
                if (window.matchMedia("(orientation: portrait)").matches) {
                    this.limitBounds.east = -175;
                    this.limitBounds.west = 60;
                    this.limitBounds.north = 80;
                    this.limitBounds.south = -60;
                    
                } else if (window.matchMedia("(orientation: landscape)").matches) {
                    this.limitBounds.east = -155;
                    this.limitBounds.west = 40;
                    this.limitBounds.north = 60;
                    this.limitBounds.south = -30;
                }
                this.map.setOptions({
                    minZoom: 1
                })
            } else {
                this.limitBounds = this.defBounds;
                this.map.setOptions({
                    minZoom: 2
                })
            }
                this.resetBounds();
        },
        initMap() {
            this.map = new google.maps.Map(
                document.getElementById('map'), {
                    zoom: this.zoom,
                    maxZoom: this.maxZoom,
                    minZoom: this.minZoom,
                    center: this.center,
                    zoomControl: true,
                    zoomControlOptions: {
                        position: google.maps.ControlPosition.LEFT_BOTTOM
                    },
                    mapTypeControl: false,
                    disableDoubleClickZoom: true,
                    mapTypeControlOptions: {
                        mapTypeIds: ['Valucre', 'roadmap']
                    },
                    scaleControl: true,
                    gestureHandling: 'greedy',
                    streetViewControl: false,
                    rotateControl: false,
                    fullscreenControl: false,
                }
            );
            if (this.hasRestriction) {
                this.map.setOptions({
                    restriction: {
                        latLngBounds: this.limitBounds,
                        strictBounds: this.isHarsh,
                    },
                })
            }
            this.setUpMap();
        },
        setUpMap() {
            // console.log(google.maps.MapRestriction);
            const self = this;
            var valMapType = new google.maps.ImageMapType({
                getTileUrl: function(coord, zoom) {
                    var normalizedCoord = self.getNormalizedCoord(coord, zoom);
                    if (!normalizedCoord) { return null; }
                    var bound = Math.pow(2, zoom);
                    // return `./valucremap/${zoom}-${normalizedCoord.x}-${normalizedCoord.y}.png`
                    let isMax = zoom == 5;
                    if (!isMax) {
                        return `https://boring-hodgkin-e172d7.netlify.com/default/${zoom}/${zoom}-${normalizedCoord.x}-${normalizedCoord.y}.png`
                    } else {
                        let direction = normalizedCoord.x > 16 ? true : false;
                        return `https://boring-hodgkin-e172d7.netlify.com/default/${zoom}/${direction ? 'B' : 'A'}/${zoom}-${normalizedCoord.x}-${normalizedCoord.y}.png`
                    }
                    // return `https://boring-hodgkin-e172d7.netlify.com/valucremap/${zoom}-${normalizedCoord.x}-${normalizedCoord.y}.png`
                },
                tileSize: new google.maps.Size(256, 256),
                maxZoom: self.maxZoom,
                minZoom: self.minZoom,
                radius: 1738000,
                name: 'Valucre'
            });
            self.map.mapTypes.set('Valucre', valMapType);
            self.map.setMapTypeId(self.mapType);
            // console.log('set map')
            self.map.setOptions({ disableDefaultUI: true })
            

            google.maps.event.addListener(this.map, 'click', (event) => {
                // self.addCustomMarker(event);
            });
            google.maps.event.addListener(this.map, 'zoom_changed', () => {
                self.zoom = self.map.getZoom();
            });
            google.maps.event.addListener(this.map, 'maptypeid_changed', () => {
                // self.mapType = self.map.getMapTypeId();
            });
            google.maps.event.addListener(this.map, 'idle', function() {
                // self.app.setZoom(self.map.getZoom());
                // self.app.setLatLng(self.map.getCenter());
                // self.app.changePoint(self.map.getCenter());
                // console.log(`${self.app.activeBoard}`)
                console.log(`${self.map.getCenter().lat().toFixed(3)} : ${self.map.getCenter().lng().toFixed(3)}`)
                let currentview = self.map.getBounds();
                // console.log(currentview)
            });

            this.hasMap = true;
            this.app.mainmap = this;
            if (this.$route.params.pathMatch) {
                // console.log(this.$route.params.pathMatch)
                // this.decodeRoute(this.$route.params.pathMatch);
            } else {
                // console.log(this.$route)
                // this.app.setLatLng(this.map.getCenter());
            }
        },
        
        getNormalizedCoord(coord, zoom) {
            var y = coord.y, x = coord.x;
            var tileRange = 1 << zoom;
            if (y < 0 || y >= tileRange) { return null; }
            if (x < 0 || x >= tileRange) { x = (x % tileRange + tileRange) % tileRange; }
            return {x: x, y: y};
        },
    }
}
</script>

<style>
  #map {
      /* margin-top: 48px; */
      height: calc(100vh - 48px);
      width: 100%;
    }
</style>