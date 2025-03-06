<template>
  <div class="bg-white dark:bg-gray-800 rounded-lg shadow-xl overflow-hidden">
    <!-- Barra de pesquisa -->
    <div class="p-4 border-b dark:border-gray-700">
      <div class="relative">
        <input 
          type="text"
          v-model="searchText"
          placeholder="Buscar por marca ou modelo..."
          class="w-full px-4 py-2 rounded-lg bg-gray-50 dark:bg-gray-700 border border-gray-200 
                 dark:border-gray-600 focus:ring-2 focus:ring-blue-500 dark:focus:ring-blue-400 
                 focus:border-transparent transition-all"
          @input="$emit('search', searchText)"
        />
        <span class="absolute right-3 top-2.5 text-gray-400">🔍</span>
      </div>
    </div>

    <!-- Controles superiores -->
    <div class="p-4 border-b dark:border-gray-700 space-y-4">
      <div class="flex flex-wrap items-center gap-4">
        <div class="flex flex-wrap gap-2">
          <button 
            v-for="(column, index) in sortableColumns" 
            :key="column.key"
            @click="toggleSort(column.key)"
            class="px-4 py-2 rounded-lg flex items-center gap-2 text-sm font-medium transition-all transform hover:scale-105"
            :class="getButtonStyle(column.key, index)"
          >
            {{ column.label }}
            <span v-if="sortConfig.key === column.key" class="text-xs">
              {{ sortConfig.direction === 'asc' ? '↑' : '↓' }}
            </span>
          </button>
        </div>

        <button 
          @click="clearFilters"
          class="px-4 py-2 text-sm font-medium rounded-lg transition-all transform hover:scale-105
                 bg-gradient-to-r from-red-500 to-pink-500 text-white shadow-lg hover:from-red-600 hover:to-pink-600"
          :class="{ 'opacity-50 cursor-not-allowed': !sortConfig.key }"
          :disabled="!sortConfig.key"
        >
          Limpar Filtros
        </button>

        <div class="flex items-center gap-4 ml-auto">
          <label class="text-sm text-gray-600 dark:text-gray-300">Itens por página:</label>
          <select 
            v-model.number="itemsPerPage" 
            @change="handleItemsPerPageChange"
            class="bg-gray-50 dark:bg-gray-700 border border-gray-200 dark:border-gray-600 rounded-lg px-3 py-1.5 text-sm"
          >
            <option :value="10">10 por página</option>
            <option :value="20">20 por página</option>
            <option :value="50">50 por página</option>
            <option :value="100">100 por página</option>
          </select>
        </div>
      </div>
    </div>

    <div class="overflow-x-auto">
      <table class="w-full divide-y divide-gray-200 dark:divide-gray-700">
        <thead class="bg-gray-50 dark:bg-gray-700">
          <tr>
            <th v-for="column in columns" 
                :key="column.key" 
                class="px-6 py-3 text-left text-xs font-medium text-gray-500 dark:text-gray-300 uppercase tracking-wider whitespace-nowrap"
                :class="{'text-right': column.key !== 'model'}"
            >
              {{ column.label }}
            </th>
            <th class="px-6 py-3 text-right">Ações</th>
          </tr>
        </thead>
        <tbody class="bg-white dark:bg-gray-800 divide-y divide-gray-200 dark:divide-gray-700">
          <tr v-for="fan in paginatedFans" 
              :key="fan[0]"
              :class="{'bg-blue-50 dark:bg-blue-900/20': isSelected(fan)}"
              class="hover:bg-gray-50 dark:hover:bg-gray-700/50 transition-colors"
          >
            <td v-for="(column, index) in columns" 
                :key="column.key"
                class="px-6 py-4 whitespace-nowrap"
                :class="{'text-right': index > 0}"
            >
              <span :class="[
                getValueClass(fan[index], column.key),
                index === 0 ? 'font-medium text-gray-900 dark:text-white' : ''
              ]">
                {{ fan[index] === '?' || fan[index] === '#VALOR!' ? 'SEM ESSA INFO' : fan[index] }}
              </span>
            </td>
            <td class="px-6 py-4 text-right whitespace-nowrap">
              <button 
                @click="$emit('select-fan', fan)"
                class="text-sm font-medium px-4 py-2 rounded-lg transition-colors"
                :class="isSelected(fan) ? 
                  'bg-red-100 text-red-600 hover:bg-red-200 dark:bg-red-900/20 dark:hover:bg-red-900/30' : 
                  'bg-blue-100 text-blue-600 hover:bg-blue-200 dark:bg-blue-900/20 dark:hover:bg-blue-900/30'"
              >
                {{ isSelected(fan) ? 'Remover' : 'Comparar' }}
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Paginação -->
    <div class="px-6 py-4 bg-gray-50 dark:bg-gray-700/50 border-t dark:border-gray-700">
      <div class="flex flex-wrap items-center justify-between gap-4">
        <span class="text-sm text-gray-700 dark:text-gray-300">
          Mostrando {{ Math.min(startIndex + 1, filteredFans.length) }} - {{ Math.min(endIndex, filteredFans.length) }} 
          de {{ filteredFans.length }} itens
        </span>

        <div class="flex flex-wrap items-center gap-2">
          <button 
            @click="goToPage(currentPage - 1)"
            :disabled="currentPage === 1"
            class="px-4 py-2 text-sm font-medium rounded-lg transition-colors"
            :class="currentPage === 1 ? 
              'bg-gray-100 text-gray-400 dark:bg-gray-700 dark:text-gray-500' :
              'bg-white text-gray-700 hover:bg-gray-50 dark:bg-gray-800 dark:text-gray-300 dark:hover:bg-gray-700'"
          >
            Anterior
          </button>

          <div class="flex items-center gap-1">
            <template v-for="pageNum in visiblePageNumbers" :key="pageNum">
              <button
                v-if="pageNum !== '...'"
                @click="goToPage(pageNum)"
                class="px-3 py-2 text-sm font-medium rounded-lg transition-colors"
                :class="pageNum === currentPage ? 
                  'bg-blue-500 text-white' : 
                  'bg-white text-gray-700 hover:bg-gray-50 dark:bg-gray-800 dark:text-gray-300 dark:hover:bg-gray-700'"
              >
                {{ pageNum }}
              </button>
              <span 
                v-else 
                class="px-2 text-gray-500 dark:text-gray-400"
              >
                ...
              </span>
            </template>
          </div>

          <button 
            @click="goToPage(currentPage + 1)"
            :disabled="currentPage >= totalPages"
            class="px-4 py-2 text-sm font-medium rounded-lg transition-colors"
            :class="currentPage >= totalPages ? 
              'bg-gray-100 text-gray-400 dark:bg-gray-700 dark:text-gray-500' :
              'bg-white text-gray-700 hover:bg-gray-50 dark:bg-gray-800 dark:text-gray-300 dark:hover:bg-gray-700'"
          >
            Próxima
          </button>

          <select 
            v-model.number="itemsPerPage" 
            @change="handleItemsPerPageChange"
            class="bg-gray-50 dark:bg-gray-700 border border-gray-200 dark:border-gray-600 rounded-lg px-3 py-1.5 text-sm"
          >
            <option :value="10">10 por página</option>
            <option :value="20">20 por página</option>
            <option :value="50">50 por página</option>
            <option :value="100">100 por página</option>
          </select>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue'

