<template>
  <div class="h-full flex flex-col bg-slate-800 text-slate-200 rounded-t-xl">
    <div class="flex items-center justify-between p-4 border-b border-slate-700">
      <div class="flex items-center gap-4">
        <h2 class="text-xl font-bold text-cyan-400">候选清单</h2>
        <div class="flex items-center gap-2 text-sm">
          <span class="px-2 py-1 rounded bg-slate-700 text-slate-300">总数 {{ store.candidateStats.total }}</span>
          <span class="px-2 py-1 rounded bg-green-900/50 text-green-400">推荐 {{ store.candidateStats.recommended }}</span>
          <span class="px-2 py-1 rounded bg-orange-900/50 text-orange-400">待评估 {{ store.candidateStats.evaluating }}</span>
          <span class="px-2 py-1 rounded bg-red-900/50 text-red-400">不推荐 {{ store.candidateStats.rejected }}</span>
        </div>
      </div>
      <div class="flex items-center gap-2">
        <button
          @click="showCriteria = !showCriteria"
          class="px-3 py-1.5 rounded text-sm font-medium bg-slate-700 hover:bg-slate-600 text-slate-200 transition-all border border-slate-600"
        >
          {{ showCriteria ? '隐藏筛选条件' : '显示筛选条件' }}
        </button>
        <button
          @click="store.rescreenAll()"
          class="px-3 py-1.5 rounded text-sm font-medium bg-blue-600 hover:bg-blue-500 text-white transition-all border border-blue-500"
        >
          重新评估
        </button>
        <button
          @click="confirmClear"
          class="px-3 py-1.5 rounded text-sm font-medium bg-red-600 hover:bg-red-500 text-white transition-all border border-red-500"
        >
          清空
        </button>
        <button
          @click="store.toggleCandidatePanel()"
          class="px-3 py-1.5 rounded text-sm font-medium bg-slate-700 hover:bg-slate-600 text-slate-200 transition-all border border-slate-600"
        >
          关闭
        </button>
      </div>
    </div>

    <div v-if="showCriteria" class="p-4 border-b border-slate-700 bg-slate-900/50">
      <div class="grid grid-cols-2 lg:grid-cols-5 gap-4">
        <div>
          <label class="block text-xs text-slate-400 mb-1">最大分子量 (maxMw)</label>
          <input
            v-model.number="localCriteria.maxMw"
            type="number"
            class="w-full bg-slate-900 border border-slate-600 rounded px-3 py-1.5 text-sm focus:outline-none focus:border-cyan-500"
          />
        </div>
        <div>
          <label class="block text-xs text-slate-400 mb-1">最大 LogP (maxLogP)</label>
          <input
            v-model.number="localCriteria.maxLogP"
            type="number"
            step="0.1"
            class="w-full bg-slate-900 border border-slate-600 rounded px-3 py-1.5 text-sm focus:outline-none focus:border-cyan-500"
          />
        </div>
        <div>
          <label class="block text-xs text-slate-400 mb-1">最小 LogS (minLogS)</label>
          <input
            v-model.number="localCriteria.minLogS"
            type="number"
            step="0.1"
            class="w-full bg-slate-900 border border-slate-600 rounded px-3 py-1.5 text-sm focus:outline-none focus:border-cyan-500"
          />
        </div>
        <div>
          <label class="block text-xs text-slate-400 mb-1">最小生物利用度% (minBioavailability)</label>
          <input
            v-model.number="localCriteria.minBioavailability"
            type="number"
            class="w-full bg-slate-900 border border-slate-600 rounded px-3 py-1.5 text-sm focus:outline-none focus:border-cyan-500"
          />
        </div>
        <div class="flex items-end">
          <label class="flex items-center gap-2 cursor-pointer">
            <input
              v-model="localCriteria.requireRuleOfFive"
              type="checkbox"
              class="w-4 h-4 rounded bg-slate-900 border-slate-600 text-cyan-500 focus:ring-cyan-500"
            />
            <span class="text-sm text-slate-300">要求通过五规则</span>
          </label>
        </div>
      </div>
      <div class="mt-4">
        <button
          @click="applyCriteria"
          class="px-4 py-1.5 rounded text-sm font-medium bg-cyan-600 hover:bg-cyan-500 text-white transition-all border border-cyan-500"
        >
          应用筛选并重新评估
        </button>
      </div>
    </div>

    <div class="flex border-b border-slate-700 bg-slate-900/30">
      <button
        v-for="tab in tabs"
        :key="tab.key"
        @click="activeTab = tab.key"
        :class="[
          'px-4 py-3 text-sm font-medium transition-all flex items-center gap-2 border-b-2',
          activeTab === tab.key
            ? 'border-cyan-500 text-cyan-400 bg-slate-800'
            : 'border-transparent text-slate-400 hover:text-slate-200 hover:bg-slate-800/50'
        ]"
      >
        {{ tab.label }}
        <span
          :class="[
            'px-2 py-0.5 rounded-full text-xs',
            tab.key === 'all' ? 'bg-slate-700 text-slate-300' :
            tab.key === '推荐' ? 'bg-green-900/50 text-green-400' :
            tab.key === '待评估' ? 'bg-orange-900/50 text-orange-400' :
            'bg-red-900/50 text-red-400'
          ]"
        >
          {{ tab.count }}
        </span>
      </button>
    </div>

    <div class="flex-1 overflow-y-auto">
      <div v-if="filteredCandidates.length === 0" class="flex flex-col items-center justify-center h-full p-12">
        <div class="text-6xl mb-4 opacity-30">📋</div>
        <div class="text-xl text-slate-500 font-medium">暂无候选分子</div>
        <div class="text-sm text-slate-600 mt-2">请从左侧列表添加</div>
      </div>
      <div v-else class="divide-y divide-slate-700">
        <div
          v-for="item in filteredCandidates"
          :key="item.molecule.id"
          class="p-4 hover:bg-slate-900/50 transition-all"
        >
          <div class="flex items-start gap-4">
            <button
              @click="selectAndView(item.molecule)"
              class="mt-1 px-3 py-1 rounded text-xs font-medium bg-slate-700 hover:bg-cyan-600 text-slate-200 hover:text-white transition-all border border-slate-600 hover:border-cyan-500"
            >
              查看
            </button>
            <div class="flex-1 min-w-0">
              <div class="flex items-center gap-3 mb-2">
                <span class="text-base font-bold text-slate-200">{{ item.molecule.name }}</span>
                <span class="text-xs px-2 py-0.5 rounded bg-slate-700 text-slate-400">{{ item.molecule.category }}</span>
                <span class="text-xs text-slate-500 font-mono">{{ item.molecule.formula }}</span>
              </div>
              <div class="flex items-center gap-6 text-sm mb-3">
                <span class="text-slate-400">MW: <span class="text-slate-200 font-medium">{{ item.molecule.mw }}</span></span>
                <span class="text-slate-400">LogP: <span :class="item.admet.logP > 3 ? 'text-red-400 font-medium' : 'text-green-400 font-medium'">{{ item.admet.logP }}</span></span>
                <span class="text-slate-400">生物利用度: <span :class="item.admet.bioavailability > 50 ? 'text-green-400 font-medium' : 'text-orange-400 font-medium'">{{ item.admet.bioavailability }}%</span></span>
              </div>
              <div class="grid grid-cols-1 lg:grid-cols-4 gap-4 items-start">
                <div class="relative" @click.stop>
                  <button
                    @click="toggleVerdictDropdown(item.molecule.id)"
                    :class="[
                      'w-full px-3 py-1.5 rounded text-sm font-medium text-left flex items-center justify-between border transition-all',
                      item.verdict === '推荐' ? 'bg-green-900/30 text-green-400 border-green-700/50 hover:bg-green-900/50' :
                      item.verdict === '待评估' ? 'bg-orange-900/30 text-orange-400 border-orange-700/50 hover:bg-orange-900/50' :
                      'bg-red-900/30 text-red-400 border-red-700/50 hover:bg-red-900/50'
                    ]"
                  >
                    <span>{{ item.verdict }}</span>
                    <span class="text-xs">▼</span>
                  </button>
                  <div
                    v-if="openVerdictDropdown === item.molecule.id"
                    class="absolute z-20 mt-1 w-full bg-slate-900 border border-slate-700 rounded-lg shadow-xl overflow-hidden"
                  >
                    <button
                      v-for="v in verdicts"
                      :key="v"
                      @click="changeVerdict(item.molecule.id, v)"
                      :class="[
                        'w-full px-3 py-2 text-left text-sm transition-all hover:bg-slate-800',
                        item.verdict === v ? 'bg-slate-800 text-cyan-400' : 'text-slate-300'
                      ]"
                    >
                      {{ v }}
                    </button>
                  </div>
                </div>
                <div>
                  <label class="block text-xs text-slate-500 mb-1">优先级</label>
                  <div class="flex items-center gap-1" @click.stop>
                    <button
                      v-for="star in 5"
                      :key="star"
                      @click="store.updatePriority(item.molecule.id, star)"
                      class="text-xl transition-transform hover:scale-110"
                    >
                      <span :class="star <= item.priority ? 'text-yellow-400' : 'text-slate-600'">★</span>
                    </button>
                  </div>
                </div>
                <div class="lg:col-span-2" @click.stop>
                  <label class="block text-xs text-slate-500 mb-1">备注</label>
                  <input
                    :value="item.notes"
                    @input="store.updateNotes(item.molecule.id, ($event.target as HTMLInputElement).value)"
                    type="text"
                    placeholder="添加备注..."
                    class="w-full bg-slate-900 border border-slate-600 rounded px-3 py-1.5 text-sm focus:outline-none focus:border-cyan-500"
                  />
                </div>
              </div>
            </div>
            <button
              @click="store.removeFromCandidates(item.molecule.id)"
              class="mt-1 p-2 rounded text-red-400 hover:bg-red-900/30 hover:text-red-300 transition-all border border-transparent hover:border-red-700/50"
              title="从候选移除"
            >
              ✕
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, reactive } from 'vue'
import { useMoleculeStore } from '../store/molecule'
import type { MoleculeData, ScreeningVerdict } from '../types'

