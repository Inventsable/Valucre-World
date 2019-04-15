<template>
    <v-card min-width="300" v-if="!active && !calculatingDistance && hover">
            <v-alert
                dense
                :value="hasAlert(marker)"
                color="info"
                
                icon="local_library"
                >
                {{getAlert(marker)}}
                </v-alert>
            <v-img
                class="hidden-sm-and-down"
                :src="(marker.image) ? marker.image : defImg"
                height="100px"
                style="background-color: rgba(0,0,0,0.625); transition: background-color 200ms var(--quart)"
                ></v-img>
            <v-img
                :src="(marker.image) ? marker.image : defImg"
                height="50px"
                class="hidden-md-and-up"
                style="background-color: rgba(0,0,0,0.625); transition: background-color 200ms var(--quart)"
                >
                    
                    <div :style="getTitleStyle()">
                        <div class="headline pr-2">{{`${marker.title}`}}</div>
                        <contactlist :marker="marker" />
                        <!-- <v-icon color="white" class="pl-3">link</v-icon> -->
                    </div>
            </v-img>
            <v-card-title primary-title class="pt-1 ma-0 pb-2">
                <div class="hidden-md-and-up">
                    <div class="mt-2 ">
                        <span class="grey--text">{{marker.desc}}</span>
                        
                    </div>
                </div>
                <div class="hidden-sm-and-down">
                    <div class="desktopTitle">
                        <contactlist :marker="marker" />
                        <div class="headline">{{`${marker.title}`}}</div>
                        <!-- <fmt :marker="marker" /> -->
                        
                    </div>
                    <div class="mt-2">
                        <span class="grey--text">{{marker.desc}}</span>
                    </div>
                </div>
                <!-- <div class="hidden-sm-only"> -->
                    <div class="pt-2" style="width: 100%;">
                        <v-chip small v-for="tag in convertedTags" :key="tag.name">
                            <!-- <v-icon small left>label</v-icon> -->
                            <span class="tagItem">
                                {{tag.label}}
                            </span>
                        </v-chip>
                    </div>
                <!-- </div> -->
            </v-card-title>


        </v-card>
</template>

<script>
import fmt from './fmt.vue';
import contactlist from './contactlist.vue';

export default {
    name: 'markerannohover',
    props: {
        marker: Object,
        hover: Boolean,
        active: Boolean,
    },
    components: {
        fmt,
        contactlist,
    },
    data: () => ({
        typeList: ['f', 'm', 't'],
    }),
    computed: {
        app() {
            return this.$root.$children[0];
        },
        defImg() {
            return this.$parent.defImg;
        },
        calculatingDistance() {
            return this.$parent.calculatingDistance;
        },
        convertedTags() {
            try {
                let prefixes = ['Low', 'Mid', 'High'];
                let genres = ['Fantasy', 'Magic', 'Tech'];
                let mirror = [];
                let fmtList = ['f', 'm', 't']
                // console.log(this.typeList)
                // this.typeList.forEach(param => {
                for (let i = 0; i < fmtList.length; i++) {
                    const fmtType = fmtList[i];
                    let real = +this.marker.fmt[i];
                    mirror.push({
                        name: fmtType,
                        value: +this.marker.fmt[i],
                        index: i,
                        label: prefixes[real] + ' ' + genres[i],
                    })
    
                }
                    // mirror.push()
                // })
                return mirror;
            } catch(err) {
                return ['Error']
            }
        }
    },
    mounted() {
        // this.$el.addEventListener('click', this.checkIfMobile)
    },
    methods: {
        getAlert(marker) {
            if (marker.alerts.length) {
                console.log('This marker has an alert')
                return marker.alerts[0];
            } else {
                return '';
            }
        },
        hasAlert(marker) {
            // console.log(this.marker.alert);
            if (marker.alerts) {
                if (marker.alerts.length) {
                    // console.log(`${this.marker.alert} has length`)
                    return true;
                } else {
                    return false;
                }
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
.desktopTitle {
    display: flex;
    justify-content: flex-start;
    align-items: center;
}

.tagItem {
    font-size: 10px;
}

</style>
