<template>
    <!-- <div class="isolated"> -->
        <v-layout row justify-center>
            <v-dialog 
                lazy
                v-model="dialog" 
                max-width="600px"
                style="z-index:500;">
            <v-card>
                <v-card-title class="pb-0">
                <span class="headline">{{`New ${this.activeBoard} Marker`}}</span>
                </v-card-title>
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
                    <small>*indicates required field</small>
                </v-card-text>
                <v-card-actions>
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
        init() {
            this.lat = this.map.simplePos.lat;
            this.lng = this.map.simplePos.lng;
            this.activeBoard = this.map.activeBoard;
        },
        close() {
            this.clear()
            this.dialog = false;
        },
        submit() {
            console.log('Submitting marker');
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
.v-dialog__content {
    top: 48px;
    height: calc(100% - 48px);
}

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
