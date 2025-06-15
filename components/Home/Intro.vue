<template>
  <div class="space-y-6">
    <NuxtImg src="/avatar.jpg" alt="Picolin64"
      class="ring-2 border ring-gray-200 border-gray-300 dark:ring-white/10 dark:border-gray-800 hover:ring-4 transition-all duration-300 bg-gray-200 dark:bg-gray-900 rounded-full h-12 w-12 sm:h-16 sm:w-16"
      sizes="48px sm:64px" placeholder format="webp" />
    <p class="text-gray-900 dark:text-gray-400">
      <b>¡Hola!</b> Soy Picolin64, estudiante de Ingeniería de Sistemas y apasionado por la ciberseguridad. 
    </p>
    <p class="text-gray-900 dark:text-gray-400">
      Aquí encontraras mis proyectos, <i>writeups</i> y posts de ciberseguridad. Descubrirás mi metodología de <i>pentesting</i> y mis pensamientos, experiencias y aportes en este campo. Este es mi sitio para recopilar todo lo que he aprendido en mi travesía por el apasionante mundo de la ciberseguridad y compartirlo contigo.
    </p>
  </div>
</template>

<script setup>
useSeoMeta({
  ogType: "website",
  ogTitle: "Picolin64",
  ogDescription: "Soy Picolin64, estudiante de Ingeniería de Sistemas y apasionado por la ciberseguridad. Este es mi blog personal de ciberseguridad.",
  ogImage: "https://picolin64.github.io/favicon.ico",
  ogUrl: "https://picolin64.github.io/",
  twitterCard: "summary_large_image"
});
</script>
