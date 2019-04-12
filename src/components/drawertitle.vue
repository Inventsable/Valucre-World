<template>
    <div>
        <v-list class="pt-0 pb-0 mb-0" two-line dense subheader>
        <v-list-tile style="background-color: rgba(0,0,0,0.04);">
          <v-list-tile-content class="pt-1">
            <v-list-tile-title class="headline">Terrenus</v-list-tile-title>
            <v-list-tile-sub-title class="mt-2 body-2">Where magic and technology meet</v-list-tile-sub-title>
          </v-list-tile-content>
        </v-list-tile>
      </v-list>
      <v-list
        class="pt-0 pb-0 mb-0"
        two-line
        subheader
        style="display: grid;grid-template-columns:1fr 1fr;"
      >
        <v-list-tile
          id="leaders"
          style="background-color: rgba(0,0,0,0.04);max-width:150px;"
          v-for="user in users"
          :key="user.name"
          avatar
          @click="goToUserProfile(user)"
        >
          <v-list-tile-avatar class="pl-2 ml-2" color="white">
            <img :src="user.avatar">
          </v-list-tile-avatar>
          <v-list-tile-content>
            <v-list-tile-title>{{ user.name }}</v-list-tile-title>
            <v-list-tile-sub-title>{{ user.role }}</v-list-tile-sub-title>
          </v-list-tile-content>
        </v-list-tile>
      </v-list>
    </div>
</template>

<script>
export default {
    name: 'drawertitle',

    data: () => ({
        users: [
            {
                name: "supernal",
                role: "Board Leader",
                avatar: "https://www.valucre.com/uploads/profile/photo-thumb-2.gif",
                link: "https://www.valucre.com/profile/2-supernal/"
            },
            {
                name: "desolate",
                role: "Board Leader",
                avatar:
                "https://www.valucre.com/uploads/monthly_2016_02/p4vzE5I_png1.thumb_146f68df7d4e920e25020fdf124f268f.ac66dd3bb24290da6172d340c3c33a6b",
                link: "https://www.valucre.com/profile/10-desolate/"
            }
        ],
    }),
    computed: {
        app() {
            return this.$root.$children[0];
        }
    },
    methods: {
        goToUserProfile(user) {
            window.open(user.link);
        }
    }
}
</script>

<style>

</style>
