<template>
  <article class="flex w-full xs:flex-col items-center">
    <div class="relative xs:w-full xs:h-84 post-left">
      <ImageMd
        :src="article.img"
        :alt="article.alt"
        class="absolute h-full w-full object-cover header-image"
      />
      <div class="overlay" />
      <div class="absolute top-32 left-32 text-white">
        <NuxtLink to="/">
          <Logo svg-color="white" />
        </NuxtLink>
        <div class="mt-16 -mb-2 flex uppercase text-sm">
          <p class="mr-3">
            {{ formatDate(article.date) }}
          </p>
        </div>
        <h1 class="text-4xl font-bold mb-2">
          {{ article.title }}
        </h1>
        <span v-for="(tag, id) in article.tags" :key="id">
          <NuxtLink :to="`/blog/tag/${tags[tag].slug}`">
            <span
              class="truncate uppercase tracking-wider font-medium text-ss px-2 py-1 rounded-full mr-2 mb-2 border border-light-border dark:border-dark-border transition-colors duration-300 ease-linear"
            >
              {{ tags[tag].name }}
            </span>
          </NuxtLink>
        </span>
      </div>
      <div class="flex absolute top-32 right-32">
        <NuxtLink
          to="/"
          class="mr-8 self-center text-white font-bold hover:underline"
        >
          All articles
        </NuxtLink>
        <AppSearchInput />
      </div>
    </div>
    <div class="px-8 my-8 mx:auto">
      <h1 class="font-bold text-4xl">
        {{ article.title }}
      </h1>
      <p>{{ article.description }}</p>
      <p class="pb-4 text-sm">Published on {{ formatDate(article.date) }}</p>
      <!-- content from markdown -->
      <nuxt-content :document="article" class="prose md:prose-lg" />

      <!-- prevNext component -->
      <PrevNext :prev="prev" :next="next" class="mt-8" />
    </div>
  </article>
</template>
<script>
export default {
  async asyncData({ $content, params }) {
    const article = await $content("articles", params.slug).fetch();
    const tagsList = await $content("tags")
      .only(["name", "slug"])
      .where({ name: { $containsAny: article.tags } })
      .fetch();
    const tags = Object.assign({}, ...tagsList.map((s) => ({ [s.name]: s })));
    const [prev, next] = await $content("articles")
      .only(["title", "slug"])
      .where({ isDraft: false })
      .sortBy("createdAt", "asc")
      .surround(params.slug)
      .fetch();
    return {
      article,
      tags,
      prev,
      next,
    };
  },
  head() {
    return {
      title: this.article.title,
      meta: [
        {
          hid: "description",
          name: "description",
          content: this.article.description,
        },
        // Open Graph
        { hid: "og:title", property: "og:title", content: this.article.title },
        {
          hid: "og:description",
          property: "og:description",
          content: this.article.description,
        },
        {
          hid: "og:type",
          property: "og:type",
          content: "article",
        },
        {
          hid: "og:url",
          property: "og:url",
          content: `https://bulenya.xyz/blog/${this.$route.params.slug}`,
        },
        // Twitter Card
        {
          hid: "twitter:url",
          property: "twitter:url",
          content: `https://bulenya.xyz/blog/${this.$route.params.slug}`,
        },
        {
          hid: "twitter:title",
          name: "twitter:title",
          content: this.article.title,
        },
        {
          hid: "twitter:description",
          name: "twitter:description",
          content: this.article.description,
        },
        {
          hid: "twitter:image",
          name: "twitter:image",
          content: this.article.img,
        },
        {
          hid: "og:image",
          property: "og:image",
          content: this.article.img,
        },
        {
          property: "article:published_time",
          content: this.article.createdAt,
        },
        {
          property: "article:modified_time",
          content: this.article.updatedAt,
        },
        {
          property: "article:tag",
          content: this.article.tags ? this.article.tags.toString() : "",
        },
        { name: "twitter:label1", content: "Written by" },
        { name: "twitter:data1", content: "Pol Milian" },
        { name: "twitter:label2", content: "Filed under" },
        {
          name: "twitter:data2",
          content: this.article.tags ? this.article.tags.toString() : "",
        },
      ],
      link: [
        {
          hid: "canonical",
          rel: "canonical",
          href: `https://bulenya.xyz/blog/${this.$route.params.slug}`,
        },
      ],
    };
  },
  methods: {
    formatDate(date) {
      const options = { year: "numeric", month: "long", day: "numeric" };
      return new Date(date).toLocaleDateString("en", options);
    },
  },
};
</script>
<style>
.header-image {
  filter: brightness(70%) blur(1px);
}
</style>
