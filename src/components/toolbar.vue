<template>
    <v-toolbar dense class="" style="z-index:400;"
        id="toolbar">
        <!-- <v-divider style="" vertical></v-divider>
        <v-overflow-btn
            v-if="false"
            id="selector"
            :items="dropdown_edit"
            editable
            label="Search"
            hide-details
            overflow
        ></v-overflow-btn>
        <v-spacer class=""></v-spacer> -->
        <v-toolbar-title class="pl-4">
            {{anno}}
        </v-toolbar-title>
        <v-spacer></v-spacer>
        <v-divider
            class="mr-2"
            vertical
        ></v-divider>

        <v-btn-toggle
            v-model="showmarkers"
            class="transparent"
            multiple
            >
            <v-btn :value="1" flat>
                <v-icon>location_city</v-icon>
            </v-btn>

            <v-btn :value="2" flat>
                <v-icon>store</v-icon>
            </v-btn>

            <v-btn :value="3" flat>
                <v-icon>terrain</v-icon>
            </v-btn>

        </v-btn-toggle>

        <v-divider
            class="mx-2"
            vertical
        ></v-divider>

        <div class="tail pr-2">
            <!-- <v-btn icon @click="toggleLock" flat>
                <v-icon>{{(toggle_exclusive) ? 'lock' : 'lock_open'}}</v-icon>
            </v-btn> -->
            <v-btn icon @click="testLink" flat>
                <v-icon>link</v-icon>
            </v-btn>
            <v-btn icon  @click="openDrawer" flat>
                <v-icon>{{drawericon}}</v-icon>
            </v-btn>
        </div>

        <!-- <v-btn-toggle
            v-model="toggle_exclusive"
            class="transparent"
            @change="changeStrict()"
            > -->
            
        <!-- </v-btn-toggle> -->

        <!-- <v-btn-toggle
            v-model="toggle_exclusive"
            class="transparent"
            @change="changeStrict()"
            >
            <v-btn :value="1" flat>
                <v-icon>{{(toggle_exclusive == 1) ? 'lock' : 'lock_open'}}</v-icon>
            </v-btn>
        </v-btn-toggle> -->
    </v-toolbar>
</template>


<script>
export default {
    name: 'toolbar',
    data: () => ({
        dropdown_font: [
            { text: 'Arial' },
            { text: 'Calibri' },
            { text: 'Courier' },
            { text: 'Verdana' }
        ],
        dropdown_edit: [
            { text: '100%' },
            { text: '75%' },
            { text: '50%' },
            { text: '25%' },
            { text: '0%' }
        ],
        toggle_exclusive: false,
        showmarkers: [1, 2, 3],
        link: {
            z: '',
            lat: '',
            lng: '',
            hover: '',
            select: '',
            board: '',
        },
        result: null,
    }),
    computed: {
        app() {
            return this.$root.$children[0];
        },
        drawericon() {
            if (this.app.isLoaded) {
                if (this.app.$refs.drawer.state) {
                    return 'clear';
                } else {
                    return 'menu';
                }
            } else {
                return 'menu';
            }
        },
        markers() {
            return this.app.mainmap.markers;
        },
        map() {
            return this.app.mainmap;
        },
        anno() {
            if (this.app.isLoaded) {
                if (this.app.mainmap.hasMap) {
                    let check = /world/i.test(this.app.mainmap.activeBoard)

                    if (this.$vuetify.breakpoint.name !== 'xs') {
                        if (this.app.$refs.drawer.state) {
                            return `z${this.zoom}`
                        } else {
                            return `z${this.zoom}, ${!check ? this.app.mainmap.activeBoard : 'World'}`
                        }
                    }
                    else
                        return `z${this.zoom}`;

                }
            } else {
                return '';
            }
        },
        zoom() {
            if (this.app.isLoaded) {
                if (this.app.mainmap.hasMap) {
                    return this.app.mainmap.zoom;
                // } else {
                //     return 'menu';
                }
            } else {
                return 0;
            }
        }
    },
    methods: {
        clearLink() {
            this.link.z = '';
            this.link.lat = '';
            this.link.lng = '';
            this.link.hover = '';
            this.link.active = '';
            this.link.board = '';
        },
        testLink() {
            // console.log(this.app.mainmap.simplePos);
            if (/xs|sm/.test(this.$vuetify.breakpoint.name)) {
                this.map.inMap = false;
            }
            this.clearLink();
            this.link.z = `z${this.map.zoom}`;
            let hovered = this.markers.find(marker => {
                return marker.hover;
            })
            let active = this.markers.find(marker => {
                return marker.active;
            })
            if ((active) || (hovered)) {
                console.log('A marker is active or hovered')
                if (active)
                    this.link.active = `select=${active.dbref}&`
                if (hovered)
                    this.link.hover = `hover=${hovered.dbref}&`
                this.link.z = (this.map.zoom > 2) ? 'z2to' + this.link.z : this.link.z;
            } else {
                console.log('Vanilla')
                this.link.lat = `lat${this.map.simplePos.lat}`;
                this.link.lng = `lng${this.map.simplePos.lng}`;
            }
            // this.link.lat = this
            this.result = this.link.z + this.link.lat + this.link.lng + this.link.active + this.link.hover + this.link.board;

            this.app.$refs.link.route = this.result;
            this.app.$refs.link.state = true;
        },
        getSearchStyle() {
            console.log(this.$vuetify.breakpoint.name);
            
            // console.log();
            
            return `
                margin-left: ${/sm|xs/.test(this.$vuetify.breakpoint.name) ? '0px' : '5rem'};
            `
        },
        toggleLock() {
            this.toggle_exclusive = !this.toggle_exclusive;
            this.changeStrict();
        },
        openDrawer() {
            if (/xs|sm/.test(this.$vuetify.breakpoint.name)) {
                this.map.inMap = false;
            }
            // this.$parent.$refs.drawer.state = !this.$parent.$refs.drawer.state;
            if (this.app.isLoaded)
                this.app.$refs.drawer.state = !this.app.$refs.drawer.state;
        },
        changeStrict() {
            // console.log(this.toggle_exclusive);
            const bounds = this.app.mainmap.limitBounds;
            let result = (this.toggle_exclusive) ? true : false;
            this.app.mainmap.map.setOptions({
                restriction: {
                    latLngBounds: bounds,
                    strictBounds: result,
                },
            })
            this.app.mainmap.hasRestriction = true;
            this.app.mainmap.isHarsh = result;
            // console.log(this.app.mainmap.map.options)
            console.log(result);
        }
    }
}
</script>

<style>
.v-toolbar__content {
    padding: 0px 0px 0px 0px;
}

div.v-menu__content.theme--light.menuable__content__active.v-autocomplete__content {
    max-width: 300px;

}
</style>
