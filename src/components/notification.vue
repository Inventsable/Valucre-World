<template>
    <v-snackbar style="z-index:9999" v-model="show" :timeout="3000" top :color="bg">
        <v-icon v-if="icon" class="white--text">{{icon}}</v-icon>
        <span style="margin-left:.5rem;">{{text}}</span>
        <v-spacer />
        <v-btn flat fab color="white" @click="show = !show">
            <v-icon>close</v-icon>
        </v-btn>
    </v-snackbar>
</template>

<script>
export default {
    name: 'notification',
    props: {
        // show: Boolean,
    },
    data: () => ({
        show: false,
        // icon: 'info',
    }),
    computed: {
        app() {
            return this.$root.$children[0]
        },
        hasNotification() {
            return this.app.hasNotification;
        },
        text() {
            return this.app.notificationText;
        },
        icon() {
            return this.app.notificationIcon;
        },
        bg() {
            return this.app.notificationColor;
        }
    },
  
    methods: {
        showNotification() {
            this.show = true;
        }
    },
    mounted() {
        // setTimeout(() => {
        //     this.$emit('done');
        // }, 1000)
        // console.log(this.index)
    }
    
}
</script>
