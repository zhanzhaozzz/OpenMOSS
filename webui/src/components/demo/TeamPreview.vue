<script setup lang="ts">
import { ref } from 'vue'
import type { AgentDef } from '@/composables/demo/types'
import { ROLE_COLORS } from '@/composables/demo/types'

const props = defineProps<{
  agents: AgentDef[]
}>()

const emit = defineEmits<{
  launch: [agents: AgentDef[]]
}>()

// 可编辑的 Agent 名字副本
const editableNames = ref<Record<string, string>>({})

function initNames() {
  for (const a of props.agents) {
    editableNames.value[a.id] = a.name
  }
}
initNames()

// 角色中文名
const roleLabels: Record<string, string> = {
  planner: '规划者',
  executor: '执行者',
  reviewer: '审查者',
  patrol: '巡查员',
}

function handleLaunch() {
  const updated = props.agents.map((a) => ({
    ...a,
    name: editableNames.value[a.id] || a.name,
  }))
  emit('launch', updated)
}
</script>

<template>
  <div class="team-preview">
    <div class="agent-grid">
      <div
        v-for="agent in props.agents"
        :key="agent.id"
        class="agent-card"
        :style="{ '--accent': ROLE_COLORS[agent.role] }"
      >
        <div class="agent-avatar-wrap">
          <span class="agent-avatar">{{ agent.avatar }}</span>
        </div>
        <input
          v-model="editableNames[agent.id]"
          class="agent-name-input"
          maxlength="8"
          :placeholder="agent.name"
        />
        <span class="agent-role">{{ roleLabels[agent.role] || agent.role }}</span>
      </div>
    </div>

    <button class="launch-btn" @click="handleLaunch">
      <span class="launch-icon">🚀</span>
      启动 AI 团队
    </button>
  </div>
</template>

<style scoped>
.team-preview {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2rem;
}

.agent-grid {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 1.25rem;
}

.agent-card {
  background: white;
  border-radius: 20px;
  padding: 1.5rem 1.25rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
  width: 130px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.05);
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  border: 2px solid transparent;
}

.agent-card:hover {
  transform: translateY(-4px);
  border-color: var(--accent);
  box-shadow: 0 8px 24px color-mix(in srgb, var(--accent) 20%, transparent);
}

.agent-avatar-wrap {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: color-mix(in srgb, var(--accent) 15%, white);
  display: flex;
  align-items: center;
  justify-content: center;
  animation: breathe 3s ease-in-out infinite;
}

.agent-avatar {
  font-size: 2rem;
}

.agent-name-input {
  width: 100%;
  text-align: center;
  border: none;
  border-bottom: 2px dashed #e2e8f0;
  background: transparent;
  font-family: 'Nunito', sans-serif;
  font-size: 1rem;
  font-weight: 700;
  color: #1e293b;
  padding: 0.25rem 0;
  outline: none;
  transition: border-color 0.2s;
}

.agent-name-input:focus {
  border-bottom-color: var(--accent);
}

.agent-role {
  font-size: 0.75rem;
  color: white;
  background: var(--accent);
  padding: 0.15rem 0.6rem;
  border-radius: 999px;
  font-weight: 600;
}

.launch-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 1rem 2.5rem;
  font-size: 1.2rem;
  font-weight: 800;
  font-family: 'Nunito', sans-serif;
  color: white;
  background: linear-gradient(135deg, #818CF8, #6366F1);
  border: none;
  border-radius: 16px;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  box-shadow: 0 4px 16px rgba(99, 102, 241, 0.3);
}

.launch-btn:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 8px 24px rgba(99, 102, 241, 0.4);
}

.launch-btn:active {
  transform: translateY(0) scale(0.98);
}

.launch-icon {
  font-size: 1.4rem;
}

@keyframes breathe {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}
</style>
