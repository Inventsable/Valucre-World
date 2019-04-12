<template>
    <div class="content">
        <div id="map"></div>
        <crosshair />
        <div v-if="hasMap">
            <v-tooltip
                v-model="marker.hover" bottom
                v-for="marker in citymarkers"
                :key="marker.dbref">
                <template v-slot:activator="{ on }">
                    <markercity
                        v-on="on"
                        :ref="marker.dbref"
                        v-show="marker.selected || showCities"
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
import crosshair from './crosshair.vue'


export default {
    name: 'mainmap',
    components: {
        toolbar,
        crosshair,
        drawer,
        markercity,
    },
    data: () => ({
        drawer: false,
        map: null,
        zoom: 2,
        isAdding: false,
        crosshairType: 'default',
        hasRestriction: false,
        hasMap: false,
        maxZoom: 6,
        minZoom: 1,
        lat: 20,
        lng: 125,
        mapType: 'Valucre',
        activeBoard: null,
        boards: [
            'Terrenus',
            'Orisia',
            'Renovatio',
            'Alterion',
            'Genesaris',
        ],
        isHarsh: false,
        bounds: {
            Terrenus: {
                north: 60,
                west: 75,
                east: 175,
                south: -30,
                ne: null,
                sw: null,
                area: null,
            },
            Renovatio: {
                north: -9,
                west: 27,
                east: 75,
                south: -37,
                ne: null,
                sw: null,
                area: null,
            },
            Alterion: {
                north: -27,
                west: -6,
                east: 27,
                south: -48,
                ne: null,
                sw: null,
                area: null,
            },
            Genesaris: {
                north: 15,
                west: 175,
                east: -35,
                south: -55,
                ne: null,
                sw: null,
                area: null,
            },
            Orisia: {
                north: -8,
                west: -35,
                east: -20,
                south: -20,
                ne: null,
                sw: null,
                area: null,
            },
        },
        realLatLng: null,
        simplePos: {
            lat: null,
            lng: null,
        },
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
        let boards = ['Terrenus', 'Orisia', 'Genesaris', 'Renovatio', 'Alterion']
        boards.forEach(board => {
            db.collection(board).get()
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
                // console.log(this.markers)
            })
        })
    },
    mounted() {
        // console.log('Hello world');
        this.initMap();
        window.addEventListener('resize', this.automateBounds);
        this.automateBounds();
        this.constructBoardBounds();
    },
    methods: {
        constructBoardBounds() {
            this.boards.forEach(name => {
                let board = this.bounds[name];
                board.sw = new google.maps.LatLng({
                    lat: board.south,
                    lng: board.west,
                });
                board.ne = new google.maps.LatLng({
                    lat: board.north,
                    lng: board.east,
                });
                board.area = new google.maps.LatLngBounds(board.sw, board.ne);
            })
        },
        findBoardInCurrentView() {
            // console.log(`lat: ${this.simplePos.lat}, lng: ${this.simplePos.lng}`);
            let count = -1, match = false;
            this.boards.forEach(name => {
                count++;
                if (this.bounds[name].area.contains(this.realLatLng)) {
                    // console.log(`${name} is in view`)
                    match = true;
                    this.activeBoard = name;
                }
            })
            if (!match) {
                this.activeBoard = 'World';
            }
            // console.log(`Active board is ${this.activeBoard}`);
        },
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
                    // let isMax = zoom == 5;
                    if (zoom < 5) {
                        return `https://boring-hodgkin-e172d7.netlify.com/default/${zoom}/${zoom}-${normalizedCoord.x}-${normalizedCoord.y}.png`
                    } else if (zoom == 5) {
                        let direction = normalizedCoord.x > 16 ? true : false;
                        return `https://boring-hodgkin-e172d7.netlify.com/default/${zoom}/${direction ? 'B' : 'A'}/${zoom}-${normalizedCoord.x}-${normalizedCoord.y}.png`
                    } else if (zoom == 6) {
                        let direction = null;
                        if (normalizedCoord.x < 14) {
                            direction = 1;
                        } else if (normalizedCoord.x < 27) {
                            direction = 2;
                        } else if (normalizedCoord.x < 40) {
                            direction = 3;
                        } else if (normalizedCoord.x < 53) {
                            direction = 4;
                        } else {
                            direction = 5;
                        }
                        return `https://boring-hodgkin-e172d7.netlify.com/default/${zoom}/${direction}/${zoom}-${normalizedCoord.x}-${normalizedCoord.y}.png`
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
                // console.log(`${}`)

                
                
                let realPos = new google.maps.LatLng({
                    lat: self.map.getCenter().lat(),
                    lng: self.map.getCenter().lng(),
                });
                self.simplePos = {
                    lat: +realPos.lat().toFixed(3),
                    lng: +realPos.lng().toFixed(3),
                };
                self.realLatLng = realPos;
                
                // console.log(`${}`)
                // console.log(self.simplePos);
                // let currentview = self.map.getBounds();
                self.findBoardInCurrentView();
                // console.log(currentview)
            });

            this.hasMap = true;
            this.app.mainmap = this;
            
        },
        setZoom(result) {
            if (result <= this.maxZoom) {
                this.zoom = result;
                this.map.setZoom(result);
                // console.log(`Zoom set to ${result}`);
            }
        },
        setLatLng(...args) {
            if (args.length) {
                this.map.panTo(args[0]);
            } else {
                this.map.panTo({
                    lat: this.lat,
                    lng: this.lng,
                })
            }
        },
        getNormalizedCoord(coord, zoom) {
            var y = coord.y, x = coord.x;
            var tileRange = 1 << zoom;
            if (y < 0 || y >= tileRange) { return null; }
            if (x < 0 || x >= tileRange) { x = (x % tileRange + tileRange) % tileRange; }
            return {x: x, y: y};
        },
        doInitAction() {
            if (this.$route.params.pathMatch) {
                console.log(this.$route.params.pathMatch)
                this.decodeRoute(this.$route.params.pathMatch);
            }
        },
        decodeRoute(str) {
            const self = this;
            let rx = {
                z: /z[\d](toz[\d])?/,
                lat: /lat[^a-zA-Z]*/,
                lng: /lng[^a-zA-Z]*/,
                ref: /ref\=.*\&/,
                info: /info/,
                points: /p[a-z][^a-z]*/gm,
                maptype: /type\=.*/,
                km: /km/,
                mi: /mi/,
            };
            let match = ['z', 'lat', 'lng', 'ref', 'info', 'points', 'maptype', 'km', 'mi']
            let changecoords = false;
            match.forEach(param => {
                if (rx[param].test(str)) {
                    let result = str.match(rx[param]);
                    // if (/points/.test(param)) {
                    //     // this.addPoints(result);
                    //     let mirror = [];
                    //     let pack = {};
                    //     result.forEach((arg, v) => {
                    //         if ((!this.isOdd(v)) || (v == 0)) {
                    //             pack = {};
                    //             // console.log(`lat: ${+arg.substring(2)}`)
                    //             pack['lat'] = +arg.substring(2);
                    //         } else {
                    //             // console.log(`lng: ${+arg.substring(2)}`)
                    //             pack['lng'] = +arg.substring(2);
                    //             mirror.push(pack);
                    //         }
                    //     })
                    //     this.points = mirror;
                    //     this.checkDistanceOfPath();
                    //     this.app.showDist = true;
                    //     this.app.enableDistance();
                    //     // this.getPathDistance(mirror, false);
                    // } else {
                        let results = result;
                        result = result[0];
                        // console.log(result);
                        if (/z/.test(param)) {
                            // console.log(result)
                            // console.log(results)
                            if (/to/i.test(result)) {
                                results = result.split('to');
                                // console.log(`Zoom in initially to ${results[0]}`)
                                this.setZoom(+results[0].replace('z', ''))
                                setTimeout(() => {
                                    // console.log(`Zoom in again to ${results[1]}`)
                                    this.setZoom(+results[1].replace('z', ''))
                                }, 1000);
                            } else {
                                this.setZoom(+result.replace('z', ''))
                            }
                        }
                        if (/lat|lng/.test(result)) {
                            changecoords = true;
                            self[param] = +result.replace(param, '');
                        }
                        if (/ref/.test(param)) {
                            // console.log('ref result')
                            setTimeout(() => {
                                console.log(result)
                                // self.setRef(result);
                            }, 500);
                        }
                        // if (/info/.test(param))
                        //     this.app.$refs.lore.drawer = true;
                        // if (/maptype/.test(param)) {
                        //     this.setMapTypeFromURL(result);
                        // }
                        // if (/(km|mi)/.test(param)) {
                        //     // @@
                        //     // console.log(`Set units to ${result}`)
                        //     self.$refs.distancebar.setSelection(result);
                        //     self.$root.$children[0].unitToMeasure = result;
                        // }
                    // }
                    console.log(result)
                    
                }
            })
            if (changecoords)
                this.setLatLng();
        },
        isOdd(num) {
            return num % 2;
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