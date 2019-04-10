<template>
    <v-toolbar dense class="" style="z-index:400;"
        id="toolbar">
        <v-divider style="" vertical></v-divider>
        <v-overflow-btn
            v-if="false"
            id="selector"
            :items="dropdown_edit"
            editable
            label="Search"
            hide-details
            overflow
        ></v-overflow-btn>
        <v-spacer class=""></v-spacer>
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
            <v-btn icon @click="toggleLock" flat>
                <v-icon>{{(toggle_exclusive) ? 'lock' : 'lock_open'}}</v-icon>
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
        showmarkers: [1, 2, 3]
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
                    return 'more_vert';
                }
            } else {
                return 'more_vert';
            }
        }
    },
    methods: {
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
            // this.$parent.$refs.drawer.state = !this.$parent.$refs.drawer.state;
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
