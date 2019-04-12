<template>
    <div>
        <v-list-group 
            v-for="type in markertypes" 
            v-show="type.show"
            :key="type.name" 
            :prepend-icon="type.icon" 
            :value="type.value">
            <template v-slot:activator>
                <v-list-tile>
                    <v-list-tile-title>{{type.title}}</v-list-tile-title>
                </v-list-tile>
            </template>
            <v-expansion-panel focusable v-model="type.foci">
                <v-expansion-panel-content v-for="(item,i) in type.markers" :key="i">
                    <template v-slot:header>
                        
                        <div class="markerTitle">{{item.title}}</div>
                    </template>
                    <div class="prependMarkerLink">
                        <v-btn icon>
                            <v-icon>link</v-icon>
                        </v-btn>
                    </div>
                    <v-card>
                        <v-card-text
                            class="grey lighten-3"
                        >{{item.desc}}</v-card-text>
                    </v-card>
                </v-expansion-panel-content>
            </v-expansion-panel>
        </v-list-group>
    </div>
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
    },
    watch: {
        activeBoard(state) {
            this.reShuffleMarkers();
        }
    },
    methods: {
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

