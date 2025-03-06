<template>
  <div class="mt-8 bg-white/10 backdrop-blur-md rounded-lg shadow-2xl border border-white/20">
    <div class="p-6">
      <div class="flex items-center justify-between mb-6">
        <h2 class="text-2xl font-bold text-white">Comparação de Fans</h2>
        <div v-if="winner" class="flex items-center gap-2">
          <div class="px-4 py-2 bg-gradient-to-r from-yellow-400 to-amber-500 rounded-lg shadow-lg">
            <span class="text-sm font-medium text-white">Vencedora:</span>
            <span class="ml-2 font-bold text-white">{{ winner.name }}</span>
          </div>
          <div class="px-4 py-2 bg-gradient-to-r from-blue-400 to-indigo-500 rounded-lg shadow-lg">
            <span class="text-sm font-medium text-white">Pontuação:</span>
            <span class="ml-2 font-bold text-white">{{ winner.score.toFixed(1) }}</span>
          </div>
        </div>
      </div>
      
      <!-- Cards de comparação -->
      <div v-if="fans.length === 2" class="space-y-8">
        <!-- Tabela de comparação lado a lado -->
        <div class="overflow-x-auto">
          <table class="w-full">
            <thead>
              <tr class="border-b border-white/10">
                <th class="pb-4 text-left text-gray-300 font-medium w-1/4">Especificação</th>
                <th class="pb-4 text-center text-gray-300 font-medium w-1/4">{{ fans[0][0] }}</th>
                <th class="pb-4 text-center text-gray-300 font-medium w-1/4">{{ fans[1][0] }}</th>
                <th class="pb-4 text-center text-gray-300 font-medium w-1/4">Diferença</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-white/10">
              <tr v-for="spec in specifications" :key="spec.key" 
                  class="hover:bg-white/5 transition-colors">
                <td class="py-4 text-white font-medium">
                  {{ spec.label }}
                  <span v-if="spec.winner" class="ml-2 text-yellow-400">★</span>
                </td>
                <td class="py-4 text-center">
                  <span :class="[
                    getDifferenceClass(spec.index, 0),
                    spec.winner === 0 ? 'bg-yellow-400/20 px-2 py-1 rounded' : ''
                  ]">
                    {{ fans[0][spec.index] === '?' || fans[0][spec.index] === '#VALOR!' ? 'SEM ESSA INFO' : fans[0][spec.index] }}
                  </span>
                </td>
                <td class="py-4 text-center">
                  <span :class="[
                    getDifferenceClass(spec.index, 1),
                    spec.winner === 1 ? 'bg-yellow-400/20 px-2 py-1 rounded' : ''
                  ]">
                    {{ fans[1][spec.index] === '?' || fans[1][spec.index] === '#VALOR!' ? 'SEM ESSA INFO' : fans[1][spec.index] }}
                  </span>
                </td>
                <td class="py-4 text-center">
                  <span v-if="getDifference(spec.index)" 
                        :class="[
                          getDifference(spec.index).class,
                          'px-2 py-1 rounded',
                          {
                            'bg-green-500/20': getDifference(spec.index).class.includes('text-green'),
                            'bg-red-500/20': getDifference(spec.index).class.includes('text-red')
                          }
                        ]"
                  >
                    {{ getDifference(spec.index).text }}
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Resumo das vantagens -->
        <div class="grid grid-cols-2 gap-6 mt-8">
          <div v-for="(fan, index) in fans" :key="fan[0]" 
               class="p-4 rounded-lg bg-white/5 border border-white/10">
            <h3 class="text-lg font-medium text-white mb-3">{{ fan[0] }} vence em:</h3>
            <ul class="space-y-2">
              <li v-for="spec in getWinningSpecs(index)" :key="spec.key"
                  class="flex items-center text-sm">
                <span class="text-yellow-400 mr-2">★</span>
                <span class="text-gray-300">{{ spec.label }}</span>
                <span class="ml-auto text-green-400">
                  +{{ getAdvantagePercentage(spec.index, index) }}%
                </span>
              </li>
            </ul>
          </div>
        </div>

        <!-- Botões de ação -->
        <div class="flex justify-end gap-4 mt-8">
          <button 
            v-for="fan in fans" 
            :key="fan[0]"
            @click="$emit('remove-fan', fan)"
            class="px-4 py-2 text-sm font-medium rounded-lg bg-red-500/20 text-red-400 
                   hover:bg-red-500/30 transition-colors border border-red-500/30"
          >
            Remover {{ fan[0] }}
          </button>
        </div>
      </div>

      <!-- Mensagem quando apenas uma fan está selecionada -->
      <div v-else-if="fans.length === 1" class="text-center py-8">
        <p class="text-gray-400">
          Selecione mais uma fan para comparar com {{ fans[0][0] }}
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import { computed } from 'vue'

