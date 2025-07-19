<script setup lang="ts">
const q = ref('')

const recentSearches = ref<string[]>([])

function saveSearch(term: string) {
  recentSearches.value = [term, ...recentSearches.value.filter(i => i !== term)].slice(0, 5)
  localStorage.setItem('recent-searches', JSON.stringify(recentSearches.value))
}

function searchYouTube() {
  if (!q.value.trim()) return
  const query = encodeURIComponent(q.value)
  saveSearch(q.value)
  window.open(`https://www.youtube.com/results?search_query=${query}`, '_blank')
}

onMounted(() => {
  const saved = localStorage.getItem('recent-searches')
  if (saved) recentSearches.value = JSON.parse(saved)
})
</script>

<template>
  <main class="w-full min-h-svh flex items-center justify-center p-4 bg-background text-foreground transition-colors">
    <div class="flex flex-col gap-6 w-full max-w-md">
      <form @submit.prevent="searchYouTube" class="flex items-center gap-2">
        <UInput v-model="q" name="q" placeholder="Search..." icon="i-heroicons-magnifying-glass-20-solid"
          autocomplete="off" :ui="{ icon: { trailing: { pointer: '' } } }" class="flex-1 placeholder:opacity-60">
          <template #trailing>
            <UButton v-show="q !== ''" color="gray" variant="link" icon="i-heroicons-x-mark-20-solid" :padded="false"
              @click="q = ''" />
          </template>
        </UInput>
        <UButton type="submit" icon="i-heroicons-arrow-right-circle-20-solid" color="primary" variant="solid" />
      </form>
      <div v-if="recentSearches.length" class="text-sm text-gray-700 dark:text-gray-300">
        <p class="mb-2">Recent searches:</p>
        <ul class="flex gap-2 flex-wrap">
          <li v-for="item in recentSearches" :key="item">
            <UButton size="xs" color="gray" variant="soft" @click="q = item"
              class="rounded-md shadow-sm hover:shadow transition">
              {{ item }}
            </UButton>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>
