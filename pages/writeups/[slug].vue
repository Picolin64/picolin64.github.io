<template>
  <main class="min-h-screen">
    <div
      class="prose dark:prose-invert prose-blockquote:not-italic prose-pre:bg-gray-900 prose-img:ring-1 prose-img:ring-gray-200 dark:prose-img:ring-white/10 prose-img:rounded-lg"
    >
      <ContentDoc v-slot="{ doc }" tag="writup">
        <article>
          <h1>{{ doc.title }}</h1>
          <ContentRenderer :value="doc" />
        </article>
      </ContentDoc>
    </div>
  </main>
</template>
<script setup>
const route = useRoute();
const { slug } = route.params;
useSeoMeta({
  ogImage: `https://picolin64.github.io/blog/${slug}.png`,
  twitterCard: "summary_large_image",
  writeupAuthor: "Picolin64",
});
</script>
<style>
.prose h2 a,
.prose h3 a,
.prose h4 a {
  @apply no-underline;
}
.prose img {
  display: block;
  margin-left: auto;
  margin-right: auto;
}
</style>
