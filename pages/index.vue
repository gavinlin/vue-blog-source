<template>
  <div>
    <app-header></app-header>
    <div class="container">
      <div class="row">
        <div
          v-for="article in articles"
          :key="article.slug"
          class="col-md-12 mb-4"
        >
          <NuxtLink
            :to="{ name: 'blog-slug', params: { slug: article.slug } }"
            class="titleLink"
          >
            <div class="mb-4">
              <h3>{{ article.title }}</h3>
            </div>
          </NuxtLink>
          <div class="mb-4">
            <p class="meta">
              <span>
                <i class="fa fa-calendar"></i>
                {{ formateDate(article.createdAt) }}
              </span>
              <span class="dot-divider"></span>
              <span class="badge badge-info">#{{ article.tags }}</span>
            </p>
          </div>
          <p class="mb-4">{{ article.excerpt }}</p>
          <NuxtLink :to="{ name: 'blog-slug', params: { slug: article.slug } }">
            <div class="mb-4">
              <p>Read More ></p>
            </div>
          </NuxtLink>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Header from '../components/Header.vue'
export default {
  async asyncData({ $content, params }) {
    const articles = await $content('articles', params.slug)
      .sortBy('createdAt', 'desc')
      .fetch()
    return {
      articles,
    }
  },
  components: {
    appHeader: Header,
  },
  methods: {
    formateDate(date) {
      const options = { year: 'numeric', month: 'long', day: 'numeric' }
      return new Date(date).toLocaleDateString('en', options)
    },
  },
}
</script>

<style scoped>
.meta {
  font-size: 14px;
}
h3 {
  color: #2d3748;
  font-weight: bold;
}
a.titleLink:hover {
  text-decoration: none !important;
}
</style>
