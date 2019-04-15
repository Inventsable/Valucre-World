<template>
    <div v-if="hasPermissions()">
        <v-list-group
            prepend-icon="add_location" value="false">
            <template v-slot:activator>
                <v-list-tile>
                    <v-list-tile-title>New</v-list-tile-title>
                </v-list-tile>
            </template>
            <v-list-tile @click="openMarkerDialog"
                @mouseenter="addingMarker()"
                @mouseleave="notAddingMarker()"
                >
                <v-list-tile-action></v-list-tile-action>
                <v-list-tile-title>
                    <span>Marker</span>
                </v-list-tile-title>
                <v-list-tile-action style="display:flex; justify-content:flex-end;">
                    <v-btn icon ripple>
                        <v-icon color="grey">add</v-icon>
                    </v-btn>
                </v-list-tile-action>
            </v-list-tile>
            <!-- <v-list-tile @click=""
                @mouseenter="addingMarker()"
                @mouseleave="notAddingMarker()">
                <v-list-tile-action></v-list-tile-action>
                <v-list-tile-title>
                    <span>Marker</span>
                </v-list-tile-title>
                <v-list-tile-action style="display:flex; justify-content:flex-end;">
                    <v-btn icon ripple>
                        <v-icon color="grey">add</v-icon>
                    </v-btn>
                </v-list-tile-action>
            </v-list-tile> -->
        </v-list-group>
        
    </div>
</template>

<script>
export default {
    name: 'draweractions',
    data: () => ({
        actions: [
            {
                title: 'Marker',
                icon: 'alert',
                crosshair: 'marker',
            },
            {
                title: 'Alert',
                icon: 'alert',
                crosshair: 'marker',
            }
            // ["Marker", "alert"],
            // ["Alert", "Alert"],
            // ["Land", "terrain"],
            // ["Alert", "notifications"]
        ]
    }),
    computed: {
        app() {
            return this.$root.$children[0];
        },
    },
    methods: {
        
        hasPermissions() {
            if (this.app.isLoaded) {
                // console.log('Checking permissions');
                if (this.app.permissions.includes('World')) {
                    return true;
                } else if (this.app.permissions.includes(this.app.mainmap.activeBoard)) {
                    return true;
                } else {
                    return false;
                }
            }
        },
        openMarkerDialog() {
            if (this.app.isLoaded) {
                this.app.$refs.dialogmarker.dialog = true;
                this.app.$refs.dialogmarker.isEditing = false;
                this.app.$refs.dialogmarker.init();
            }
        },
        addingMarker() {
            // console.log('Should be adding');
            this.app.mainmap.isAdding = true;
            this.app.mainmap.crosshairType = 'marker';
            // console.log(this.app.mainmap.isAdding)
        },
        notAddingMarker() {
            // console.log('Is not adding')
            this.app.mainmap.crosshairType = 'default';
            this.app.mainmap.isAdding = false;
        },
    }
}
</script>
