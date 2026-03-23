<script setup lang="ts">
import type { ScenarioData } from '@/composables/demo/types'

import contentCompany from '@/composables/demo/scenarios/content-company.json'
import ecommerce from '@/composables/demo/scenarios/ecommerce.json'
import chaos from '@/composables/demo/scenarios/chaos.json'

const scenarios = [contentCompany, ecommerce, chaos] as unknown as ScenarioData[]

const emit = defineEmits<{
  select: [scenario: ScenarioData]
}>()
</script>

<template>
  <div class="scenario-grid">
    <button
      v-for="s in scenarios"
      :key="s.id"
      class="scenario-card"
      :class="`scenario-card--${s.id}`"
      @click="emit('select', s)"
    >
      <span class="scenario-icon">{{ s.icon }}</span>
      <h3 class="scenario-name">{{ s.name }}</h3>
      <p class="scenario-desc">{{ s.description }}</p>
      <span class="scenario-meta">
        {{ s.agents.length }} 个 AI 员工 · {{ s.duration }}s
      </span>
    </button>
  </div>
</template>

<style scoped>
.scenario-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 1.5rem;
  max-width: 900px;
  margin: 0 auto;
}

.scenario-card {
  background: white;
  border: 2px solid transparent;
  border-radius: 20px;
  padding: 2rem 1.5rem;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
  text-align: center;
}

.scenario-card:hover {
  transform: translateY(-6px) scale(1.03);
  box-shadow: 0 12px 32px rgba(0, 0, 0, 0.1);
}

.scenario-card:active {
  transform: translateY(-2px) scale(0.99);
}

/* 每个场景的个性色 */
.scenario-card--content-company:hover {
  border-color: #818CF8;
  box-shadow: 0 12px 32px rgba(129, 140, 248, 0.2);
}
.scenario-card--ecommerce:hover {
  border-color: #FB923C;
  box-shadow: 0 12px 32px rgba(251, 146, 60, 0.2);
}
.scenario-card--chaos:hover {
  border-color: #F87171;
  box-shadow: 0 12px 32px rgba(248, 113, 113, 0.2);
}

.scenario-icon {
  font-size: 3rem;
  display: block;
  margin-bottom: 0.25rem;
}

.scenario-name {
  font-size: 1.3rem;
  font-weight: 800;
  color: #1e293b;
  margin: 0;
}

.scenario-desc {
  font-size: 0.9rem;
  color: #64748b;
  line-height: 1.5;
  margin: 0;
}

.scenario-meta {
  font-size: 0.8rem;
  color: #94a3b8;
  margin-top: 0.5rem;
  padding: 0.25rem 0.75rem;
  background: #f8fafc;
  border-radius: 999px;
}
</style>