export default {
  name: 'FanComparison',
  props: {
    fans: {
      type: Array,
      required: true,
      validator: value => value.length <= 2
    }
  },
  setup(props) {
    const specifications = [
      { key: 'maxRpm', label: 'RPM Máximo', index: 3, weight: 1 },
      { key: 'cfmDelivered', label: 'CFM Entregue', index: 7, weight: 2 },
      { key: 'pressureMMH2O', label: 'Pressão MMH2O', index: 9, weight: 2 },
      { key: 'blades', label: 'Número de Hélices', index: 2, weight: 0.5 },
      { key: 'consumption', label: 'Consumo', index: 1, weight: 1 },
      { key: 'cfmNormal', label: 'CFM/27.13 Normal', index: 4, weight: 1.5 },
      { key: 'pressure', label: 'Pressão Entregue', index: 5, weight: 1.5 },
      { key: 'testRpm', label: 'RPM FAN Teste', index: 6, weight: 1 },
      { key: 'cfmVariation', label: 'Variação CFM', index: 8, weight: 1 },
      { key: 'pressureAirCooler', label: 'Pressão Air Cooler', index: 10, weight: 1.5 },
      { key: 'pressureVariation', label: 'Variação Pressão', index: 11, weight: 1 },
      { key: 'average', label: 'Média', index: 12, weight: 2 }
    ]

    const getWinnerForSpec = (spec) => {
      if (!props.fans[0] || !props.fans[1]) return null
      
      const value1 = parseFloat(props.fans[0][spec.index])
      const value2 = parseFloat(props.fans[1][spec.index])
      
      if (isNaN(value1) || isNaN(value2)) return null
      
      // Para consumo, menor é melhor
      if (spec.key === 'consumption') {
        return value1 < value2 ? 0 : value2 < value1 ? 1 : null
      }
      
      return value1 > value2 ? 0 : value2 > value1 ? 1 : null
    }

    // Atualiza as especificações com o vencedor de cada comparação
    const specsWithWinners = computed(() => {
      return specifications.map(spec => ({
        ...spec,
        winner: getWinnerForSpec(spec)
      }))
    })

    const winner = computed(() => {
      if (props.fans.length !== 2) return null

      let scores = [0, 0]
      let validComparisons = 0

      specsWithWinners.value.forEach(spec => {
        if (spec.winner !== null) {
          scores[spec.winner] += spec.weight
          validComparisons++
        }
      })

      if (validComparisons === 0) return null

      const winnerIndex = scores[0] > scores[1] ? 0 : 1
      return {
        name: props.fans[winnerIndex][0],
        score: scores[winnerIndex]
      }
    })

    const getWinningSpecs = (fanIndex) => {
      return specsWithWinners.value.filter(spec => spec.winner === fanIndex)
    }

    const getAdvantagePercentage = (index, fanIndex) => {
      const value1 = parseFloat(props.fans[fanIndex][index])
      const value2 = parseFloat(props.fans[1 - fanIndex][index])
      
      if (isNaN(value1) || isNaN(value2) || value2 === 0) return '?'
      
      return Math.abs(((value1 - value2) / value2) * 100).toFixed(1)
    }

    const getDifferenceClass = (index, fanIndex) => {
      if (!props.fans[0] || !props.fans[1]) return getValueClass(props.fans[fanIndex][index])
      
      const value1 = parseFloat(props.fans[0][index])
      const value2 = parseFloat(props.fans[1][index])
      
      if (isNaN(value1) || isNaN(value2)) return getValueClass(props.fans[fanIndex][index])
      
      const isBetter = fanIndex === 0 ? value1 > value2 : value2 > value1
      
      return isBetter ? 'text-green-400 font-medium' : 'text-red-400 font-medium'
    }

    const getValueClass = (value) => {
      if (value === '?' || value === '#VALOR!') {
        return 'text-gray-500 italic'
      }
      return 'text-white font-medium'
    }

    const getDifference = (index) => {
      if (!props.fans[0] || !props.fans[1]) return null
      
      const value1 = parseFloat(props.fans[0][index])
      const value2 = parseFloat(props.fans[1][index])
      
      if (isNaN(value1) || isNaN(value2)) return null

      const diff = value1 - value2
      const percentDiff = ((diff / value2) * 100).toFixed(1)
      
      if (diff === 0) return null

      const isPositive = diff > 0
      return {
        text: `${isPositive ? '+' : ''}${percentDiff}%`,
        class: isPositive ? 'text-green-400 font-medium' : 'text-red-400 font-medium'
      }
    }

    return {
      specsWithWinners,
      winner,
      getWinningSpecs,
      getAdvantagePercentage,
      getDifferenceClass,
      getDifference,
      specifications: specsWithWinners
    }
  }
}
</script>