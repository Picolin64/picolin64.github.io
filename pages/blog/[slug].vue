<template>
  <main class="min-h-screen">
    <div
      class="prose dark:prose-invert prose-code:before:hidden prose-code:after:hidden prose-code:text-base prose-code:bg-gray-300 dark:prose-code:bg-gray-700 prose-blockquote:not-italic before:[&_blockquote_p:first-of-type]:content-none after:[&_blockquote_p:last-of-type]:content-none prose-pre:bg-gray-900 prose-img:ring-1 prose-img:ring-gray-200 dark:prose-img:ring-white/10 prose-img:rounded-lg prose-blockquote:border-lime-500 max-w-5xl"
    >
      <ContentDoc v-slot="{ doc }" tag="article">
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
  ogType: "website",
  ogImage: `https://picolin64.github.io/blog/${slug}/thumbnail.png`,
  ogUrl: `https://picolin64.github.io/blog/${slug}/`,
  twitterCard: "summary_large_image",
  writeupAuthor: "Picolin64",
});
</script>
<style>
.prose h3 a,
.prose h4 a {
  @apply no-underline;
}
.prose img {
  display: block;
  margin-left: auto;
  margin-right: auto;
}
details {
  padding: 9px;
  border: 2px solid rgba(132, 204, 22, 0.6);
  border-radius: 10px;
}
code[class=""]{
  padding-left: 4px;
  padding-right: 4px;
  padding-bottom: 2px;
  border-radius: 5px;
}
</style>
