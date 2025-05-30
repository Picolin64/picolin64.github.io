<template>
  <main class="min-h-screen">
    <AppHeader class="mb-16" title="Writeups" :description="description" />
    <ul class="space-y-16">
      <li v-for="(writeup, id) in writeups" :key="id">
        <AppWriteupCard :writeup="writeup" />
      </li>
    </ul>
  </main>
</template>

<script setup>
const description =
  "Todos los writeups de desafíos, laboratorios y maquinas que he resuelto en plataformas como TryHackMe y Hack The Box.";
useSeoMeta({
  title: "Writeups | Picolin64",
  description,
});

const { data: writeups } = await useAsyncData("all-writeups", () =>
  queryContent("/writeups").where({ slug: { $ne: "template" } }).sort({ published: -1 }).find()
);
</script>
