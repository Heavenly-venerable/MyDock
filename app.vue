<script setup lang="ts">
import { ref, onMounted } from 'vue'

const q = ref('')
const recentSearches = ref<string[]>([])

type Bookmark = { label: string; url: string }
const bookmarks = ref<Bookmark[]>([])

function loadBookmarks() {
  const saved = localStorage.getItem('bookmarks')
  if (saved) bookmarks.value = JSON.parse(saved)
}

function saveBookmarks() {
  localStorage.setItem('bookmarks', JSON.stringify(bookmarks.value))
}

function addBookmark() {
  const label = prompt('Label situs? (misal: GitHub)')
  const url = prompt('URL situs? (harus diawali http/https)')
  if (label && url && url.startsWith('http')) {
    bookmarks.value.push({ label, url })
    saveBookmarks()
  } else {
    alert('Label dan URL harus valid.')
  }
}

function removeBookmark(index: number) {
  if (confirm('Hapus bookmark ini?')) {
    bookmarks.value.splice(index, 1)
    saveBookmarks()
  }
}

function handleBookmarkClick(bookmark: Bookmark) {
  window.open(bookmark.url, '_blank')
}

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

const showModal = ref(false)
const newBookmarkLabel = ref('')
const newBookmarkURL = ref('')

function openAddBookmarkModal() {
  newBookmarkLabel.value = ''
  newBookmarkURL.value = ''
  showModal.value = true
}

function submitBookmark() {
  if (
    newBookmarkLabel.value.trim() &&
    newBookmarkURL.value.trim().startsWith('http')
  ) {
    bookmarks.value.push({
      label: newBookmarkLabel.value.trim(),
      url: newBookmarkURL.value.trim(),
    })
    saveBookmarks()
    showModal.value = false
  } else {
    alert('Label dan URL harus valid (URL harus mulai dengan http/https).')
  }
}

onMounted(() => {
  const savedSearches = localStorage.getItem('recent-searches')
  if (savedSearches) recentSearches.value = JSON.parse(savedSearches)

  loadBookmarks()
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
      <div class="flex flex-col gap-2">
        <div class="flex items-center justify-between text-sm text-gray-700 dark:text-gray-300">
          <p>Bookmarks:</p>
          <UButton icon="i-heroicons-plus" size="xs" color="gray" variant="ghost" @click="openAddBookmarkModal">
            Tambah
          </UButton>
        </div>
        <ul class="flex flex-wrap gap-2">
          <li v-for="(bm, i) in bookmarks" :key="bm.url" class="flex items-center gap-1">
            <UButton size="xs" color="gray" variant="soft" class="rounded-md" @click="handleBookmarkClick(bm)">
              {{ bm.label }}
            </UButton>
            <UButton icon="i-heroicons-trash" color="red" variant="ghost" size="2xs" @click="removeBookmark(i)" />
          </li>
        </ul>
      </div>
    </div>
  </main>
  <UModal v-model="showModal"
    :ui="{ container: 'flex items-start justify-center', base: 'relative max-w-md w-full p-6 rounded-lg bg-white dark:bg-gray-900' }">
    <div class="p-4 space-y-4">
      <h3 class="text-lg font-semibold">Tambah Bookmark</h3>
      <UInput v-model="newBookmarkLabel" placeholder="Label (misal: GitHub)" />
      <UInput v-model="newBookmarkURL" placeholder="URL (harus diawali http/https)" />
      <div class="flex justify-end gap-2">
        <UButton color="gray" variant="soft" @click="showModal = false">Batal</UButton>
        <UButton color="primary" @click="submitBookmark">Simpan</UButton>
      </div>
    </div>
  </UModal>

</template>