const store = useMoleculeStore()
const showCriteria = ref(false)
const activeTab = ref<string>('all')
const openVerdictDropdown = ref<number | null>(null)
const verdicts: ScreeningVerdict[] = ['推荐', '待评估', '不推荐', '待定']

const localCriteria = reactive({
  maxMw: store.screeningCriteria.maxMw,
  maxLogP: store.screeningCriteria.maxLogP,
  minLogS: store.screeningCriteria.minLogS,
  minBioavailability: store.screeningCriteria.minBioavailability,
  requireRuleOfFive: store.screeningCriteria.requireRuleOfFive,
  allowToxicity: [...store.screeningCriteria.allowToxicity],
  maxViolations: store.screeningCriteria.maxViolations
})

watch(() => store.screeningCriteria, (newVal) => {
  localCriteria.maxMw = newVal.maxMw
  localCriteria.maxLogP = newVal.maxLogP
  localCriteria.minLogS = newVal.minLogS
  localCriteria.minBioavailability = newVal.minBioavailability
  localCriteria.requireRuleOfFive = newVal.requireRuleOfFive
  localCriteria.allowToxicity = [...newVal.allowToxicity]
  localCriteria.maxViolations = newVal.maxViolations
}, { deep: true })

const tabs = computed(() => [
  { key: 'all', label: '全部', count: store.candidateStats.total },
  { key: '推荐', label: '推荐', count: store.candidateStats.recommended },
  { key: '待评估', label: '待评估', count: store.candidateStats.evaluating },
  { key: '不推荐', label: '不推荐', count: store.candidateStats.rejected }
])

