<template>
    <v-card min-width="300" v-if="calculatingDistance">
            <v-card-title primary-title class="pt-1 ma-0 pb-2">
                    <div class="my-2">
                        <span class="grey--text">{{distanceAnno}}</span>
                    </div>
                    <v-layout style="width:100%;">
                        <v-flex xs12>
                            <span class="subheading">{{miles}}</span>
                        </v-flex>
                        <v-flex xs12>
                            <span class="subheading">{{kilometers}}</span>
                        </v-flex>
                    </v-layout>
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
        app() {
            return this.$root.$children[0];
        },
        calculatingDistance() {
            return this.$parent.calculatingDistance;
        },
        defImg() {
            return this.$parent.defImg;
        },
        distanceAnno() {
            if (this.app.isLoaded) {
                if (this.app.mainmap.selectedMarker) {
                    return `Distance from ${this.app.mainmap.selectedMarker.title} to ${this.marker.title} is: `
                }
            }
        },
        meters() {
            if (this.app.isLoaded && this.calculatingDistance) {
                if (this.app.mainmap.selectedMarker)
                    return this.getDistance(this.app.mainmap.selectedMarker.latlng, this.marker.latlng);
            }
        },
        kilometers() {
            return (this.meters/1000).toFixed(4) + 'km';
        },
        miles() {
            return (this.meters * 0.000621371192).toFixed(4) + 'mi';
        }
    },
    data: () => ({
        
    }),
    methods: {
        rad(x) {
            return x * Math.PI / 180;
        },
        getDistance(p1, p2) {
            var R = 6378137; // Earth’s mean radius in meter
            var dLat = this.rad(p2.lat() - p1.lat());
            var dLong = this.rad(p2.lng() - p1.lng());
            var a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
                Math.cos(this.rad(p1.lat())) * Math.cos(this.rad(p2.lat())) *
                Math.sin(dLong / 2) * Math.sin(dLong / 2);
            var c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
            var d = R * c;
            return d; // returns the distance in meter
        },
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
