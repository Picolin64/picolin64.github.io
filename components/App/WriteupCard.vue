<template>
  <NuxtLink :to="writeup._path" class="group">
    <article>
      <time
        class="relative z-10 order-first mb-3 flex items-center text-sm text-gray-400 dark:text-gray-500 pl-3.5"
        datetime="2022-09-05"
        ><span
          class="absolute inset-y-0 left-0 flex items-center"
          aria-hidden="true"
          ><span
            class="h-4 w-0.5 rounded-full bg-gray-200 dark:bg-gray-500"
          ></span
        ></span>
        {{ getReadableDate(writeup.published) }}
      </time>
      <h2
        class="text-base font-semibold font-display tracking-tight text-gray-800 dark:text-gray-100 group-hover:text-primary-600"
      >
        {{ writeup.title }}
      </h2>
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