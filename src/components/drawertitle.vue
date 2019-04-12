<template>
    <div>
        <v-list class="pt-0 pb-0 mb-0" two-line dense subheader>
        <v-list-tile style="background-color: rgba(0,0,0,0.04);">
          <v-list-tile-content class="pt-1">
            <v-list-tile-title class="headline">{{activeBoard}}</v-list-tile-title>
            <v-list-tile-sub-title class="mt-2 body-2">{{activeBoardDesc}}</v-list-tile-sub-title>
          </v-list-tile-content>
        </v-list-tile>
      </v-list>
      <v-list
        class="pt-0 pb-0 mb-0"
        two-line
        subheader
        :style="getUserGrid()"
      >
        <v-list-tile
          id="leaders"
          :style="getUserStyle()"
          v-for="user in activeUsers"
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
      <v-divider></v-divider>
      <v-list dense class="py-0 mt-0">
        <v-list-tile id="navbtns" color="grey" style="padding: 0px 0px 0px 0px;">
          <div id="navbtn">
            <v-btn @click="gotoActiveHome()" block flat ripple>
              <v-icon color="grey" class="pb-1 px-1">home</v-icon>
              <span class="grey--text px-2">Home</span>
            </v-btn>
            <v-btn @click="gotoActiveAMA()" block flat ripple>
              <v-icon color="grey" class="pb-1 px-1">comment</v-icon>
              <span class="grey--text px-2">AMA</span>
            </v-btn>
          </div>
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
                name: 'supernal',
                board: 'Terrenus',
                role: 'Board Leader',
                avatar: 'https://www.valucre.com/uploads/profile/photo-thumb-2.gif',
                link: 'https://www.valucre.com/profile/2-supernal/'
            },
            {
                name: 'desolate',
                board: 'Terrenus',
                role: 'Board Leader',
                avatar: 'https://www.valucre.com/uploads/monthly_2016_02/p4vzE5I_png1.thumb_146f68df7d4e920e25020fdf124f268f.ac66dd3bb24290da6172d340c3c33a6b',
                link: 'https://www.valucre.com/profile/10-desolate/'
            },
            {
                name: 'King',
                board: 'Genesaris',
                role: 'Board Leader',
                avatar: 'https://www.valucre.com/uploads/monthly_2019_03/drink.thumb.png.e8a0972472908f9f7473f5b2045c4dbe.png',
                link: 'https://www.valucre.com/profile/305-king/'
            },
            {
                name: 'Aleksei',
                board: 'Renovatio',
                role: 'Board Leader',
                avatar: 'https://www.valucre.com/uploads/profile/photo-thumb-224.gif',
                link: 'https://www.valucre.com/profile/224-aleksei/'
            },
            {
                name: 'Pasion Pasiva',
                board: 'Orisia',
                role: 'Leader',
                avatar: 'https://www.valucre.com/uploads/monthly_2018_05/1300835281_Picture14_zps6bcf126b(1).thumb.png.29843fe8f6fa5a3ea546ba6ed3856ec7.png',
                link: 'https://www.valucre.com/profile/1414-pasion-pasiva/'
            },
            {
                name: 'paradigm',
                board: 'Alterion',
                role: 'Leader',
                avatar: 'https://www.valucre.com/uploads/monthly_2019_02/E21C3C75-AF87-4DF6-BAEE-E8499921DF8A.thumb.jpeg.16381e92f810c4008da58f5275949c05.jpeg',
                link: 'https://www.valucre.com/profile/153-paradigm/'
            },
        ],
    }),
    computed: {
        app() {
            return this.$root.$children[0];
        },
        activeBoard() {
            if (this.app.isLoaded && this.app.mainmap.hasMap) {
                return this.app.mainmap.activeBoard;
            }
        },
        activeUsers() {
            return this.users.filter(user => {
                return user.board == this.activeBoard;
            })
        },
        activeBoardDesc() {
            if (this.app.isLoaded) {
                return this.app.findBoardByName(this.activeBoard).desc;
            }
        },
        activeAMA() {
            if (this.app.isLoaded)
                return this.app.findBoardByName(this.activeBoard).ama;
        },
        activeHome() {
            if (this.app.isLoaded)
                return this.app.findBoardByName(this.activeBoard).landingPage;
        }
    },
    methods: {

        goToUserProfile(user) {
            window.open(user.link);
        },
        gotoActiveAMA() {
            window.open(this.activeAMA);
        },
        gotoActiveHome() {
            window.open(this.activeHome);
        },
        getUserStyle() {
            return `
                background-color: rgba(0,0,0,0.04);
                max-width: ${this.activeUsers.length > 1 ? 150 : 300 }px;  
            `
        },
        getUserGrid() {
            return `
                display: grid;
                grid-template-columns: ${this.activeUsers.length > 1 ? '1fr 1fr' : '1fr'};
            `
        }
    }
}
</script>

<style>

</style>