export default {
  name: 'FanTable',
  props: {
    fans: {
      type: Array,
      required: true
    },
    selectedFans: {
      type: Array,
      required: true
    }
  },
  setup(props) {
    const searchText = ref('')
    const columns = [
      { key: 'model', label: 'Marca/Modelo' },
      { key: 'consumption', label: 'Consumo' },
      { key: 'blades', label: 'Hélices' },
      { key: 'maxRpm', label: 'RPM MÁX' },
      { key: 'cfmNormal', label: 'CFM/27.13 Normal' },
      { key: 'pressure', label: 'Pressão Entregue' },
      { key: 'testRpm', label: 'RPM FAN Teste' },
      { key: 'cfmDelivered', label: 'CFM Entregue' },
      { key: 'cfmVariation', label: 'Variação CFM' },
      { key: 'pressureMMH2O', label: 'Pressão MMH2O' },
      { key: 'pressureAirCooler', label: 'Pressão Air Cooler' },
      { key: 'pressureVariation', label: 'Variação Pressão' },
      { key: 'average', label: 'Média' }
    ]

    const sortableColumns = [
      { key: 'maxRpm', label: 'RPM' },
      { key: 'cfmDelivered', label: 'CFM' },
      { key: 'pressureMMH2O', label: 'Pressão' }
    ]

    const sortConfig = ref({ key: '', direction: 'asc' })
    const currentPage = ref(1)
    const itemsPerPage = ref(20)

    const filteredFans = computed(() => {
      if (!sortConfig.value.key) return props.fans

      return [...props.fans].sort((a, b) => {
        const columnIndex = columns.findIndex(col => col.key === sortConfig.value.key)
        const aValue = parseFloat(a[columnIndex]) || 0
        const bValue = parseFloat(b[columnIndex]) || 0
        
        return sortConfig.value.direction === 'asc' 
          ? aValue - bValue 
          : bValue - aValue
      })
    })

    const totalPages = computed(() => Math.max(1, Math.ceil(filteredFans.value.length / itemsPerPage.value)))
    const startIndex = computed(() => (currentPage.value - 1) * itemsPerPage.value)
    const endIndex = computed(() => Math.min(startIndex.value + itemsPerPage.value, filteredFans.value.length))

    const paginatedFans = computed(() => {
      return filteredFans.value.slice(startIndex.value, endIndex.value)
    })

    const toggleSort = (key) => {
      if (sortConfig.value.key === key) {
        sortConfig.value.direction = sortConfig.value.direction === 'asc' ? 'desc' : 'asc'
      } else {
        sortConfig.value.key = key
        sortConfig.value.direction = 'asc'
      }
      currentPage.value = 1 // Reset to first page when sorting
    }

    const getSortButtonClass = (key) => {
      const isActive = sortConfig.value.key === key
      return {
        'bg-blue-500 text-white': isActive,
        'bg-gray-100 dark:bg-gray-700 text-gray-700 dark:text-gray-300': !isActive,
        'hover:bg-blue-600': isActive,
        'hover:bg-gray-200 dark:hover:bg-gray-700': !isActive
      }
    }

    const getValueClass = (value, key) => {
      if (value === '?' || value === '#VALOR!') {
        return 'text-gray-400 dark:text-gray-500'
      }
      if (key === 'cfmVariation' || key === 'pressureVariation') {
        const numValue = parseFloat(value)
        return numValue > 0 ? 'text-green-500' : numValue < 0 ? 'text-red-500' : ''
      }
      return 'text-gray-700 dark:text-gray-300'
    }

    const isSelected = (fan) => {
      return props.selectedFans.some(f => f[0] === fan[0])
    }

    const clearFilters = () => {
      sortConfig.value = { key: '', direction: 'asc' }
      currentPage.value = 1
    }

    const handleItemsPerPageChange = () => {
      const newTotalPages = Math.ceil(filteredFans.value.length / itemsPerPage.value)
      currentPage.value = Math.min(currentPage.value, newTotalPages)
    }

    const goToPage = (page) => {
      if (page >= 1 && page <= totalPages.value) {
        currentPage.value = page
      }
    }

    const visiblePageNumbers = computed(() => {
      const delta = 2
      const range = []
      const rangeWithDots = []
      let l

      range.push(1)
      
      for (let i = currentPage.value - delta; i <= currentPage.value + delta; i++) {
        if (i < totalPages.value && i > 1) {
          range.push(i)
        }
      }
      
      range.push(totalPages.value)

      for (let i of range) {
        if (l) {
          if (i - l === 2) {
            rangeWithDots.push(l + 1)
          } else if (i - l !== 1) {
            rangeWithDots.push('...')
          }
        }
        rangeWithDots.push(i)
        l = i
      }

      return rangeWithDots
    })

    const getButtonStyle = (key, index) => {
      const isActive = sortConfig.value.key === key
      const colorClasses = [
        'from-blue-500 to-indigo-500 hover:from-blue-600 hover:to-indigo-600',
        'from-green-500 to-teal-500 hover:from-green-600 hover:to-teal-600',
        'from-purple-500 to-violet-500 hover:from-purple-600 hover:to-violet-600'
      ]
      
      if (isActive) {
        return `bg-gradient-to-r ${colorClasses[index]} text-white shadow-lg`
      }
      
      return `bg-gray-100 dark:bg-gray-700 text-gray-700 dark:text-gray-300 hover:bg-gray-200 dark:hover:bg-gray-600`
    }

    return {
      columns,
      sortableColumns,
      sortConfig,
      currentPage,
      itemsPerPage,
      totalPages,
      startIndex,
      endIndex,
      paginatedFans,
      filteredFans,
      toggleSort,
      getSortButtonClass,
      getValueClass,
      isSelected,
      clearFilters,
      visiblePageNumbers,
      goToPage,
      handleItemsPerPageChange,
      searchText,
      getButtonStyle
    }
  }
}
</script>

<style scoped>
.transform {
  transition: transform 0.2s;
}
</style>