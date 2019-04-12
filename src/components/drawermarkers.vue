<template>
    <div>
        <v-list-group v-for="type in markertypes" 
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
                markers: [],
                foci: [true],
            },
            {
                name: 'town',
                icon: 'store',
                title: 'Towns',
                value: false,
                markers: [],
                foci: [false],
            },
            {
                name: 'land',
                icon: 'terrain',
                title: 'Landmarks',
                value: false,
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
                return this.app.mainmap.citymarkers;
            }
        },
        townmarkers() {
            if (this.app.isLoaded && this.app.mainmap.hasMap) {
                return this.app.mainmap.townmarkers;
            }
        },
        landmarkers() {
            if (this.app.isLoaded && this.app.mainmap.hasMap) {
                return this.app.mainmap.landmarkers;
            }
        },
    },
    methods: {
        init() {
            this.markertypes[0].markers = this.citymarkers;
            this.markertypes[1].markers = this.townmarkers;
            this.markertypes[2].markers = this.landmarkers;
        }
    }
}
</script>

