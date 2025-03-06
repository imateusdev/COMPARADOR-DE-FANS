<template>
  <div class="min-h-screen bg-gray-100 dark:bg-gray-900">
    <header v-show="!selectedFans.length" class="bg-gradient-to-r from-blue-600 to-purple-600 shadow-lg sticky top-0 z-50">
      <div class="max-w-[98%] mx-auto px-4 py-6">
        <div class="flex items-center justify-between">
          <h1 class="text-3xl font-bold text-white tracking-wider">
            COMPARADOR DE FANS
            <span class="text-yellow-300 text-xl ml-2">★</span>
          </h1>
          <button @click="toggleDarkMode" class="p-3 rounded-full bg-white/10 hover:bg-white/20 transition-colors">
            <span v-if="isDarkMode" class="text-xl">🌞</span>
            <span v-else class="text-xl">🌙</span>
          </button>
        </div>
      </div>
    </header>

    <main class="max-w-[98%] mx-auto px-4 py-8">
      <FanTable 
        :fans="filteredFans"
        :selected-fans="selectedFans"
        @select-fan="toggleFanSelection"
      />

      <Transition name="slide-fade">
        <div v-if="selectedFans.length > 0" 
             class="fixed bottom-0 left-0 right-0 z-40 bg-gradient-to-t from-gray-900/95 to-gray-900/80 backdrop-blur-sm shadow-2xl"
        >
          <div class="max-w-[98%] mx-auto p-6">
            <FanComparison
              :fans="selectedFans"
              @remove-fan="removeFanFromComparison"
            />
          </div>
        </div>
      </Transition>

      <!-- Créditos -->
      <footer class="mt-12 py-8 border-t border-gray-200 dark:border-gray-800">
        <div class="max-w-[98%] mx-auto">
          <div class="flex flex-col items-center justify-center gap-4 text-center">
            <div class="bg-gradient-to-r from-purple-600 to-blue-600 bg-clip-text">
              <p class="text-transparent text-lg font-bold">Créditos</p>
            </div>
            
            <div class="space-y-2">
              <p class="text-gray-600 dark:text-gray-400">
                Informações e testes por 
                <a 
                  href="https://www.youtube.com/@rockmgameroficial" 
                  target="_blank" 
                  rel="noopener noreferrer"
                  class="font-medium text-blue-600 hover:text-blue-500 dark:text-blue-400 dark:hover:text-blue-300 transition-colors underline decoration-2 decoration-blue-400/30 hover:decoration-blue-400"
                >
                  Rockm Gamer
                </a>
              </p>
              
              <p class="text-gray-600 dark:text-gray-400">
                Desenvolvido por 
                <a 
                  href="https://www.linkedin.com/in/mateus-anacleto-18b548237/" 
                  target="_blank" 
                  rel="noopener noreferrer"
                  class="font-medium text-purple-600 hover:text-purple-500 dark:text-purple-400 dark:hover:text-purple-300 transition-colors underline decoration-2 decoration-purple-400/30 hover:decoration-purple-400"
                >
                  Mateus Anacleto
                </a>
              </p>
            </div>
          </div>
        </div>
      </footer>
    </main>
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue'
import FanTable from './components/FanTable.vue'
import FanComparison from './components/FanComparison.vue'
import fansData from './data/fans'

export default {
  name: 'App',
  components: {
    FanTable,
    FanComparison
  },
  setup() {
    const isDarkMode = ref(false)
    const fans = ref(fansData)
    const selectedFans = ref([])
    const searchQuery = ref('')

    const filteredFans = computed(() => {
      if (!searchQuery.value) return fans.value
      const query = searchQuery.value.toLowerCase()
      return fans.value.filter(fan => fan[0].toLowerCase().includes(query))
    })

    const toggleDarkMode = () => {
      isDarkMode.value = !isDarkMode.value
      document.documentElement.classList.toggle('dark', isDarkMode.value)
    }

    const toggleFanSelection = (fan) => {
      const index = selectedFans.value.findIndex(f => f[0] === fan[0])
      if (index >= 0) {
        selectedFans.value.splice(index, 1)
      } else if (selectedFans.value.length < 2) {
        selectedFans.value.push(fan)
      }
    }

    const removeFanFromComparison = (fan) => {
      const index = selectedFans.value.findIndex(f => f[0] === fan[0])
      if (index >= 0) {
        selectedFans.value.splice(index, 1)
      }
    }

    onMounted(() => {
      isDarkMode.value = window.matchMedia('(prefers-color-scheme: dark)').matches
      document.documentElement.classList.toggle('dark', isDarkMode.value)
    })

    return {
      isDarkMode,
      filteredFans,
      selectedFans,
      searchQuery,
      toggleDarkMode,
      toggleFanSelection,
      removeFanFromComparison
    }
  }
}
</script>

<style>
.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: all 0.3s ease;
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  transform: translateY(100%);
  opacity: 0;
}
</style>