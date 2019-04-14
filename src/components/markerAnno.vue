<template>
    <div class="annoBox" :style="getAnnoBoxStyle()" v-show="active || hover">
        
        <markerannohover
            :marker="marker"
            :hover="hover"
            :active="active"
            />
        <markerannoactive
            :marker="marker"
            :hover="hover"
            :active="active"
            />
        <markerannodistance
            :marker="marker"
            :hover="hover"
            :active="active"
            />
    </div>
</template>

<script>
import markerannohover from './markerAnnoHover.vue'
import markerannoactive from './markerAnnoActive.vue'
import markerannodistance from './markerAnnoDistance.vue'

export default {
    name: 'markeranno',
    components: {
        markerannoactive,
        markerannohover,
        markerannodistance
    },
    data: () => ({
        defImg: 'https://cdn.vuetifyjs.com/images/cards/sunshine.jpg',
        calculatingDistance: false,
    }),
    mounted() {
        
    },
    props: {
        marker: Object,
        active: Boolean,
        hover: Boolean,
        length: 3,
        window: 0,
        
    },
    methods: {
        init() {
            this.marker.annoElt = this.$el;
            // console.log('Marker assigned anno elt');
            // console.log(this.marker.annoElt);
        },
        getAnnoBoxStyle() {
            return `
                z-index: 110;
                width: 300px;
            `
        },
        
    },
    computed: {
        app() {
            return this.$root.$children[0];
        },
        // calculatingDistance() {
        //     if (this.app.isLoaded) {
        //         let active = this.app.mainmap.markers.find(marker => {
        //             return marker.active;
        //         })
        //         let hover = this.app.mainmap.markers.find(marker => {
        //             return marker.hover && marker.title !== active.title;
        //         })
        //         console.log(active);
        //         console.log(hover);
        //         if (active && hover) {

        //             if (active.title !== hover.title) {
    
        //                 return true;
        //             } else {
        //                 return false;
        //             }
        //         } else {
                    
        //             return false;
        //         }
        //     }
        // }
    },
    mounted() {

    },
}
</script>

<style scoped>
.annoBox {
    /* border: 2px solid blue; */
    position: absolute;
    bottom: 30px;
    z-index: 200;
}

.titleBar {
    display: flex;
    justify-content: space-between;
    padding-left: 2rem;
    padding-right: 2rem;
    align-items: center;
}

.tagsList {
    border: 2px solid red;
    display: flex;
    justify-content: flex-start;
    flex-wrap: wrap;
}


</style>
