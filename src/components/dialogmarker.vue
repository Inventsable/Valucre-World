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
                    <v-toolbar-title>{{getTitle}}</v-toolbar-title>
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
                            <v-flex xs12 sm6 md3>
                                <v-text-field 
                                    v-model="link"
                                    prepend-inner-icon="link"
                                    label="Link"
                                    hide-details
                                ></v-text-field>
                            </v-flex>
                            <div v-if="$vuetify.breakpoint.name == 'sm'" style="width:100%; height: 4px; margin-top: 4px;"></div>
                            <div v-if="$vuetify.breakpoint.name == 'xs'" style="width:100%; height: 4px; margin-top: 4px;"></div>
                            <v-flex sm2 xs4 md1 class="py-1">
                                <v-select
                                    :items="['City', 'Town', 'Land']"
                                    label="Type"
                                    value="City"
                                    hide-details
                                    @input="setType"
                                ></v-select>
                            </v-flex>
                            <v-flex xs8 sm10 md4>
                                <v-text-field 
                                    v-model="image"
                                    prepend-inner-icon="image"
                                    label="Image"
                                    hide-details
                                ></v-text-field>
                            </v-flex>
                            

                            <v-flex xs12 class="pt-4">
                                <v-textarea 
                                    v-model="desc"
                                    auto-grow
                                    box
                                    label="Description"
                                    hide-details
                                    rows="3"
                                    ></v-textarea>
                            </v-flex>
                            <!-- <v-flex xs12 sm4 md4 v-for="cat in categories" :key="cat">
                                <v-item-group multiple>
                                    <v-subheader>{{cat}}</v-subheader>
                                    <v-item
                                    v-for="n in prefixes"
                                    :key="n"
                                    >
                                    <v-chip
                                        slot-scope="{ active, toggle }"
                                        :selected="active"
                                        @click="toggle"
                                    >
                                        {{ n }}
                                    </v-chip>
                                    </v-item>
                                </v-item-group>
                            </v-flex> -->
                            <div v-if="$vuetify.breakpoint.name == 'sm'" style="width:100%; height: 0px; margin-top: 2px;"></div>
                            <v-flex xs12 sm6>
                                <v-text-field 
                                    v-model="contactAvatar"
                                    prepend-inner-icon="account_circle"
                                    label="Contact Avatar"
                                    hide-details
                                ></v-text-field>
                            </v-flex>
                            <v-flex xs12 sm6>
                                <v-text-field 
                                    v-model="contactLink"
                                    prepend-inner-icon="alternate_email"
                                    label="Contact Link"
                                    hide-details
                                ></v-text-field>
                            </v-flex>
                            <v-flex xs12 sm3 md3>
                                <v-slider
                                    v-model="Fantasy"
                                    @change="changeFSlider"
                                    @input="updateFSlider"
                                    :style="getSliderStyle()"
                                    hide-details
                                    inverse-label
                                    label="Fantasy"
                                    :tick-labels="prefixes"
                                    :max="2"
                                    step="1"
                                    ticks="always"
                                    tick-size="2"
                                ></v-slider>
                            </v-flex>
                            <v-spacer></v-spacer>
                            <v-flex xs12 sm3 offset-sm1 md3 offset-md1>
                                <v-slider
                                    v-model="Magic"
                                    @change="changeMSlider"
                                    @input="updateMSlider"
                                    :style="getSliderStyle()"
                                    hide-details
                                    inverse-label
                                    label="Magic"
                                    :tick-labels="prefixes"
                                    :max="2"
                                    step="1"
                                    ticks="always"
                                    tick-size="2"
                                ></v-slider>
                            </v-flex>
                            <v-spacer></v-spacer>
                            <v-flex xs12 sm3 offset-sm1 md3 offset-md1>
                                <v-slider
                                    v-model="Tech"
                                    @change="changeTSlider"
                                    @input="updateTSlider"
                                    :style="getSliderStyle()"
                                    hide-details
                                    inverse-label
                                    label="Tech"
                                    :tick-labels="prefixes"
                                    :max="2"
                                    step="1"
                                    ticks="always"
                                    tick-size="2"
                                ></v-slider>
                            </v-flex>
                        </v-layout>

                    </v-container>
                    <div id="preview" class="pa-4">
                        <markerpreview :marker="marker" />
                    </div>
                </v-card-text>
                <v-card-actions class="pr-4">
                    <v-container>
                        <v-divider class="pb-3"></v-divider>
                        <v-layout row wrap>

                            <v-btn :class="(/xs/.test(this.$vuetify.breakpoint.name) ? '' : 'ml-3')" v-if="isEditing" color="error" flat @click="deleteMarker()">Delete</v-btn>
                            <!-- <v-divider v-if="isEditing" vertical></v-divider> -->
                            <v-spacer></v-spacer>
                            <!-- <v-divider vertical></v-divider> -->
                            <!-- <v-btn icon color="blue darken-1" flat @click="close()">
                                <v-icon>clear</v-icon>
                            </v-btn> -->
                            <!-- <v-divider vertical></v-divider> -->
                            <v-btn color="blue darken-1" flat @click="submit()">Save</v-btn>
                        </v-layout>
                    </v-container>
                </v-card-actions>
            </v-card>
            </v-dialog>
        </v-layout>
    <!-- </div> -->
