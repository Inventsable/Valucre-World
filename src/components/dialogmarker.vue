<template>
    <!-- <div class="isolated"> -->
        <v-layout row justify-center>
            <v-dialog 
                persistent
                v-model="dialog" 
                fullscreen hide-overlay transition="dialog-bottom-transition"
                :style="checkFullScreenStyle()"
                >
                <!-- max-width="600px"
                :style="checkFullScreenStyle()" -->
            <v-card>
                <v-toolbar dark color="primary" class="px-4">
                    <v-btn icon dark @click="dialog = false">
                        <v-icon>close</v-icon>
                    </v-btn>
                    <v-toolbar-title>{{`New ${this.activeBoard} Marker`}}</v-toolbar-title>
                    <v-spacer></v-spacer>
                </v-toolbar>
                <!-- <v-card-title class="pb-0">
                    <span class="headline">{{`New ${this.activeBoard} Marker`}}</span>
                </v-card-title> -->
                <v-card-text>
                    <v-container grid-list-md class="pt-0">
                        <v-layout wrap>
                            <v-flex xs12 sm6 md4>
                                <v-text-field
                                    label="Title"
                                    v-model="title"
                                    hide-details
                                    ></v-text-field>
                            </v-flex>
                            <v-flex xs12 sm6 md4>
                                <v-text-field 
                                    v-model="link"
                                    prepend-inner-icon="link"
                                    label="Link"
                                    hide-details
                                ></v-text-field>
                            </v-flex>
                            <v-flex sm2 xs4 class="py-2">
                                <v-select
                                    :items="['City', 'Town', 'Land', 'Region', 'Waters']"
                                    label="Type"
                                    value="City"
                                    hide-details
                                    @input="setType"
                                ></v-select>
                            </v-flex>
                            <v-flex xs12>
                                <v-textarea 
                                    v-model="desc"
                                    auto-grow
                                    box
                                    label="Description"
                                    hide-details
                                    rows="3"
                                    ></v-textarea>
                            </v-flex>
                        </v-layout>
                    </v-container>
                </v-card-text>
                <v-card-actions class="pr-4">
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" flat @click="close()">Close</v-btn>
                    <v-btn color="blue darken-1" flat @click="submit()">Save</v-btn>
                </v-card-actions>
            </v-card>
            </v-dialog>
        </v-layout>
    <!-- </div> -->
</template>

<script>
import db from '@/firebase/init'

export default {
    name: 'dialogmarker',
    data: () => ({
        dialog: false,
        activeBoard: null,
        lat: null,
        lng: null,
        isEditing: false,
        board: null,
        title: null,
        type: null,
        link: null,
        desc: null,
        tooltip: null,
        defaultItem: {
            board: 'Terrenus',
            title: 'Greenwitch',
            link: null,
            desc: 'A world of wonder',
            lat: -40,
            lng: -50,
            tags: [],
            tooltip: 'Tooltip here',
            type: 'City',
        }
    }),
    mounted() {
        console.log('Hello');
        window.addEventListener('resize', this.checkBreakpoint);
        this.checkBreakpoint();
    },
    computed: {
        app() {
            return this.$root.$children[0];
        },
        map() {
            if (this.app.isLoaded) {
                return this.app.mainmap;
            }
        },
        drawer() {
            if (this.app.isLoaded) {
                return this.app.$refs.drawer.state;
            }
        }
    },
    methods: {
        addNewMarker(marker) {
            console.log(marker)
            // db.collection(marker.board).doc(marker.dbref).set({
            //     board: marker.board,
            //     dbref: marker.dbref,
            //     desc: marker.desc,
            //     lat: marker.lat,
            //     lng: marker.lng,
            //     link: marker.link,
            //     tags: marker.tags,
            //     tooltip: marker.tooltip,
            //     title: marker.title,
            //     type: marker.type,
            // });
        },
        getDBRef(marker) {
            marker['dbref'] = marker.title.split(' ').join('_').split('\'').join('-');
            // 
            return marker;
        },
        checkFullScreenStyle() {
            return `
                z-index:500;
                height: calc(100% - 48px);
                margin: 0px 0px 0px 0px;
            `
                // max-height: ${(/xs/.test(this.$vuetify.breakpoint.name)) ? (/sm/.test(this.$vuetify.breakpoint.name)) ? '100%' : '90%' : '90%'};            
        },
        checkBreakpoint() {
            let cards = document.querySelector('.v-dialog__content');
            // console.log(cards);
            let dialogs = document.querySelector('.v-dialog');
            let fullscreen = document.querySelector('.v-dialog:not(.v-dialog--fullscreen)')
            cards.style.top = (/sm|xs/.test(this.$vuetify.breakpoint.name)) ? 0 : 48 + 'px';
            dialogs.style.margin = (/sm|xs/.test(this.$vuetify.breakpoint.name)) ? '0px 0px 0px 0px' : '24px 24px 24px 24px';
            let altscreen = document.querySelector('.v-dialog--fullscreen')
            altscreen.style.margin = '0px 0px 0px 0px';
            // fullscreen.style.maxHeight = (/xs/.test(this.$vuetify.breakpoint.name)) ? (/sm/.test(this.$vuetify.breakpoint.name)) ? '100%' : '90%' : '90%';
        },
        init() {
            this.lat = this.map.simplePos.lat;
            this.lng = this.map.simplePos.lng;
            this.activeBoard = this.map.activeBoard;
            this.board = this.map.activeBoard;
            this.type = 'City';
        },
        close() {
            this.clear()
            this.dialog = false;
        },
        submit() {
            console.log('Submitting marker');
            let marker = {
                lat: this.lat,
                lng: this.lng,
                board: this.board,
                title: this.title,
                dbref: this.title.split(' ').join('_').split('\'').join('-'),
                type: this.type,
                link: this.link,
                desc: this.desc,
                tooltip: this.tooltip,
            }
            console.log(marker);
            db.collection(this.board).doc(marker.dbref).set(marker)
            .then(() => {
                console.log('Done');
            })
            // this.addNewMarker();
            this.close();
        },
        clear() {
            this.activeBoard = null;
            this.lat = null;
            this.lng = null;
            this.isEditing = false;
            this.board = null;
            this.title = null;
            this.type = null;
            this.link = null;
            this.desc = null;
            this.tooltip = null;
        },
        setType(data) {
            this.type = data;
        },
    },
}
</script>

<style scoped>


.isolated {
    border: 2px solid red;
    width: 100%;
    height: calc(100% - 48px);
    position: absolute;
    top: 48px;
    left: 0px;
    display: flex;
    justify-content: center;
    align-items: center;
}
</style>
