<template>
    <div class="content">
        <div id="map"
            @mouseenter="cursorOnMap()"
            @mouseleave="cursorOffMap()"
        ></div>
        <div class="titleAlert">
            <v-alert
                class="pa-3"
                v-model="hasTitleAlert"
                dismissible
                type="info"
                >
                    This map is in beta
                </v-alert>
        </div>
        <crosshair />
        <div v-if="hasMap" id="mapOverlay">
            <markercity
                v-for="marker in landmarkers"
                :key="marker.dbref"
                :ref="marker.dbref"
                v-show="showLands"
                :marker="marker"
                :map="map"
                align="bottom"
                />
            <markercity
                v-for="marker in townmarkers"
                :key="marker.dbref"
                :ref="marker.dbref"
                v-show="showTowns"
                :marker="marker"
                :map="map"
                align="bottom"
                />
            <markercity
                v-for="marker in citymarkers"
                :key="marker.dbref"
                :ref="marker.dbref"
                v-show="marker.selected || showCities"
                :marker="marker"
                :map="map"
                align="bottom"
                />
        </div>
        <div id="overlay"></div>
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
        inMap: false,
        drawer: false,
        crossElt: null,
        map: null,
        zoom: 2,
        starting: true,
        hasTitleAlert: false,
        isAdding: false,
        wasScrolling: false,
        crosshairType: 'default',
        hasRestriction: false,
        hasMap: false,
        maxZoom: 6,
        minZoom: 0,
        lat: 20,
        lng: 125,
        mapType: 'Valucre',
        activeBoard: null,
        hovMarker: null,
        selMarker: null,
        boards: [
            'Terrenus',
            'Orisia',
            'Renovatio',
            'Alterion',
            'Genesaris',
        ],
        isHarsh: false,
        fitToBoard: false,
        bounds: {
            Terrenus: {
                north: 60,
                west: 75,
                east: 180,
                south: -30,
                ne: null,
                sw: null,
                area: null,
                center: null,
            },
            Renovatio: {
                north: -9,
                west: 27,
                east: 75,
                south: -37,
                ne: null,
                sw: null,
                area: null,
                center: null,
            },
            Alterion: {
                north: -27,
                west: -6,
                east: 27,
                south: -48,
                ne: null,
                sw: null,
                area: null,
                center: null,
            },
            Genesaris: {
                north: 15,
                west: 175,
                east: -35,
                south: -55,
                ne: null,
                sw: null,
                area: null,
                center: null,
            },
            Orisia: {
                north: -6,
                west: -40,
                east: -20,
                south: -20,
                ne: null,
                sw: null,
                area: null,
                center: null,
            },
        },
        mapLoaded: false,
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
        center: {lat: 20, lng: -125},
        markers: [],
        hideCities: false,
        hideTowns: false,
        hideLands: false,
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
        hoveredMarker() {
            return this.markers.find((marker) => {
                return marker.hover;
            });
        },
        markerDBsInActiveBoard() {
            let actives = this.markers.filter((marker) => {
                return marker.board == this.activeBoard;
            })
            let mirror = [];
            actives.forEach(val => {
                mirror.push(val.dbref);
            })
            return mirror;
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
        this.starting = true;
        let boards = ['Terrenus', 'Orisia', 'Genesaris', 'Renovatio', 'Alterion']
        boards.forEach(board => {
            let ref = db.collection(board)
                ref.onSnapshot(snapshot => {
                    snapshot.docChanges().forEach(change => {
                        let marker = change.doc.data()
                        // console.log(change.type)
                        if (/added|modified/i.test(change.type)) {
                            if (/modified/i.test(change.type)) {
                                let edited = this.markers.find((targ) => {
                                    return marker.dbref == targ.dbref;
                                })
                                edited = {
                                    board: marker.board,
                                    contactLink: marker.contactLink,
                                    contactAvatar: marker.contactAvatar,
                                    annoElt: null,
                                    labelElt: null,
                                    dbref: marker.dbref,
                                    desc: marker.desc,
                                    lat: marker.lat,
                                    lng: marker.lng,
                                    image: marker.image,
                                    details: marker.details,
                                    hide: false,
                                    minZoom: marker.minZoom,
                                    maxZoom: marker.maxZoom,
                                    alert: marker.alert,
                                    latlng: new google.maps.LatLng({
                                        lat: marker.lat,
                                        lng: marker.lng,
                                    }),
                                    link: marker.link,
                                    tags: marker.tags,
                                    title: marker.title,
                                    // tooltip: marker.tooltip,
                                    type: marker.type,
                                    hover: false,
                                    active: false,
                                }
                                console.log(`${marker.title} was edited`)
                                console.log(edited);
                            } else {

                                this.markers.push({
                                    board: marker.board,
                                    contactLink: marker.contactLink,
                                    contactAvatar: marker.contactAvatar,
                                    dbref: marker.dbref,
                                    desc: marker.desc,
                                    lat: marker.lat,
                                    lng: marker.lng,
                                    image: marker.image,
                                    minZoom: marker.minZoom,
                                    maxZoom: marker.maxZoom,
                                    details: marker.details,
                                    alert: marker.alert,
                                    latlng: new google.maps.LatLng({
                                        lat: marker.lat,
                                        lng: marker.lng,
                                    }),
                                    link: marker.link,
                                    tags: marker.tags,
                                    title: marker.title,
                                    type: marker.type,
                                    // tooltip: marker.tooltip,
                                    annoElt: null,
                                    labelElt: null,
                                    hide: false,
                                    hover: false,
                                    active: false,
                                })
                            }
                        } else if (/removed/i.test(change.type)) {
                            this.markers = this.markers.filter((targ) => {
                                return marker.dbref !== targ.dbref;
                            })
                        }
                        // console.log('Tier 3')
                    })
                    // console.log('Tier 2')
                })
            // console.log('Tier 1')
        })
        // console.log('Tier 0')
        setTimeout(() => {
            // console.log(this.markers)
            // console.log('Checking route')
            if (this.$route.params.pathMatch)
                console.log('Decode now')
                this.decodeStartPos(this.$route.params.pathMatch);
        }, 1600);
    },
    mounted() {
        // console.log('Hello world');
        this.initMap();
        // this.createCanvas();
        window.addEventListener('resize', this.automateBounds);
        this.automateBounds();
        this.constructBoardBounds();
        this.crossElt = document.getElementById('crossHair');
        this.$el.addEventListener('touchstart', this.handleTouch)
    },
    methods: {
        handleTouch(evt) {
            this.inMap = true;
            // console.log('Touching')
        },
        panToPoint(point) {
            this.map.panTo(point);
        },
        cursorOnMap() {
            // console.log('Entering map');
            this.inMap = true;
        },
        cursorOffMap() {
            // console.log('Leaving map')
            this.inMap = false;
        },
        getMarkerDesc(marker) {
            // let test = new google.maps.OverlayView();
            // this.createCanvas(marker);
            // console.log(test)
            console.log(marker)
        },
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
                board.center = board.area.getCenter();
            })
            console.log('Bounds have been created')
            console.log(this.bounds);
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
            const self = this;
            // if (/xs|sm/i.test(this.$vuetify.breakpoint.name)) {
            //     if (/xs/i.test(this.$vuetify.breakpoint.name)) {
            //         let check = this.map.getBounds();
            //         self.map.panTo({
            //             // lat: check.ma.l,
            //             lat: self.selectedMarker.lat,
            //             lng: self.selectedMarker.lng,
            //         })
            //     } else {
            //         let check = this.map.getBounds();
            //         self.map.panTo({
            //             lat: check.ma.l,
            //             lng: self.selectedMarker.lng,
            //         })

            //     }
            // } else {
                this.map.panTo({
                    lat: this.selectedMarker.lat,
                    lng: this.selectedMarker.lng,
                })
            // }
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
            // console.log(`Center on creation is [${this.center.lat}, ${this.center.lng}]`)
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
                    // disableDoubleClickZoom: true,
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
            self.map.setOptions({ disableDefaultUI: true })
            this.hasMap = true;

            google.maps.event.addListener(this.map, 'click', (event) => {
                // deselection should occur here
                if (/xs|sm/.test(this.$vuetify.breakpoint.name))
                    self.inMap = true;
            });
            google.maps.event.addListener(this.map, 'center_changed', () => {
                self.assignCenter();
                self.wasScrolling = true
                setTimeout(() => {
                    self.wasScrolling = false;
                }, 800);
                // console.log(simplePos);

            });
            google.maps.event.addListener(this.map, 'zoom_changed', () => {
                self.zoom = self.map.getZoom();
            });
            google.maps.event.addListener(this.map, 'maptypeid_changed', () => {
                // self.mapType = self.map.getMapTypeId();
            });
            
            google.maps.event.addListener(this.map, 'idle', function() {
                self.assignCenter();
                self.checkAnnoBox();
                
            });

            this.mapLoaded = true;
            this.app.mainmap = this;
            
        },
        resetHovers() {
            this.markers.forEach(marker => {
                marker.hover = false;
            })
            const refList = this.$refs;
            this.markers.forEach(test => {
                refList[test.dbref][0].resetHover();
            })
        },
        resetHidden() {
            this.markers.forEach(marker => {
                marker.hide = false;
            })
            console.log('Unhiding all')
        },
        resetActivesExcept(ref) {
            // console.log(`Reset all except ${ref}`)
            let temp = this.markers.filter(marker => {
                return marker.dbref !== ref;
            })
            // console.log(temp)
            const refList = this.$refs;
            temp.forEach(mark => {
                // console.log(`Reset ${mark.title}`)
                refList[mark.dbref][0].active = false;
                refList[mark.dbref][0].resetActive();
                mark.active = false;
            })
            // this.markers.forEach(marker => {
            //     marker.active = false;
            // })
        },
        resetAllCalcDistance() {
            const refList = this.$refs;
            this.markers.forEach(test => {
                refList[test.dbref][0].resetDistance();
            })
        },
        resetAllScales() {
            const refList = this.$refs;
            this.markers.forEach(test => {
                refList[test.dbref][0].resetScale();
            })
        },
        checkHasCollisions(ref) {
            // setTimeout(() => {
            //     console.log('Forcing collision check')
            //     this.checkAnnoBox();
            // }, 200);
            // const refList = this.$refs;
            // refList[ref][0].checkForCollisionOfAnnoBox();
        },
        assignCenter() {
            const self = this;
            let realPos = new google.maps.LatLng({
                lat: self.map.getCenter().lat(),
                lng: self.map.getCenter().lng(),
            });
            this.simplePos = {
                lat: +realPos.lat().toFixed(3),
                lng: +realPos.lng().toFixed(3),
            };
            this.realLatLng = realPos;
            this.findBoardInCurrentView();
            const refList = this.$refs;
            this.markerDBsInActiveBoard.forEach(test => {
                refList[test][0].checkForCollision();
            })
            let checkHover = this.markers.find(marker => {
                return marker.hover == true;
            })
            let checkActive = this.markers.find(marker => {
                return marker.active == true;
            })
            if (!checkActive || !checkHover) {
                // console.log('Repaint')
                this.markers.forEach(marker => {
                    marker.hide = false;
                })
            }
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
        checkAnnoBox() {
            const refList = this.$refs;
            this.markers.forEach(mark => {
                refList[mark.dbref][0].checkForCollisionOfAnnoBox();
            })
        },
        doInitAction() {
            // console.log('Initializing');
            const refList = this.$refs;
            this.markers.forEach(mark => {
                // console.log(refList[mark]);
                refList[mark.dbref][0].assignAnnoElt();
            })
            if (this.$route.params.pathMatch) {
                console.log(this.$route.params.pathMatch)
                this.decodeRoute(this.$route.params.pathMatch);
            }
        },
        decodeStartPos(str) {
            const self = this;
            let rx = {
                select: /select\=[^&]*&/,
                hover: /hover\=[^&]*&/,
                board: /board\=[^0-9]*&/,
                z: /z[\d](toz[\d])?/,
                lat: /lat[^a-zA-Z]*/,
                lng: /lng[^a-zA-Z]*/,
                key: /key\=[^&]*&/,
            };
            if (rx.key.test(str)) {
                let result = str.match(rx.key)[0];
                result = result.replace('key\=', '').replace('\&', '');
                // console.log(`Trying to unlock with ${result}`);
                db.collection('keys').where('key', '==', result).get()
                    .then(snapshot => {
                        // try {
                            if (snapshot.docs.length) {
                                let unlocked = snapshot.docs[0];
                                let id = unlocked.id;
                                self.app.permissions.push(id);
                                console.log(`Unlocked ${id} permissions`);
                            } else {
                                console.log('Failed')
                                self.app.permissions = [];
                            }
                    })

            }
            if (rx.board.test(str)) {
                // console.log('Fit to board');
                this.fitToBoard = true;
                let result = str.match(rx.board)[0];
                result = result.replace('board\=', '').replace('\&', '');
                console.log(`Fit to ${result}`)
                let active = this.bounds[result];
                if (this.mapLoaded) {
                    // console.log(`Setting center to ${active.center}`)
                    this.map.setCenter(active.center);
                } else {
                    this.center = active.center;
                }
            } else {
                if ((rx.lat.test(str)) && (rx.lng.test(str))) {
                    if ((!rx.hover.test(str)) && (!rx.hover.test(str))) {
                        let resultLat = str.match(rx.lat)[0];
                        let resultLng = str.match(rx.lng)[0];
        
                        this.center.lat = +resultLat.replace('lat','');
                        this.center.lng = +resultLng.replace('lng','');
                        console.log(`Changed map center to [${resultLat}, ${resultLng}]`)
                        this.map.setCenter(this.center);
                    } else {
                        console.log(`Start pos overridden by hover or active`)
                    }
                }
                if (rx.z.test(str)) {
                    let result = str.match(rx.z)[0];
                    if (/to/i.test(result)) {
                        result = result.split('to');
                        result = +result[0].replace('z', '');
                    } else {
                        result = +result.replace('z', '')
                    }
                    this.zoom = result;
                    this.map.setZoom(this.zoom)
                    // console.log(`Changed zoom to ${result}`)
                }
                if (rx.select.test(str)) {
                    let result = str.match(rx.select)[0];
                    result = result.replace('select\=', '').replace('\&', '');
                    // console.log('Change select');
                    let temp = this.markers.find(marker => {
                        return marker.dbref == result;
                    })
                    // temp.active = true;
                    // console.log(`Change coords to selection: ${result}`)
                    this.center.lat = temp.lat;
                    this.center.lng = temp.lng;
                    // console.log(`${temp.lat}, ${temp.lng}`)
                    if (rx.hover.test(str)) {
                        temp.hover = false;
                    }
                    if (this.mapLoaded) {
                        // console.log('Map is already loaded')
                        this.map.setCenter({
                            lat: temp.lat,
                            lng: temp.lng,
                        })
                        this.$refs[temp.dbref][0].clickOn();
                        this.selMarker = temp;
                    } else {
                        // console.log(`Center should be [${this.center.lat}, ${this.center.lng}]`);
                    }
                    if (!rx.hover.test(str)) {
                        // console.log('Should not pan to hovered')
                    }
                }
                if (rx.hover.test(str)) {
                    let result = str.match(rx.hover)[0];
                    result = result.replace('hover\=', '').replace('\&', '');
                    let temp = this.markers.find(marker => {
                        return marker.dbref == result;
                    })
                    temp.hover = true;
    
                    this.center.lat = temp.lat;
                    this.center.lng = temp.lng;
                    this.hovMarker = temp;
                    if (this.mapLoaded) {
                        if (!rx.select.test(str)) {
                            this.map.setCenter({
                                lat: temp.lat,
                                lng: temp.lng,
                            })
                            // this.$refs[temp.dbref][0].clickOn();
                        }
                    } else {
                        // console.log(`Center should be [${this.center.lat}, ${this.center.lng}]`);
                    }
                }
            }

        },
        decodeRoute(str) {
            const self = this;
            let rx = {
                z: /z[\d](toz[\d])?/,
                select: /select\=[^&]*&/,
                hover: /hover\=[^&]*&/,
                board: /board\=[^0-9]*&/,
            };
            if (rx.board.test(str)) {
                let match = str.match(rx.board)[0];
                match = match.replace('board\=', '').replace('\&', '');
                console.log(`Fitting to ${match}`)
                setTimeout(() => {
                    self.map.fitBounds(self.bounds[match].area);
                }, 600);
            } else {
                let match = ['z']
                match.forEach(param => {
                    if (rx[param].test(str)) {
                        let result = str.match(rx[param]);
                        let results = result;
                        result = result[0];
                        if (/z/.test(param)) {
                            if (/to/i.test(result)) {
                                results = result.split('to');
                                setTimeout(() => {
                                    // console.log(`Zoom in again to ${+results[1].replace('z', '')}`)
                                    if (rx.select.test(str)) {
                                        // console.log(`Start on ${self.selectedMarker.title}`)
                                        if (rx.hover.test(str)) {
                                            self.map.panTo(this.hovMarker.latlng);
                                        }
                                    }
                                    self.setZoom(+results[1].replace('z', ''))
                                }, 600);
                            } else {
                                // console.log(`Map is already at ${+results[0].replace('z', '')}`)
                            }
                        }
                    }
                })
            }

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

    .titleAlert {
        position: absolute;
        /* top: 48px; */
        top: 0px;
        width: 100%;
        /* border: 2px solid red; */
    }

</style>