<template>
  <div class="blog">
    <el-hero
      :subheadline="setting('blog.pretitle')"
      :headline="setting('blog.title')"
      class="blog-hero"
    >
      <template v-slot:hero-content>
        <div v-formatted-text="setting('blog.content')" class="content entry-content" />
      </template>
    </el-hero>
    <section class="blog-posts">
      <div class="mast">
        <div class="blog-entries">
          <div v-if="loading" class="posts-loading">
            <factor-loading-ring />
          </div>
          <div v-else-if="blogPosts.length > 0" class="blog-entries-inner">
            <div v-if="page == 1" class="blog-post">
              <div class="blog-promo">
                <div class="pretitle">{{ setting('blog.promo.pretitle') }}</div>
                <h1 class="title">{{ setting('blog.promo.title') }}</h1>
                <p class="content">{{ setting('blog.promo.content') }}</p>

                <factor-link
                  v-if="setting('blog.promo.button.link')"
                  btn="primary"
                  :path="setting('blog.promo.button.link')"
                  class="mt-8"
                >
                  {{ setting('blog.promo.button.text') }}
                  <factor-icon icon="fas fa-angle-right" />
                </factor-link>
              </div>
            </div>

            <div v-for="post in blogPosts" :key="post._id" class="blog-post">
              <div class="blog-post-inner">
                <component
                  :is="setting(`blog.components.${_component}`)"
                  v-for="(_component, i) in setting('blog.layout.index')"
                  :key="i"
                  :post-id="post._id"
                  format="index"
                />
              </div>
            </div>
          </div>
          <div v-else class="posts-not-found">
            <div class="text">
              <div class="title">{{ setting("blog.notFound.title") }}</div>
              <div class="sub-title">{{ setting("blog.notFound.subTitle") }}</div>
            </div>
          </div>
          <component :is="setting('blog.components.pagination')" :post-type="postType" />
        </div>
      </div>
    </section>
  </div>
</template>
<script lang="ts">
import { factorLoadingRing, factorLink, factorIcon } from "@factor/ui"
import { setting, stored } from "@factor/api"
import { requestPostIndex } from "@factor/post/request"
import Vue from "vue"

export default Vue.extend({
  components: {
    factorLoadingRing,
    factorLink,
    factorIcon,
    "el-hero": () => import("../el/hero.vue")
  },
  data() {
    return {
      postType: "blog",
      loading: false
    }
  },
  routeClass() {
    return "nav-white"
  },
  metaInfo() {
    const title = this.tag ? `Tag "${this.tag}"` : setting("blog.metatags.index.title")

    const description = this.tag
      ? `Articles related to tag: ${this.tag}`
      : setting("blog.metatags.index.description")

    return {
      title,
      description
    }
  },
  computed: {
    index(this: any) {
      return stored(this.postType) || {}
    },
    blogPosts(this: any) {
      const { posts = [] } = this.index
      return posts
    },
    page(this: any) {
      return this.$route.query.page || 1
    }
  },
  watch: {
    $route: {
      handler: function(this: any) {
        this.getPosts()
      }
    }
  },
  serverPrefetch() {
    return this.getPosts()
  },
  mounted() {
    if (this.blogPosts.length == 0) {
      this.getPosts()
    }
  },
  methods: {
    setting,
    getPost(_id: any) {
      return stored(_id) || {}
    },
    async getPosts(this: any) {
      this.loading = true

      const theLimit = this.page === 1 ? setting("blog.limit") - 1 : setting("blog.limit")

      await requestPostIndex({
        postType: this.postType,
        tag: this.tag,
        status: "published",
        sort: "-date",
        page: this.page,
        limit: theLimit
      })

      this.loading = false
    }
  }
})
</script>

<style lang="less">
.plugin-blog {
  .mast {
    max-width: 1000px;
    margin: 0 auto;
  }
  .blog-hero {
    .mast {
      padding: 0.5rem;
    }
    // Light version
    // background-color: #e6ebf1;
    // background-image: url("../img/dot.svg");
    // Dark Version
    // background-color: #1b223c;
    // background-image: url("../img/dot-light.svg");

    // .headline,
    // .subheadline,
    // .hero-content {
    //   color: #f3f4fa;
    // }
  }
  .blog-posts {
    padding-bottom: 2rem;

    .blog-entries {
      .blog-entries-inner {
        display: flex;
        flex-wrap: wrap;
        //padding-top: 3rem; // only needed when hero has bg
        padding-bottom: 3rem;
        @media (max-width: 767px) {
          padding-top: 2rem;
          padding-bottom: 2rem;
        }
      }
    }
  }
  .blog-post {
    padding: 0.5rem;
    width: 33.333333%;
    transition: all 500ms cubic-bezier(0.165, 0.84, 0.44, 1);

    &:hover {
      transform: translateY(-0.5rem);
    }

    .blog-post-inner {
      height: 100%;
      overflow: hidden;
      display: flex;
      flex-direction: column;
      background-color: #fff;
      border-radius: 0.5rem;
      box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1),
        0 10px 10px -5px rgba(0, 0, 0, 0.04);
    }

    @media (max-width: 1200px) {
      width: 100%;
    }
  }
  .blog-promo {
    display: flex;
    flex-direction: column;
    justify-content: center;
    height: 100%;
    padding: 5rem 2rem;
    overflow: hidden;

    color: var(--color-text);
    background-color: #e6ebf1;
    background-image: url("../img/dot.svg");
    // background-image: url(../img/promo.svg);
    // background-repeat: no-repeat;
    // background-size: cover;
    // background-position: center;
    border-radius: 0.5rem;
    box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);

    .pretitle {
      margin-bottom: 0.5rem;
      font-size: 0.875rem;
      font-weight: 600;
      line-height: 1.25;
      letter-spacing: 0.1em;
      text-transform: uppercase;
      color: var(--color-primary);

      @media (max-width: 767px) {
        font-size: 1rem;
      }
    }
    .title {
      margin-top: 0.5rem;
      font-size: 1.5rem;
      letter-spacing: -0.025em;
    }
    .content {
      margin-top: 0.5rem;
      opacity: 0.7;
    }
    a.btn-link {
      margin-top: 2rem;
    }
  }
  .loader .ring-path {
    stroke: var(--color-primary);
  }
  .posts-not-found,
  .posts-loading {
    min-height: 50vh;
    display: flex;
    text-align: center;
    align-items: center;
    justify-content: center;
    .title {
      font-size: 1.4em;
      font-weight: var(--font-weight-bold);
    }
  }
}
</style>
