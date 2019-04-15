<template>
    <div class="contactWrapper pr-2">
        <!-- <div v-for="item in marker.contacts" :key="item.name" 
            class="fmtBlock"
            :style="getBlockStyle(item)">
                <span class="text-uppercase subheading">{{item.name}}</span>
                <span style="padding-left: 1.5px;">{{item.value}}</span>
        </div> -->
        <div style="border:1.5px solid rgba(0,0,0,0.12); border-radius: 30px;" v-if="hasContact">
            <v-avatar
                size="30px"
                color="grey"
                
              >
                <img
                    v-if="marker.contactAvatar"
                    :src="marker.contactAvatar"
                    alt="Avatar"
                    >
                <v-icon
                    v-else
                    color="grey"
                    v-text="user"
                ></v-icon>
            </v-avatar>
        </div>
        <div style="border:1.5px solid grey; border-radius: 30px;" v-else>
            <v-avatar
                size="30px"
                color="grey"
              >
                <img
                    v-if="boardLeader().avatar"
                    :src="boardLeader().avatar"
                    alt="Avatar"
                    >
                <v-icon
                    v-else
                    color="grey"
                    v-text="user"
                ></v-icon>
            </v-avatar>
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
        hasContact() {
            if (this.marker.contactAvatar) {
                if (this.marker.contactAvatar.length > 1) {
                    return true;
                } else {
                    return false;
                }
            } else {
                return false;
            }
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
.contactWrapper {
    /* border: 1.5px solid red; */
    display: flex;
    /* flex-direction: column; */
    justify-content: flex-end;
    align-items: center;
}

</style>
