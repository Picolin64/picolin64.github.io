<template>
  <div>
    <h2 class="uppercase text-sm font-semibold text-gray-400 mb-4">ENCUENTRAME EN</h2>
    <div class="space-y-5">
      <NuxtLink
        v-for="link in links"
        :key="link.icon"
        :to="link.url"
        target="_blank"
        external
        class="flex items-end gap-4 dark:hover:text-lime-500 group"
      >
        <span class="text-base">
          {{ link.name }}
        </span>
        <div
          class="flex-1 border-b border-dashed border-gray-300 dark:border-gray-800 group-hover:border-gray-700"
        ></div>
        <Icon :name="link.icon" class="w-9 h-9"></Icon>
      </NuxtLink>
    </div>
  </div>
</template>

<script lang="ts" setup>
const links = [
  {
    name: "LinkedIn",
    url: "https://www.linkedin.com/in/david-cleves-guzm%C3%A1n-4b9214369/",
    icon: "simple-icons:linkedin",
  },
  {
    name: "GitHub",
    url: "https://github.com/Picolin64",
    icon: "simple-icons:github",
  },
  {
    name: "TryHackMe",
    url: "https://tryhackme.com/p/Picolin64",
    icon: "simple-icons:tryhackme",
  },
  {
    name: "Hack The Box",
    url: "https://app.hackthebox.com/profile/2012205",
    icon: "simple-icons:hackthebox",
  }
];
</script>
