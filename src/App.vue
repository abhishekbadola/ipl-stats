<template>
  <div id="app">
    
  <div class="page-container md-layout-column">
    <md-toolbar class="md-primary">
      <md-button class="md-icon-button" @click="showNavigation = true">
        <md-icon>menu</md-icon>
      </md-button>
      <span class="md-title">Vue IPL Stats</span>

      <div class="md-toolbar-section-end">
          <md-tabs class="md-primary" md-alignment="left" md-sync-route>
            <md-tab id="tab-home" md-label="Home" to="/"></md-tab>
            <md-tab id="tab-pages" md-label="Charts" to="/charts"></md-tab>
          </md-tabs>
      </div>
    </md-toolbar>

    <md-drawer :md-active.sync="showNavigation">
      <md-toolbar md-elevation="0">
        <span class="md-title md-layout-item md-size-100 text-center">IPL Seasons</span>
      </md-toolbar>

      <md-list>
        <md-list-item v-for="(season, index) in seasons" :key="season + index">
          <md-button class="md-layout-item md-size-100" @click="seasonChanged(season)">{{season}}</md-button>
        </md-list-item>
      </md-list>
    </md-drawer>

    <md-content>
      <router-view/>
    </md-content>
  </div>
  </div>
</template>

<script>
import router from './router';
import { bus } from "./main";
export default {
  name: "app",
  data: function() {
    return {
      showNavigation: false,
      seasons: ['2016', '2015', '2014', '2013', '2012', '2011', '2010', '2009', '2008']
    };
  },
  created() {

  },
  methods: {
    seasonChanged: function(season){
      this.showNavigation = false;
      bus.$emit("season", season);
      router.push({ path: '/' })
    }
  }
};
</script>

<style>
.page-container {
  overflow: hidden;
  position: relative;
  border: 1px solid rgba(#000, 0.12);
}
.md-drawer {
  position: fixed;
  width: 230px;
  max-width: calc(100vw - 125px);
}
.md-overlay {
  position: fixed;
}
.md-content {
  padding: 16px;
  height: 100%;
}
.text-center {
  text-align: center;
}
</style>
