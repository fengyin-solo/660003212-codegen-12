<template>
  <div class="min-h-screen bg-slate-900 text-slate-200 relative">
    <header class="border-b border-slate-700 px-6 py-4 flex items-center justify-between">
      <div>
        <h1 class="text-2xl font-bold text-cyan-400">药物分子 3D 结构与 ADMET 性质预测</h1>
        <p class="text-sm text-slate-500 mt-1">SMILES解析 · Three.js球棍模型 · ADMET预测 · Tanimoto相似搜索</p>
      </div>
      <button
        @click="store.toggleCandidatePanel()"
        class="relative px-4 py-2 rounded-lg text-sm font-medium bg-slate-800 hover:bg-slate-700 text-slate-200 transition-all border border-slate-700 hover:border-cyan-500 flex items-center gap-2"
      >
        <span>📋 候选清单</span>
        <span
          v-if="store.candidateStats.total > 0"
          class="absolute -top-2 -right-2 bg-cyan-500 text-white text-xs font-bold w-6 h-6 rounded-full flex items-center justify-center border-2 border-slate-900"
        >
          {{ store.candidateStats.total }}
        </span>
      </button>
    </header>
    <div class="flex flex-col lg:flex-row gap-4 p-4">
      <div class="lg:w-1/4 space-y-4">
        <MoleculeSearch />
      </div>
      <div class="lg:w-1/2 space-y-4">
        <MoleculeViewer />
      </div>
      <div class="lg:w-1/4 space-y-4">
        <AdmetPanel />
      </div>
    </div>

    <Teleport to="body">
      <Transition name="fade">
        <div
          v-if="store.showCandidatePanel"
          class="fixed inset-0 bg-black/60 z-40"
          @click="store.toggleCandidatePanel()"
        ></div>
      </Transition>
      <Transition name="slide-up">
        <div
          v-if="store.showCandidatePanel"
          class="fixed inset-x-0 bottom-0 z-50 shadow-2xl border-t border-slate-700 bg-slate-800 rounded-t-xl"
          style="height: 80vh; max-height: 80vh;"
        >
          <CandidateList />
        </div>
      </Transition>
    </Teleport>
  </div>
</template>

<script setup lang="ts">
import { onMounted } from 'vue'
import { useMoleculeStore } from './store/molecule'
import MoleculeViewer from './components/MoleculeViewer.vue'
import AdmetPanel from './components/AdmetPanel.vue'
import MoleculeSearch from './components/MoleculeSearch.vue'
import CandidateList from './components/CandidateList.vue'

const store = useMoleculeStore()
onMounted(() => store.loadMolecules())
</script>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-up-enter-active,
.slide-up-leave-active {
  transition: transform 0.3s ease;
}
.slide-up-enter-from,
.slide-up-leave-to {
  transform: translateY(100%);
}
</style>
