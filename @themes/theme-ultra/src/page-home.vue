<template>
  <div class="page-home">
    <sectionIntro />
    <sectionAbout />
    <sectionServices />
    <sectionPortfolio />
    <sectionNews />
    <sectionContact />
  </div>
</template>

<script lang="ts">
import Vue from "vue"
export default Vue.extend({
  components: {
    sectionIntro: () => import("./section-intro.vue"),
    sectionAbout: () => import("./section-about.vue"),
    sectionServices: () => import("./section-services.vue"),
    sectionPortfolio: () => import("./section-portfolio.vue"),
    sectionNews: () => import("./section-news.vue"),
    sectionContact: () => import("./section-contact.vue")
  },
  data() {
    return {
      loading: true
    }
  }
})
</script>

<style lang="less">
</style>
