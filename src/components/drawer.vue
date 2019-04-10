<template>
  <div>
    <v-navigation-drawer
      right
      :style="getDrawerStyle()"
      v-model="state"
      absolute
      clipped
      hide-overlay
    >
      <v-divider></v-divider>
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
      <v-divider></v-divider>
      <v-list dense class="pt-0 mt-0">
        <v-list-tile id="navbtns" color="grey" style="padding: 0px 0px 0px 0px;">
          <div id="navbtn">
            <v-btn block flat ripple>
              <v-icon color="grey" class="pb-1 px-1">home</v-icon>
              <span class="grey--text px-2">Home</span>
            </v-btn>
            <v-btn block flat ripple>
              <v-icon color="grey" class="pb-1 px-1">comment</v-icon>
              <span class="grey--text px-2">AMA</span>
            </v-btn>
          </div>
        </v-list-tile>
        <v-divider></v-divider>
        <v-list-group v-for="type in markertypes" :key="type.name" :prepend-icon="type.icon" :value="type.value">
          <template v-slot:activator>
            <v-list-tile>
              <v-list-tile-title>{{type.title}}</v-list-tile-title>
            </v-list-tile>
          </template>
          <v-expansion-panel focusable v-model="type.foci">
              <v-expansion-panel-content v-for="(item,i) in type.markers" :key="i">
                <template v-slot:header>
                    
                  <div class="markerTitle">{{item.title}}</div>
                </template>
                <div class="prependMarkerLink">
                    <v-btn icon>
                        <v-icon>link</v-icon>
                    </v-btn>
                </div>
                <v-card>
                  <v-card-text
                    class="grey lighten-3"
                  >{{item.desc}}</v-card-text>
                </v-card>
              </v-expansion-panel-content>
            </v-expansion-panel>
        </v-list-group>


        <!-- <v-divider></v-divider> -->
        <v-list-group prepend-icon="edit" value="false">
          <template v-slot:activator>
            <v-list-tile>
              <v-list-tile-title>New</v-list-tile-title>
            </v-list-tile>
          </template>
          <v-list-tile v-for="(crud, i) in cruds" :key="i" @click="">
            <v-list-tile-action></v-list-tile-action>
            <v-list-tile-title>
              <!-- <v-icon v-text="crud[1]"></v-icon> -->
              <!-- <v-icon>add</v-icon> -->
              <span>{{crud[0]}}</span>
            </v-list-tile-title>
            <v-list-tile-action style="display:flex; justify-content:flex-end;">
              <v-btn icon ripple>
                <v-icon color="grey">add</v-icon>
              </v-btn>
            </v-list-tile-action>
          </v-list-tile>
        </v-list-group>
      </v-list>
    </v-navigation-drawer>
  </div>
</template>

<script>
export default {
  name: "drawer",
  data: () => ({
    state: false,
    citypanel: [true],
    markertypes: [
        {
            name: 'city',
            icon: 'location_city',
            title: 'Cities',
            value: true,
            markers: [],
            foci: [true],
        },
        {
            name: 'town',
            icon: 'store',
            title: 'Towns',
            value: false,
            markers: [],
            foci: [false],
        },
        {
            name: 'land',
            icon: 'terrain',
            title: 'Landmarks',
            value: false,
            markers: [],
            foci: [false],
        },
    ],

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
    cruds: [
      ["City", "location_city"],
      ["Town", "store"],
      ["Land", "terrain"],
      ["Alert", "notifications"]
    ]
  }),
  computed: {
    app() {
      return this.$root.$children[0];
    },
    citymarkers() {
        if (this.app.isLoaded && this.app.mainmap.hasMap) {
            return this.app.mainmap.citymarkers;
        }
    },
    townmarkers() {
        if (this.app.isLoaded && this.app.mainmap.hasMap) {
            return this.app.mainmap.townmarkers;
        }
    },
    landmarkers() {
        if (this.app.isLoaded && this.app.mainmap.hasMap) {
            return this.app.mainmap.landmarkers;
        }
    },
  },
  mounted() {
    if (/sm|xs/.test(this.$vuetify.breakpoint.name)) {
      this.state = false;
    } else {
      this.state = true;
    }
  },
  methods: {
    init() {
        this.markertypes[0].markers = this.citymarkers;
        this.markertypes[1].markers = this.townmarkers;
        this.markertypes[2].markers = this.landmarkers;
    },
    getDrawerStyle() {
      return `
                position: absolute;
                top:48px;
                z-index:401;
                width: ${this.state ? "300" : "0"}px;
            `;
    },
    goToUserProfile(user) {
      window.open(user.link);
    }
  }
};
</script>

<style>
/* .v-navigation-drawer {
    margin-top: 48px;
} */
.prependMarkerLink {
    position: absolute;
    top: 0px;
    left: 2px;
}

.markerTitle {
    font-size: 13px;
    font-weight: 500;
}

.v-list__tile__sub-title {
  font-size: 12px;
}
.v-expansion-panel__header {
    padding-right: 16px;
    padding-left: 5.125rem;
}
.v-expansion-panel__header__icon {
    opacity: 0;
}

#leaders.v-list__tile,
#navbtns.v-list__tile {
  padding: 0px 0px 0px 0px;
}

#navbtn {
  width: 100%;
  height: 100%;
  display: grid;
  grid-template-columns: 1fr 1fr;
}

#navbtn button {
  /* border: 2px solid red; */
  margin: 0px 0px 0px 0px;
  padding: 0px 0px 0px 0px;
  height: 40px;
}

aside.v-navigation-drawer.v-navigation-drawer--clipped.v-navigation-drawer--open.v-navigation-drawer--right.theme--light {
  max-height: calc(100vh - 48px);
  /* width: var(--drawer-width); */
}

.v-navigation-drawer[data-booted="true"] {
  transition: 0.4s var(--quart);
}
</style>
