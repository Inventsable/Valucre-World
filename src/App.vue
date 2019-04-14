<template>
  <v-app>
    <toolbar ref="toolbar"/>
    <drawer ref="drawer"/>
    <v-content>
      <mainmap ref="mainmap" />
    </v-content>
    <div class="fakescreen" v-if="!isLoaded"></div>
    <div class="fullscreen" v-if="isLoaded"></div>
    <loadingicon @done="doneLoading()" v-show="!isLoaded"/>
    <div class="soil mb-1">
      <div @click="goToGitHub()" class="bttn">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 256 249.6">
          <path d="M85.63,203c0,1-1.19,1.86-2.69,1.86-1.7.16-2.89-.67-2.89-1.86,0-1,1.19-1.86,2.69-1.86C84.28,201,85.63,201.8,85.63,203Zm-16.06-2.32c-.36,1,.68,2.22,2.22,2.53,1.35.52,2.89,0,3.2-1s-.67-2.22-2.22-2.69c-1.34-.36-2.83.16-3.2,1.19Zm22.82-.88c-1.5.36-2.53,1.34-2.38,2.53.16,1,1.5,1.7,3,1.34s2.53-1.34,2.37-2.37S93.88,199.63,92.39,199.78ZM126.35,2C54.76,2,0,56.35,0,127.94,0,185.18,36,234.16,87.48,251.4c6.61,1.18,8.93-2.89,8.93-6.25,0-3.2-.15-20.85-.15-31.69,0,0-36.13,7.74-43.72-15.38,0,0-5.88-15-14.35-18.89,0,0-11.82-8.1.83-7.95a27.23,27.23,0,0,1,19.92,13.32c11.31,19.92,30.25,14.19,37.63,10.78,1.18-8.25,4.54-14,8.26-17.39-28.86-3.2-58-7.38-58-57,0-14.2,3.93-21.32,12.19-30.4-1.35-3.36-5.73-17.19,1.34-35C71.17,52.12,96,69.41,96,69.41a121.87,121.87,0,0,1,64.83,0s24.82-17.34,35.61-13.94c7.07,17.91,2.68,31.69,1.34,35.05,8.26,9.13,13.32,16.26,13.32,30.4,0,49.8-30.4,53.78-59.25,57,4.74,4.08,8.77,11.82,8.77,24,0,17.39-.16,38.91-.16,43.15,0,3.35,2.38,7.43,8.93,6.24C221,234.16,256,185.18,256,127.94,256,56.35,197.94,2,126.35,2ZM50.17,180c-.67.52-.52,1.71.36,2.69.82.82,2,1.19,2.68.51.67-.51.52-1.7-.36-2.68C52,179.71,50.84,179.34,50.17,180Zm-5.58-4.18c-.36.68.16,1.5,1.19,2a1.54,1.54,0,0,0,2.22-.36c.36-.68-.15-1.5-1.19-2C45.78,175.16,45,175.32,44.59,175.83Zm16.73,18.38c-.83.67-.52,2.22.67,3.2,1.18,1.19,2.68,1.34,3.35.51.67-.67.36-2.21-.67-3.2C63.54,193.54,62,193.38,61.32,194.21Zm-5.89-7.59c-.82.52-.82,1.86,0,3s2.22,1.7,2.89,1.18c.83-.67.83-2,0-3.2s-2.06-1.7-2.89-1Z" transform="translate(0 -2)"/>
        </svg>
      </div>
    </div>
    <dialogmarker ref="dialogmarker" />
    <linkdialog ref="link" />
  </v-app>
</template>

<script>
import toolbar from './components/toolbar.vue'
import drawer from './components/drawer.vue'
import mainmap from './components/mainmap.vue'
import loadingicon from './components/loadingicon.vue'
import dialogmarker from './components/dialogmarker.vue'
import linkdialog from './components/linkdialog.vue'