const filteredCandidates = computed(() => {
  if (activeTab.value === 'all') return store.candidates
  return store.candidates.filter(c => c.verdict === activeTab.value)
})

function applyCriteria() {
  store.screeningCriteria.maxMw = localCriteria.maxMw
  store.screeningCriteria.maxLogP = localCriteria.maxLogP
  store.screeningCriteria.minLogS = localCriteria.minLogS
  store.screeningCriteria.minBioavailability = localCriteria.minBioavailability
  store.screeningCriteria.requireRuleOfFive = localCriteria.requireRuleOfFive
  store.screeningCriteria.allowToxicity = [...localCriteria.allowToxicity]
  store.screeningCriteria.maxViolations = localCriteria.maxViolations
  store.rescreenAll()
}

function selectAndView(mol: MoleculeData) {
  store.selectMolecule(mol)
  store.toggleCandidatePanel()
}

function toggleVerdictDropdown(molId: number) {
  openVerdictDropdown.value = openVerdictDropdown.value === molId ? null : molId
}

function changeVerdict(molId: number, verdict: ScreeningVerdict) {
  store.updateVerdict(molId, verdict)
  openVerdictDropdown.value = null
}

function confirmClear() {
  if (store.candidates.length === 0) return
  if (confirm('确定要清空所有候选分子吗？')) {
    store.clearCandidates()
  }
}

document.addEventListener('click', (e) => {
  const target = e.target as HTMLElement
  if (!target.closest('.relative')) {
    openVerdictDropdown.value = null
  }
})
</script>
