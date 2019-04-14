<template>
  <div>
    <v-navigation-drawer
      right
      :style="getDrawerStyle()"
      :stateless="dialog"
      v-model="state"
      absolute
      clipped
      hide-overlay
    >
      <v-divider></v-divider>
      <drawertitle />
      <v-divider></v-divider>
      <v-list dense class="pt-0 mt-0">
        
        <drawermarkers ref="drawermarkers" />
        <!-- <v-divider></v-divider> -->
        <draweractions />
      </v-list>
    </v-navigation-drawer>
  </div>
</template>

<script>
import drawertitle from './drawertitle.vue'
import drawermarkers from './drawermarkers.vue'
import draweractions from './draweractions.vue'

export default {
  name: "drawer",
  components: {
    drawertitle,
    drawermarkers,
    draweractions,
  },
  data: () => ({
    state: false,
    citypanel: [false],
    
  }),
  computed: {
    app() {
      return this.$root.$children[0];
    },
    dialog() {
      if (this.app.isLoaded) {
        return this.app.$refs.dialogmarker.dialog;
      }
    }
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
        this.$refs.drawermarkers.init();
    },
    getDrawerStyle() {
      return `
        position: absolute;
        top:48px;
        z-index:401;
        width: ${this.state ? "300" : "0"}px;
      `;
    },
    
  }
};
</script>

<style>
/* .v-navigation-drawer {
    margin-top: 48px;
} */



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
