<script setup lang="ts">
import { ref, watch } from 'vue'
import type { ScenarioData, AgentDef } from '@/composables/demo/types'
import { useScenarioPlayer, useSimulationStore } from '@/composables/demo'
import ScenarioSelector from '@/components/demo/ScenarioSelector.vue'
import TeamPreview from '@/components/demo/TeamPreview.vue'
import KanbanBoard from '@/components/demo/KanbanBoard.vue'
import AgentCards from '@/components/demo/AgentCards.vue'
import LiveLog from '@/components/demo/LiveLog.vue'
import FloatingScore from '@/components/demo/FloatingScore.vue'
import ResultPage from '@/components/demo/ResultPage.vue'

type Phase = 'select' | 'preview' | 'simulation' | 'result'

const phase = ref<Phase>('select')
const selectedScenario = ref<ScenarioData | null>(null)
const player = useScenarioPlayer()
const store = useSimulationStore()

// 自动切到结果页
watch(() => store.state.phase, (p) => {
  if (p === 'result') phase.value = 'result'
})

function onSelectScenario(scenario: ScenarioData) {
  selectedScenario.value = scenario
  phase.value = 'preview'
}

function onLaunch(agents: AgentDef[]) {
  if (!selectedScenario.value) return
  // 用用户可能修改过的名字更新场景
  const updated: ScenarioData = {
    ...selectedScenario.value,
    agents,
  }
  player.load(updated)
  phase.value = 'simulation'
  player.play()
}

function goBack() {
  if (phase.value === 'preview') {
    phase.value = 'select'
    selectedScenario.value = null
  }
}

function restart() {
  player.stop()
  store.reset()
  phase.value = 'select'
  selectedScenario.value = null
}

function togglePlay() {
  if (player.playing.value) {
    player.pause()
  } else {
    player.play()
  }
}

function cycleSpeed() {
  const speeds = [1, 2, 3]
  const idx = speeds.indexOf(player.speed.value)
  const next = speeds[(idx + 1) % speeds.length] ?? 1
  player.setSpeed(next)
}
</script>

<template>
  <div class="demo-page">
    <header class="demo-header">
      <h1 class="demo-title">
        <span class="demo-logo">🏢</span>
        OpenMOSS AI 公司体验
      </h1>
      <p class="demo-subtitle">2 分钟看懂 AI 团队如何自动工作、分工、复盘、优化</p>
    </header>

    <main class="demo-main">
      <Transition name="fade-slide" mode="out-in">
        <!-- Phase 1: 选择场景 -->
        <div v-if="phase === 'select'" key="select" class="demo-phase">
          <h2 class="phase-title">选择你的 AI 公司类型</h2>
          <p class="phase-desc">选一个场景，看看 AI 团队怎么干活</p>
          <ScenarioSelector @select="onSelectScenario" />
        </div>

        <!-- Phase 2: 团队预览 -->
        <div v-else-if="phase === 'preview' && selectedScenario" key="preview" class="demo-phase">
          <button class="back-btn" @click="goBack">← 重新选择</button>
          <h2 class="phase-title">{{ selectedScenario.icon }} {{ selectedScenario.name }}</h2>
          <p class="phase-desc">你可以修改 AI 员工的名字，然后启动团队</p>
          <TeamPreview :agents="selectedScenario.agents" @launch="onLaunch" />
        </div>

        <!-- Phase 3: 工作流模拟 -->
        <div v-else-if="phase === 'simulation'" key="simulation" class="demo-phase">
          <div class="sim-header">
            <h2 class="phase-title">{{ selectedScenario?.icon }} AI 团队工作中...</h2>
            <div class="sim-controls">
              <button class="ctrl-btn" @click="togglePlay">
                {{ player.playing.value ? '⏸️' : '▶️' }}
              </button>
              <button class="ctrl-btn" @click="cycleSpeed">{{ player.speed.value }}x</button>
              <button class="ctrl-btn" @click="restart">🔄</button>
            </div>
          </div>

          <!-- 进度条 -->
          <div class="progress-bar">
            <div class="progress-fill" :style="{ width: `${(player.progress.value * 100).toFixed(1)}%` }" />
          </div>

          <!-- Agent 卡片排 -->
          <AgentCards />

          <!-- 看板 -->
          <KanbanBoard />

          <!-- 实时日志 -->
          <LiveLog />

          <!-- 飘分动画 -->
          <FloatingScore />
        </div>

        <!-- Phase 4: 结果页 -->
        <div v-else-if="phase === 'result'" key="result" class="demo-phase">
          <ResultPage @restart="restart" />
        </div>
      </Transition>
    </main>
  </div>
