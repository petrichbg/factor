<template>
  <div class="page-blog">
    <el-hero
      v-if="page == 1 && !tag"
      :headline="setting('blog.headline')"
      :subheadline="setting('blog.subheadline')"
      :image="setting('blog.heroImage')"
    >
      <template v-slot:hero-content>
        <div v-formatted-text="setting('blog.content')" class="content entry-content" />
      </template>
    </el-hero>

    <div class="blog-posts">
      <component :is="setting('blog.components.blogReturnLink')" v-if="tag || page > 1" />
      <div v-if="loading" class="posts-loading">
        <factor-loading-ring />
      </div>
      <div v-else-if="blogPosts.length > 0" class="post-index">
        <div v-for="post in blogPosts" :key="post._id" class="post">
          <component
            :is="setting(`blog.components.${comp}`)"
            v-for="(comp, i) in setting('blog.layout.index')"
            :key="i"
            :post-id="post._id"
            scope="index"
          />
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
    <el-cta />
  </div>
</template>
<script>
import { setting, stored } from "@factor/tools"
import { requestPostIndex } from "@factor/post"
export default {
  components: {
    "el-hero": () => import("../el/hero.vue"),
    "el-cta": () => import("../el/cta.vue")
  },
  data() {
    return {
      postType: "blog",
      loading: false
    }
  },
  routeClass() {
    return ["nav-light"]
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
  serverPrefetch() {
    return this.getPosts()
  },
  computed: {
    tag() {
      return this.$route.params.tag || this.$route.query.tag || ""
    },
    index() {
      return stored(this.postType) || {}
    },
    blogPosts() {
      const { posts = [] } = this.index
      return posts
    },
    page() {
      return this.$route.query.page || 1
    }
  },
  watch: {
    $route: {
      handler: function() {
        this.getPosts()
      }
    }
  },
  mounted() {
    this.getPosts()
  },
  methods: {
    setting,
    async getPosts() {
      this.loading = true

      await requestPostIndex({
        postType: this.postType,
        tag: this.tag,
        status: "published",
        sort: "-date",
        page: this.page,
        limit: setting("blog.limit")
      })

      this.loading = false
    }
  }
}
</script>

<style lang="less">
.page-blog {
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
  .blog-posts {
    padding: 4em 2em;
    line-height: 1.2;
    max-width: 50rem;
    margin: 0 auto;

    .post-index {
      display: grid;
      grid-gap: 3em;
      grid-template-columns: 1fr;
      margin-bottom: 2em;
      @media (max-width: 767px) {
        grid-template-columns: 1fr;
        grid-row-gap: 100px;
      }
      .post {
        border-radius: 8px;
        border: 1px solid rgba(90, 122, 190, 0.08);
        transition: all 0.2s ease-in-out;
        box-shadow: 0 3px 0 0 rgba(90, 122, 190, 0.12);

        &:hover {
          transform: translateY(-6px);
          border: 1px solid rgba(90, 122, 190, 0.08);
          box-shadow: 0 1px 1px 0 rgba(90, 122, 190, 0.1),
            0 10px 20px 0 rgba(90, 122, 190, 0.2);
        }
      }
    }
  }
}
</style>