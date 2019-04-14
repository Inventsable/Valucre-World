<template>
    <v-layout row>
        <v-flex xs12>
        <!-- <v-card> -->
            <!-- <v-toolbar color="teal" dark>
            <v-toolbar-side-icon></v-toolbar-side-icon>

            <v-toolbar-title>Topics</v-toolbar-title>

            <v-spacer></v-spacer>

            <v-btn icon>
                <v-icon>more_vert</v-icon>
            </v-btn>
            </v-toolbar> -->

            <v-list class="pa-0">
                <v-list-group
                    v-for="item in markertypes"
                    :key="item.title"
                    v-model="item.active"
                    no-action
                >
                    <template v-slot:activator>
                        <v-list-tile>
                            <v-list-tile-content>
                                <v-list-tile-title class="text-uppercase typeTitle">{{ item.title }}</v-list-tile-title>
                            </v-list-tile-content>
                        </v-list-tile>
                    </template>

                    <v-list-tile
                        v-for="subItem in item.markers"
                        :key="subItem.title"
                        @click="goToMarker(subItem)"
                        >
                        <div class="prependEdit">
                            <v-btn icon 
                                class="prependEdit"
                                @click="editMarker(subItem)"
                                >
                                    <v-icon color="grey darken-1">edit</v-icon>
                                </v-btn>
                        </div>
                        <v-list-tile-content>
                            <v-list-tile-title class="markerTitle">{{ subItem.title }}</v-list-tile-title>
                        </v-list-tile-content>

                        <v-list-tile-action>
                            <v-btn icon 
                            @click="selectMarker(subItem)"
                            >
                                <v-icon color="grey darken-1">{{checkStatus(subItem)}}</v-icon>
                            </v-btn>
                        </v-list-tile-action>
                    </v-list-tile>
                </v-list-group>
            </v-list>
        <!-- </v-card> -->
        </v-flex>
    </v-layout>
</template>

<script>
export default {
    name: 'drawermarkers',
    data: () => ({
        markertypes: [
            {
                name: 'city',
                icon: 'location_city',
                title: 'Cities',
                value: false,
                show: true,
                markers: [],
                foci: [true],
            },
            {
                name: 'town',
                icon: 'store',
                title: 'Towns',
                value: false,
                show: false,
                markers: [],
                foci: [false],
            },
            {
                name: 'land',
                icon: 'terrain',
                title: 'Landmarks',
                value: false,
                show: true,
                markers: [],
                foci: [false],
            },
        ],
    }),
    computed: {
        app() {
            return this.$root.$children[0];
        },
        citymarkers() {
            if (this.app.isLoaded && this.app.mainmap.hasMap) {
                return this.app.mainmap.citymarkers.filter(item => {
                    return item.board == this.app.mainmap.activeBoard;
                });
            }
        },
        townmarkers() {
            if (this.app.isLoaded && this.app.mainmap.hasMap) {
                return this.app.mainmap.townmarkers.filter(item => {
                    return item.board == this.app.mainmap.activeBoard;
                });
            }
        },
        landmarkers() {
            if (this.app.isLoaded && this.app.mainmap.hasMap) {
                return this.app.mainmap.landmarkers.filter(item => {
                    return item.board == this.app.mainmap.activeBoard;
                });
            }
        },
        activeBoard() {
             if (this.app.isLoaded && this.app.mainmap.hasMap) {
                return this.app.mainmap.activeBoard;
            }
        },
        
        map() {
            if (this.app.isLoaded) {
                return this.app.mainmap.map;
            }
        }
    },
    watch: {
        activeBoard(state) {
            this.reShuffleMarkers();
        }
    },
    methods: {
        canEdit(marker) {
            if (marker.active) {
                if (this.hasPermissions())
                    return true;
                else
                    return false;
            } else {
                return false;
            }
        },
        checkEditStatus(marker) {
            if (this.canEdit(marker))
                return 'edit';
            else
                return 'my_location';
        },
        checkStatus(marker) {
            if (marker.active)
                return 'clear';
            else
                return 'my_location';
        },
        editMarker(marker) {
            console.log(`Edit ${marker.title}`)
        },
        hasPermissions() {
            if (this.app.isLoaded) {
                console.log('Checking permissions');
                if (this.app.permissions.includes('World')) {
                    return true;
                } else if (this.app.permissions.includes(this.app.mainmap.activeBoard)) {
                    return true;
                } else {
                    return false;
                }
            }
        },
        selectMarker(marker) {
            if (!marker.active) {
                console.log('Selecting marker');
                this.app.mainmap.resetActivesExcept(marker.dbref);
                this.app.mainmap.$refs[marker.dbref][0].clickOn();
            } else {
                this.app.mainmap.$refs[marker.dbref][0].clickOn()
            }
            
        },
        goToMarker(marker) {
            const self = this;
            // if (marker.active) {
            console.log(`Travel to ${marker.title}`);
            if (this.app.mainmap.zoom < marker.minZoom) {
                this.map.setZoom(+marker.minZoom);
            }
            this.map.panTo(marker.latlng);
            // }
            setTimeout(() => {
                console.log('Forcing check')
                self.app.mainmap.checkAnnoBox();
            }, 1000);
            // this.app.mainmap.checkHasCollisions(marker.dbref);
        },
        init() {
            this.reShuffleMarkers();
            //
        },
        
        reShuffleMarkers() {
            this.markertypes.forEach(list => {
                list.markers = this[list.name + 'markers'];
                if (list.markers.length)
                    list.show = true;
                else
                    list.show = false;
            })
        },
    }
}
</script>

<style>
.prependMarkerLink {
    position: absolute;
    top: 0px;
    left: 2px;
}

.typeTitle {
    letter-spacing: .25ch;
}

.prependEdit {
    position: absolute;
    bottom: -2px;
    left: 2px;
}

.markerTitle {
    font-size: 13px;
    font-weight: 500;
}

.markerTitle {
    letter-spacing: .25ch;
}

.appendMarkerLink {
    position: absolute;
    top: 0px;
    right: 2px;
}

.appendMarker2Link {
    position: absolute;
    top: 0px;
    right: 52px;
}

</style>
