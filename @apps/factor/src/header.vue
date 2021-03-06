<template>
  <div class="site-head">
    <div class="site-head-pad">
      <site-brand class="site-brand" />
      <div class="head-nav page-nav">
        <factor-link
          v-for="navItem in pageNav"
          :key="navItem.path"
          v-formatted-text="navItem.name"
          :path="navItem.path"
        />
        <github-stars />
      </div>
      <div class="head-nav action-nav">
        <account-menu v-if="isLoggedIn()" />
        <factor-link v-else event="sign-in-modal">Sign In</factor-link>
        <factor-link
          v-if="$route.path != '/install'"
          path="/install"
          btn="primary"
        >Install Factor &rarr;</factor-link>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import Vue from "vue"
import { factorLink } from "@factor/ui"
import { setting } from "@factor/api"
import { isLoggedIn } from "@factor/user"
import { accountMenu } from "@factor/plugin-standard-signin"
export default Vue.extend({
  components: {
    factorLink,
    accountMenu,
    siteBrand: () => import("./el/brand.vue"),
    githubStars: () => import("./el/github-stars.vue")
  },
  data() {
    return {
      navConfig: setting("site.nav"),
      pageNav: [
        { path: "/guide", name: "Documentation" },
        { path: "/themes", name: "Themes" },
        { path: "/plugins", name: "Plugins" }
      ],
      actionNav: [
        {
          event: "sign-in-modal",
          name: "Sign In &rarr;",
          condition: (): boolean => !isLoggedIn()
        },
        { component: accountMenu, condition: (): boolean => isLoggedIn() }
      ]
    }
  },
  computed: {
    siteNav(this: any) {
      return this.navConfig.filter(item => !item.condition || item.condition())
    }
  },
  methods: { isLoggedIn }
})
</script>
<style lang="less">
.nav-light:not(.scrolled) .site-head {
  color: #fff;
  background: transparent;
}

.site-head {
  transition: all 0.1s;

  padding: 0 1.5em;
  position: fixed;
  width: 100%;
  z-index: 5000;
  font-weight: 600;
}

.factor-site {
  .site-head {
    opacity: 0;
  }
  &.scrolled,
  &.top {
    .site-head {
      opacity: 1;
    }
  }
  &.scrolled {
    .site-head {
      padding: 0rem 2rem;
      background: #fff;
      box-shadow: 0 1px 1px rgba(0, 0, 0, 0.1);
    }
  }
}

.site-head-pad {
  height: 45px;
  align-items: center;
  display: grid;
  grid-template-columns: minmax(150px, 300px) 1fr 300px;

  .head-nav {
    display: flex;
    align-items: center;

    &.page-nav {
      justify-content: center;
      .github-actions {
        margin-left: 1rem;
        width: 140px;
      }
    }
    &.action-nav {
      justify-content: flex-end;
    }

    @media (max-width: 767px) {
      flex-grow: 2;
      justify-content: flex-end;
    }
    > .factor-link {
      color: inherit;

      margin: 0 1em;
      &:last-child {
        margin-right: 0;
      }
      @media (max-width: 767px) {
        margin: 0 1em;
        .fa {
          display: none;
        }
      }
      &:hover,
      &.router-link-active {
        opacity: 0.8;
      }
      &:active {
        opacity: 0.5;
      }
      &.factor-link {
        .fa {
          margin-right: 4px;
          opacity: 0.7;
        }
      }
    }
    .account-menu {
      margin-left: 1em;
    }
  }
}
</style>
