<template>
    <v-card min-width="300" v-if="checkingDistance">
            <v-alert
                dense
                :value="hasAlert(marker)"
                color="info"
                icon="new_releases"
                >
                {{marker.alert}}
                </v-alert>
            <v-img
                class="hidden-sm-and-down"
                :src="(marker.image) ? marker.image : defImg"
                height="100px"
                ></v-img>
            <v-img
                :src="(marker.image) ? marker.image : defImg"
                height="50px"
                class="hidden-md-and-up"
                >
                    
                    <div :style="getTitleStyle()">
                        <div class="headline">{{`${marker.title}`}}</div>
                        <v-icon color="white" class="pl-3">link</v-icon>
                    </div>
            </v-img>
            <v-card-title primary-title class="pt-1 ma-0 pb-2">
                <div class="hidden-md-and-up">
                    <div class="mt-2">
                        <span class="grey--text">{{marker.desc}}</span>
                    </div>
                </div>
                <div class="hidden-sm-and-down">
                    <div class="headline">{{`${marker.title}, ${marker.board}`}}</div>
                    <div class="mt-2">
                        <span class="grey--text">{{marker.desc}}</span>
                    </div>
                </div>

            </v-card-title>


        </v-card>
</template>

<script>
export default {
    name: 'markerannoactive',
    props: {
        marker: Object,
        hover: Boolean,
        active: Boolean,
    },
    computed: {
        defImg() {
            return this.$parent.defImg;
        }
    },
    data: () => ({

    }),
    methods: {
        hasAlert() {
            // console.log(this.marker.alert);
            if (this.marker.alert.length) {
                console.log(`${this.marker.alert} has length`)
                return true;
            } else {
                return false;
            }
        },
        getTitleStyle() {
            return `
                width: 100%;
                height: 100%;
                display: flex;
                justify-content: flex-end;
                align-items: center;
                color: white;
                padding-right: 2rem;
                text-shadow: 2px 2px 2px black;
            `
        },
    }

}
</script>

<style>


</style>