</template>

<style scoped>
.demo-page {
  min-height: 100vh;
  background: linear-gradient(180deg, #FFF8F0 0%, #F0F4FF 100%);
  font-family: 'Nunito', sans-serif;
  padding: 2rem 1.5rem;
}

.demo-header {
  text-align: center;
  margin-bottom: 2.5rem;
}

.demo-title {
  font-size: 2rem;
  font-weight: 800;
  color: #1e293b;
  margin: 0 0 0.5rem;
}

.demo-logo {
  font-size: 2.2rem;
  margin-right: 0.5rem;
}

.demo-subtitle {
  font-size: 1.05rem;
  color: #64748b;
  font-weight: 400;
  margin: 0;
}

.demo-main {
  max-width: 1000px;
  margin: 0 auto;
}

.demo-phase {
  min-height: 200px;
}

.phase-title {
  font-size: 1.5rem;
  font-weight: 700;
  color: #334155;
  text-align: center;
  margin: 0 0 0.5rem;
}

.phase-desc {
  text-align: center;
  color: #94a3b8;
  margin: 0 0 2rem;
}

.back-btn {
  display: inline-flex;
  align-items: center;
  background: none;
  border: none;
  color: #818CF8;
  font-family: 'Nunito', sans-serif;
  font-size: 0.95rem;
  font-weight: 600;
  cursor: pointer;
  padding: 0.5rem 0;
  margin-bottom: 1rem;
  transition: color 0.2s;
}
.back-btn:hover {
  color: #6366F1;
}

.sim-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 1.5rem;
}

.restart-btn {
  background: none;
  border: 2px solid #e2e8f0;
  border-radius: 12px;
  padding: 0.5rem 1rem;
  font-family: 'Nunito', sans-serif;
  font-weight: 600;
  color: #64748b;
  cursor: pointer;
  transition: all 0.2s;
}
.restart-btn:hover {
  border-color: #818CF8;
  color: #818CF8;
}

.sim-controls {
  display: flex;
  gap: 0.5rem;
}

.ctrl-btn {
  background: white;
  border: 2px solid #e2e8f0;
  border-radius: 10px;
  padding: 0.4rem 0.75rem;
  font-family: 'Nunito', sans-serif;
  font-weight: 700;
  font-size: 0.85rem;
  color: #475569;
  cursor: pointer;
  transition: all 0.2s;
}
.ctrl-btn:hover {
  border-color: #818CF8;
  color: #818CF8;
}

.progress-bar {
  width: 100%;
  height: 4px;
  background: #e2e8f0;
  border-radius: 2px;
  margin-bottom: 1.5rem;
  overflow: hidden;
}
.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #818CF8, #6366F1);
  border-radius: 2px;
  transition: width 0.2s linear;
}

.sim-placeholder {
  text-align: center;
  padding: 6rem 2rem;
  background: white;
  border-radius: 20px;
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.05);
  color: #94a3b8;
  font-size: 1.1rem;
}

/* Transition */
.fade-slide-enter-active,
.fade-slide-leave-active {
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
}
.fade-slide-enter-from {
  opacity: 0;
  transform: translateY(16px);
}
.fade-slide-leave-to {
  opacity: 0;
  transform: translateY(-16px);
}
</style>
