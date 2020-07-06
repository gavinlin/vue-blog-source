<template>
  <article>
    <!-- <pre>{{ article }}</pre> -->
    <app-header></app-header>
    <div class="container">
      <h2 class="title mb-4">{{ article.title }}</h2>
      <p class="mb-4">
        <span>
          <i class="fa fa-calendar"></i>
          {{ formateDate(article.createdAt) }}
        </span>
        <span class="dot-divider"></span>
        <span class="badge badge-info">#{{ article.tags }}</span>
      </p>
      <nuxt-content :document="article" />
    </div>
  </article>
</template>

<script>
import Header from '../../components/Header'
export default {
  async asyncData({ $content, params }) {
    const article = await $content('articles', params.slug).fetch()

    return { article }
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
.title {
  font-weight: bold;
}
</style>
