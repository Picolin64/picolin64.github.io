<template>
  <div>
    <h2 class="uppercase text-sm font-semibold text-gray-400 mb-6">
      WRITEUPS RECIENTES
    </h2>
    <div class="w-3/1 grid grid-cols-3 gap-y-10">
      <div v-for="(writeup, id) in writeups" :key="id">
        <AppWriteupCard :writeup="writeup" />
      </div>
    </div>
    <div class="flex items-center justify-center mt-6 text-sm">
      <UButton
        label="Todos mis writeups &rarr;"
        to="/writeups"
        variant="link"
        color="gray"
      />
    </div>
  </div>
</template>

<script lang="ts" setup>
const { data: writeups } = await useAsyncData("writeups-home", () =>
  queryContent("/writeups")
    .where({ slug: { $ne: "template" } })
    .sort({ published: -1 })
    .limit(3)
    .only(["title", "description", "published", "slug", "image", "_path"])
    .find()
);
</script>