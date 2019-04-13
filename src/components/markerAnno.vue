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
        
    </div>
</template>

<script>
import markerannohover from './markerAnnoHover.vue'
import markerannoactive from './markerAnnoActive.vue'

export default {
    name: 'markeranno',
    components: {
        markerannoactive,
        markerannohover,
    },
    data: () => ({
        defImg: 'https://cdn.vuetifyjs.com/images/cards/sunshine.jpg',
    }),
    props: {
        marker: Object,
        active: Boolean,
        hover: Boolean,
        length: 3,
        window: 0,
    },
    methods: {
        
        getAnnoBoxStyle() {
            return `
                width: 300px;
            `
        },
        getAnnoBoxLegacyStyle() {
            return `
                position: absolute;
                bottom: 26px;
                opacity: ${this.marker.active ? 1 : 0};
                transition: width 200ms var(--quart) 20ms;
                
                padding: ${this.marker.active ? '4px 10px' : '4px 10px'};
                width: ${this.marker.active ? 150 : 150}px;
                height: 50px;
                background-color: rgba(255,255,255,.95);
                z-index: 8;
            `
        },
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
