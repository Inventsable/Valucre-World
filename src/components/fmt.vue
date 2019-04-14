<template>
    <div class="enumWrapper">
        <div v-for="item in enums" :key="item.name" 
            class="fmtBlock"
            :style="getBlockStyle(item)">
                <span class="text-uppercase subheading">{{item.name}}</span>
                <span style="padding-left: 2px;">{{item.value}}</span>
        </div>
    </div>
</template>

<script>
export default {
    name: 'fmt',
    props: {
        marker: Object,
    },
    data: () => ({
        enum: ['f', 'm', 't'],
    }),
    computed: {
        app() {
            return this.$root.$children[0];
        },
        enums() {
            let mirror = [];
            this.enum.forEach((item, i) => {
                console.log(i)
                mirror.push({
                    name: item,
                    value: +this.marker.fmt[i],
                })
            })
            return mirror;
        }
    },
    methods: {
        getBlockStyle(item) {
            let colors = ['#757575', '#FB8C00', '#3949AB']
            return `
                background-color: ${colors[item.value - 1]}

            `
        }
    }
}
</script>

<style>
.enumWrapper {
    /* border: 2px solid red; */
    display: flex;
    /* flex-direction: column; */
    justify-content: flex-end;
    align-items: center;
}

.fmtBlock {
    /* font-size: 16px; */
    padding: 0px 4px;
    margin: 4px;
    color: white;
    border-radius: 2px;
    /* border: 2px solid green; */
    
}
</style>
