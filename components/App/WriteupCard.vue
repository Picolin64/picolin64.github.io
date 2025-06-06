<template>
  <NuxtLink :to="writeup._path" class="group">
    <article>
      <NuxtImg :src="writeup.image" alt="writeup-thumbnail"
      class="ring-2 border ring-gray-200 border-gray-300 dark:ring-white/10 dark:border-gray-800 hover:ring-4 transition-all duration-300 bg-gray-200 dark:bg-gray-900 rounded-full h-20 w-20 sm:h-24 sm:w-24 mb-1"
      sizes="96px sm:128px" placeholder format="webp" />
      <h2
        class="text-lg text-wrap font-semibold font-display tracking-tight text-gray-800 dark:text-gray-100 group-hover:text-primary-600"
      >
        {{ writeup.title }}
      </h2>
      <time
        class="relative z-10 order-first mb-3 text-sm text-gray-400 dark:text-gray-500"
        datetime="2022-09-05"
        >
        {{ getReadableDate(writeup.published) }}
      </time>
    </article>
  </NuxtLink>
</template>

<script setup>
defineProps({
  writeup: {
    type: Object,
    required: true,
  },
});

const getReadableDate = (dateString) => {
  const date = new Date(dateString);
  return date.toLocaleDateString("es-CO", {
    year: "numeric",
    month: "long",
    day: "numeric",
  });
};
</script>