</template>

<script>
import db from '@/firebase/init'
import markerpreview from './dialogMarkerPreview.vue'

export default {
    name: 'dialogmarker',
    components: {
        markerpreview,
    },
    data: () => ({
        Fantasy: 1,
        Magic: 1,
        Tech: 1,
        // ticksLabels: [
        //   'Figs',
        //   'Lemon',
        //   'Pear',
        //   'Apple'
        // ],
        prefixes: ['Low', 'Mid', 'High'],
        categories: ['Fantasy', 'Magic', 'Tech'],
        dialog: false,
        hover: false,
        active: false,
        isEditing: false,
        activeBoard: null,
        lat: null,
        lng: null,
        board: null,
        title: 'New Marker',
        type: 'City',
        dbref: '',
        link: '',
        image: '',
        desc: 'Description goes here',
        fmt: [1,1,1],
        contactAvatar: '',
        contactLink: '',
        tags: [],
        alerts: [],
        // tooltip: null,
        defaultItem: {
            title: 'New Marker',
            type: 'City',
            lat: -40,
            lng: -50,
            link: '',
            image: '',
            desc: 'Description goes here',
            fmt: [1,1,1],
            tags: [],
            alerts: [],
            // tooltip: 'Tooltip here',
        },
    }),
    mounted() {
        // console.log('Hello');
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
        marker() {
            // if (this.isEditing) {
            //     return this.app.editingMarker;
            // } else {
                return this.defaultMarker;
            // }
        },
        defaultMarker() {
            return {
                lat: this.lat,
                lng: this.lng,
                board: this.board,
                title: this.title,
                dbref: this.dbref,
                image: this.image,
                type: this.type,
                link: this.link,
                desc: this.desc,
                hover: false,
                active: false,
                fmt: [this.Fantasy, this.Magic, this.Tech],
                alerts: this.alerts,
                tags: this.tags,
                contactAvatar: this.contactAvatar,
                contactLink: this.contactLink,
            }
        },
        getTitle() {
            if (!this.isEditing)
                return `New ${this.activeBoard} Marker`;
            else
                return `Editing ${this.marker.title}`;
        },
        drawer() {
            if (this.app.isLoaded) {
                return this.app.$refs.drawer.state;
            }
        }
    },
    methods: {
        inheritMarker(marker) {
            console.log('Assigning marker contents to edit')
            this.activeBoard = marker.activeBoard;
            this.lat = marker.lat;
            this.Fantasy = marker.fmt[0];
            this.Magic = marker.fmt[1];
            this.Tech = marker.fmt[2];
            this.lng = marker.lng;
            this.dbref = marker.dbref;
            this.image = marker.image;
            this.isEditing = marker.isEditing;
            this.board = marker.board;
            this.title = marker.title;
            this.type = marker.type;
            this.alerts = marker.alerts;
            this.tags = marker.tags;
            this.link = marker.link;
            this.fmt = marker.fmt;
            this.desc = marker.desc;
            this.contactAvatar = marker.contactAvatar;
            this.contactLink = marker.contactLink;
            this.isEditing = true;
        },
        initializeAsEditMarker(name) {
            console.log(`Should edit ${name}`);
            
        },
        deleteMarker() {
            const self = this;
            let confirm = window.confirm(`Are you sure you want to delete ${this.title}?`)
            if (confirm) {
                db.collection(this.board).doc(this.dbref).delete()
                .then(() => {
                    // console.log()
                    self.app.notificationText = `${this.title} was removed.`;
                    self.app.notificationIcon = 'info';
                    self.app.notificationColor = 'primary';
                    self.app.hasNotification = true;
                    self.close();
                })
            } else {
                // return;
            }
        },
        getSliderStyle() {
            if (/sm/.test(this.$vuetify.breakpoint.name)) {
                return `
                    font-size: 12px;
                `
            } else {
                return `
                    font-size: 16px;
                `
            }
        },
        changeFSlider(val) {
            console.log(`Change f from ${this.fmt[0]} to ${+val + 1}`)
            this.fmt[0] = +val + 1;
            this.desc += ' '
            this.desc = this.desc.trim();
        },
        updateFSlider(val) {
            this.fmt[0] = +val + 1;
            this.desc += ' '
            this.desc = this.desc.trim();
        },
        changeMSlider(val) {
            this.fmt[1] = +val + 1;
            this.desc += ' '
            this.desc = this.desc.trim();
        },
        updateMSlider(val) {
            this.fmt[1] = +val + 1;
            this.desc += ' '
            this.desc = this.desc.trim();
        },
        changeTSlider(val) {
            this.fmt[2] = +val + 1;
            this.desc += ' '
            this.desc = this.desc.trim();
        },
        updateTSlider(val) {
            this.fmt[2] = +val + 1;
            this.desc += ' '
            this.desc = this.desc.trim();
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
            this.clear();
            this.lat = this.map.simplePos.lat;
            this.lng = this.map.simplePos.lng;
            this.activeBoard = this.map.activeBoard;
            this.board = this.map.activeBoard;
            this.type = 'City';
            console.log('Was auto init');
        },
        close() {
            this.clear()
            this.dialog = false;
        },
        boardLeader() {
            if (this.app.isLoaded) {
                let result = this.app.$refs.drawer.$refs.infoTitle.activeUsers[0];
                // console.log(`Result is:`)
                // console.log(result);
                return result;
            } else {
                return {
                    avatar: this.app.defaultAvatar
                }
            }
        },
        submit() {
            const self = this;
            console.log('Submitting marker');

            let marker = {
                lat: this.lat,
                lng: this.lng,
                board: this.board,
                title: this.title,
                fmt: [this.Fantasy, this.Magic, this.Tech],
                dbref: this.title.split(' ').join('_').split('\-').join('\#').split('\'').join('-'),
                type: this.type,
                link: this.link,
                image: this.image,
                desc: this.desc,
                alerts: this.alerts,
                tags: this.tags,
                contactAvatar: this.contactAvatar,
                contactLink: this.contactLink,
            }

            if (this.isEditing) {
                marker.dbref = this.dbref;
            }

            if (!marker.alerts) {
                marker.alerts = [];
            }
            if (!marker.contactAvatar) {
                // marker.contactAvatar = this.boardLeader().avatar;
                marker.contactAvatar = '';
            }
            if (!marker.contactLink) {
                // marker.contactLink = this.boardLeader().link;
                marker.contactLink = '';
            }
            console.log(marker);
            db.collection(this.board).doc(marker.dbref).set({
                lat: marker.lat,
                lng: marker.lng,
                board: marker.board,
                title: marker.title,
                fmt: marker.fmt,
                dbref: marker.title.split(' ').join('_').split('\-').join('\#').split('\'').join('-'),
                type: marker.type,
                link: marker.link,
                image: marker.image,
                desc: marker.desc,
                alerts: marker.alerts,
                tags: marker.tags,
                contactAvatar: marker.contactAvatar,
                contactLink: marker.contactLink,
            })
            .then(() => {
                if (self.isEditing)
                    self.app.notificationText = `${this.title} was modified successfully`;
                else
                    self.app.notificationText = `${this.title} was added successfully`;
                self.app.notificationIcon = 'location_on';
                self.app.notificationColor = 'primary';
                self.app.hasNotification = true;
                setTimeout(() => {
                    console.log('Toggling to world')
                    self.app.mainmap.activeBoard = 'World';
                    setTimeout(() => {
                        self.app.mainmap.findBoardInCurrentView();
                        console.log(`Re-reading ${self.app.mainmap.activeBoard}`);
                    }, 100);
                }, 50);
                self.close();
            }).catch(err => {
                self.app.notificationText = `Error ${this.isEditing ? 'editing' : 'adding'} ${this.title}`;
                self.app.notificationIcon = 'warning';
                self.app.notificationColor = 'error';
                self.app.hasNotification = true;
                self.close();
            })
            // this.addNewMarker();
            
        },
        clear() {
            this.activeBoard = null;
            this.lat = null;
            this.lng = null;
            this.isEditing = false;
            this.board = null;
            this.title = 'New Marker';
            this.type = 'City';
            this.image = null;
            this.link = null;
            this.fmt = [1,1,1];
            this.desc = 'Description goes here';
            this.contactAvatar = '';
            this.contactLink = '';
            this.Fantasy = 1;
            this.Magic = 1;
            this.Tech = 1;
        },
        setType(data) {
            this.type = data;
        },
    },
}
</script>

<style scoped>
#preview {
    display: flex;
    justify-content: center;
    width: 100%;
    /* height: 100px; */
}

.isolated {
    /* border: 2px solid red; */
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
