<template>
  <main class="min-h-screen">
    <AppHeader class="mb-16" title="Writeups" :description="description" />
    <div class="w-3/1 grid grid-cols-3 gap-y-10">
      <div v-for="(writeup, id) in writeups" :key="id">
        <AppWriteupCard :writeup="writeup" />
      </div>
    </div>
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
