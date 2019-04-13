<template>
    <v-card min-width="300" v-if="active">
        <v-img
            :src="(marker.image) ? marker.image : defImg"
            height="50px"
            >
                <div :style="getTitleStyle()">
                    <div class="headline">{{`${marker.title}`}}</div>
                    <v-icon color="white" class="pl-3">link</v-icon>
                </div>
        </v-img>
        <v-card-title primary-title class="pt-1 ma-0">
            <div>
                <span class="grey--text">{{marker.details}}</span>
            </div>
        </v-card-title>
        <v-card-text>

        </v-card-text>
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
.v-alert {
    margin: 0px 0px 0px 0px;
    padding: 4px 16px;
}
</style>
