<template>
  <main class="min-h-screen">
    <AppHeader class="mb-12" title="Sobre mi" :description="description" />
    <div class="space-y-6">
      <NuxtImg src="/avatar.jpg" alt="Picolin64"
      class="ring-2 border ring-gray-200 border-gray-300 dark:ring-white/10 dark:border-gray-800 hover:ring-4 transition-all duration-300 bg-gray-200 dark:bg-gray-900 rounded-full h-12 w-12 sm:h-16 sm:w-16"
      sizes="48px sm:64px" placeholder format="webp" />
      <h1 class="text-xl font-bold tracking-tight text-gray-800 dark:text-gray-100">
        David Cleves Guzmán
      </h1>
      <p class="text-gray-900 dark:text-gray-400">
        Soy estudiante de Ingeniería de Sistemas y aspirante a operador de Red Team. 
      </p>
      <p class="text-gray-900 dark:text-gray-400">
        En las ultimas etapas de mi formación profesional, me encontraba perdido sobre la forma en que debía enfocar mi carrera. Por circunstancias casuales y nada relacionadas, tuve mi primer contacto con el campo de la ciberseguridad. Y finalmente encontré mi pasión.
      </p>
      <p class="text-gray-900 dark:text-gray-400">
        Desde entonces, dedico gran parte de mi tiempo libre a formarme en la ciberseguridad ofensiva y escribir todo lo que aprendo en mi base de conocimientos personal. Ahora es hora de compartir ese conocimiento contigo.
      </p>
      <p class="text-gray-900 dark:text-gray-400">
        Soy fiel creyente de que la mejor manera de aprender, es enseñando. Es por eso que he decidido crear este sitio web para compartir con el mundo todo lo que he aprendido y sigo aprendiendo sobre la ciberseguridad y el <i>hacking</i> ético.
      </p>
    </div>
    <SocialLinks class="mt-24"/>
  </main>
</template>

<script setup>
import SocialLinks from '~/components/Home/SocialLinks.vue';

const description = "";
useSeoMeta({
  title: "Sobre mi | Picolin64",
  description,
});
</script>