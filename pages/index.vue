<template>
  <div>
    <nav class="navbar navbar-light bg-light mb-4">
      <a class="navbar-brand" href="#">Blog</a>
    </nav>
    <div class="container">
      <div class="row">
        <div
          v-for="article in articles"
          :key="article.slug"
          class="col-md-12 mb-4"
        >
          <NuxtLink :to="{ name: 'blog-slug', params: { slug: article.slug } }">
            <div class="mb-4">
              <h3>{{ article.title }}</h3>
            </div>
          </NuxtLink>
          <div class="mb-4">
            <p class="meta">
              <span class="mr-2">
                <i class="fa fa-calendar"></i>
                {{ formateDate(article.createdAt) }}
              </span>
              <span>
                <i class="fa fa-folder-o"></i>
                {{ article.category }}
              </span>
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
export default {
  async asyncData({ $content, params }) {
    const articles = await $content('articles', params.slug)
      .sortBy('createdAt', 'asc')
      .fetch()
    return {
      articles,
    }
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
.container {
  font-family: 'Poppins', Arial, Helvetica, sans-serif;
}
.meta {
  font-size: 14px;
}
h3 {
  color: black;
}
NuxtLink div h3:hover {
  text-decoration: none !important;
}
</style>