export default {
  name: 'App',
  components: {
        toolbar,
        loadingicon,
        drawer,
        mainmap,
        dialogmarker,
        linkdialog,
    },
  data () {
    return {
      isLoaded: false,
      mainmap: null,
      isPrepped: false,
      clickingOnMarker: false,
      permissions: [],
      boards: [
        {
          name: 'Terrenus',
          desc: 'Birthplace of magitech',
          tags: ['test', 'test'],
          landingPage: 'https://www.valucre.com/lore.html/terrenus/terrenus-landing-page/',
          ama: 'https://www.valucre.com/topic/39612-terrenus-ama-20/',
        },
        {
          name: 'Genesaris',
          desc: 'Birthplace of airships',
          tags: ['test', 'test'],
          landingPage: 'https://www.valucre.com/lore.html/genesaris/genesaris-landing-page/',
          ama: 'https://www.valucre.com/topic/41870-genesaris-ama/',
        },
        {
          name: 'Renovatio',
          desc: 'Divine skylands',
          tags: ['test', 'test'],
          landingPage: 'https://www.valucre.com/lore.html/tellusmater/renovatio/renovatio-r689/',
          ama: 'https://www.valucre.com/topic/40375-renovatio-ama/',
        },
        {
          name: 'Alterion',
          desc: 'Birthplace of combat alchemy',
          tags: ['test', 'test'],
          landingPage: 'https://www.valucre.com/lore.html/tellusmater/alterion/the-realm-of-the-crystal-r440/',
          ama: 'https://www.valucre.com/topic/24496-alterion-ama/',
        },
        {
          name: 'Orisia',
          desc: 'Tropical island nation',
          tags: ['test', 'test'],
          landingPage: 'https://www.valucre.com/lore.html/genesaris/orisia/orisia-landing-page/',
          ama: 'https://www.valucre.com/topic/38311-orisia-ask-me-anything/',
        },
        {
          name: 'World',
          desc: 'The whole of Valucre',
          tags: ['test', 'test'],
          landingPage: 'https://www.valucre.com/lore.html/world/valucre-overview/',
          ama: 'https://www.valucre.com/topic/24379-valucre-ama-series/',
        },
      ]
    }
  },
  computed: {
    hasPermissions() {
      if (this.permissions.length)
        return true;
      else 
        return false;
    }
  },
  created() {
    // db.collection('users').get()
    // .then(snapshot => {
    //   console.log(snapshot);
    // })
  },
  mounted() {
    window.addEventListener('scroll', this.checkScroll)
    this.dressStyle();
    this.calcImages();
  },
  methods: {
    findBoardByName(str) {
      return this.boards.find(board => {
        return board.name == str;
      })
    },
    calcImages() {
      let result = 0;
      for (let i = 0; i <= 6; i++)
        result += Math.pow(4, i);
      console.log(`Map is made of ${result} images`);
    },
    doneLoading() {
      this.isLoaded = true;
      this.$refs.drawer.init();
      const self = this;
      // console.log(this.mainmap)
      this.mainmap.doInitAction();
    },
    checkScroll(evt) {
      console.log(evt);
      // evt.preventDefault();
    },
    dressStyle() {
      const app = document.querySelector("#app");
      // app.children[0].style.minWidth = '300px';
    },
    goToGitHub() {
      window.open("https://github.com/Inventsable/Valucre-World");
    },
    getCSS(prop) { return window.getComputedStyle(document.documentElement).getPropertyValue('--' + prop); },
    setCSS(prop, data) { document.documentElement.style.setProperty('--' + prop, data); },
  },
}
</script>


<style>
:root {
  /* --color-bg: #e5e3df; */
  --color-bg: #fff;
  --color-invert: #1d1d1d;
  --color-active: #46a0f5;
  --color-idle: #000;
  --color-class: #ffca23;
  --color-id: #50db29;

  --anno-opacity: .5;

  --quad: cubic-bezier(0.48, 0.04, 0.52, 0.96);
  --quart: cubic-bezier(0.76, 0.00, 0.24, 1.00);
  --quint: cubic-bezier(0.84, 0.00, 0.16, 1.00);

  --drawer-width: 300px;

  --text-titles-color: black;
  --text-titles-stroke: white;
  --text-titles-size: 1.8rem;
  --text-city-size: 1.8rem;
  --text-town-size: 1.5rem;
  --text-land-size: 1.5rem;
  --marker-city-width: 10px;
  --marker-land-width: 7.5px;
  --marker-town-width: 7.5px;

  /* --opacity-anno: 0; */
  /* background-color: var(--color-bg-dark); */
}

svg {
  width: 100%;
}

body::-webkit-scrollbar {
    display: none;
}

.fullscreen {
  -webkit-animation: fadeout 300ms var(--quart);
  -moz-animation: fadeout 300ms var(--quart);
  animation: fadeout 300ms var(--quart);
  position: absolute;
  z-index: 9999;
  width: 100vw;
  height: 100vh;
  background-color: var(--color-bg);
  opacity: 0;
  pointer-events: none;
  /* opacity: var(--anno-opacity); */
}


.fakescreen {
  position: absolute;
  z-index: 9999;
  width: 100vw;
  height: 100vh;
  background-color: var(--color-bg);
}

.theme--light.application {
  background-color: var(--color-bg);
}

@keyframes fadeout {
    0%   { opacity: 1; }
    100% { opacity: 0; }
}

.soil {
  display: flex;
  width: 100vw;
  position: absolute;
  /* bottom: 16px; */
  bottom: 14px;
  right: 6px;
  /* margin-right: .325rem; */

  justify-content: flex-end;
  align-items: center;
}

.bttn {
  margin-left: 4rem;
  color: var(--color-invert) !important;
  /* border: 2 px solid red; */
  background-color: transparent;
  cursor: pointer;
  /* padding: 5px 5px; */
  /* width: 30px; */
  padding: .25rem 0px 0px 1.5rem;
  width: 40px;
  display: flex;
  justify-content: center;
  align-items: center;
}

</style>
