<template>
  <main class="min-h-screen">
    <AppHeader class="mb-16" title="Blog" :description="description" />
    <ul class="space-y-16">
      <li v-for="(article, id) in articles" :key="id">
        <AppArticleCard :article="article" />
      </li>
    </ul>
  </main>
</template>

<script setup>
const description =
  "Mi blog personal, donde recopilo mis pensamientos y experiencias relacionados a distintos tópicos del campo de la ciberseguridad.";
useSeoMeta({
  title: "Blog | Picolin64",
  description,
});

const { data: articles } = await useAsyncData("all-articles", () =>
  queryContent("/blog").where({ slug: { $ne: "template" } }).sort({ published: -1 }).find()
);
</script>
