<script setup lang="ts">
import { useSimulationStore } from '@/composables/demo/useSimulationStore'
import { ROLE_COLORS } from '@/composables/demo/types'

const store = useSimulationStore()

const statusLabels: Record<string, string> = {
  idle: '空闲',
  thinking: '思考中',
  working: '工作中',
  reviewing: '审查中',
  reworking: '返工中',
  patrolling: '巡逻中',
  done: '完成',
}
</script>

<template>
  <div class="agents-row">
    <div
      v-for="agent in store.agentList.value"
      :key="agent.id"
      class="agent-card"
      :class="`agent-card--${agent.status}`"
      :style="{ '--accent': ROLE_COLORS[agent.role] }"
    >
      <!-- 头像 + 表情 -->
      <div class="agent-avatar-area">
        <div class="agent-avatar-circle">
          <span class="agent-avatar">{{ agent.avatar }}</span>
        </div>
        <span class="agent-expression">{{ agent.expression }}</span>
      </div>

      <!-- 名字 -->
      <div class="agent-name">{{ agent.name }}</div>

      <!-- 状态 -->
      <div class="agent-status-badge" :class="`status--${agent.status}`">
        <span v-if="agent.status === 'thinking' || agent.status === 'working' || agent.status === 'reviewing' || agent.status === 'patrolling'" class="thinking-dots">
          <span class="dot" />
          <span class="dot" />
          <span class="dot" />
        </span>
        {{ statusLabels[agent.status] || agent.status }}
      </div>

      <!-- 气泡消息 -->
      <Transition name="bubble">
        <div v-if="agent.message" class="agent-bubble">
          {{ agent.message }}
        </div>
      </Transition>

      <!-- 飘分动画会在 Step 6 加 -->
    </div>
  </div>
</template>

<style scoped>
.agents-row {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.agent-card {
  background: white;
  border-radius: 16px;
  padding: 1rem 0.75rem 0.75rem;
  width: 120px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.4rem;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.04);
  transition: all 0.3s ease;
  position: relative;
  border: 2px solid transparent;
}

.agent-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.08);
}

.agent-card--working,
.agent-card--thinking,
.agent-card--reviewing,
.agent-card--patrolling {
  border-color: var(--accent);
  box-shadow: 0 2px 16px color-mix(in srgb, var(--accent) 15%, transparent);
}

.agent-card--reworking {
  border-color: #F87171;
  animation: shake 0.5s ease;
}

.agent-card--done {
  opacity: 0.7;
}

.agent-avatar-area {
  position: relative;
}

.agent-avatar-circle {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: color-mix(in srgb, var(--accent) 12%, white);
  display: flex;
  align-items: center;
  justify-content: center;
}

.agent-avatar {
  font-size: 1.6rem;
}

.agent-expression {
  position: absolute;
  bottom: -4px;
  right: -6px;
  font-size: 1rem;
  filter: drop-shadow(0 1px 2px rgba(0,0,0,0.1));
  transition: all 0.3s ease;
}

.agent-name {
  font-size: 0.85rem;
  font-weight: 700;
  color: #1e293b;
}

.agent-status-badge {
  font-size: 0.65rem;
  font-weight: 600;
  padding: 0.15rem 0.5rem;
  border-radius: 999px;
  display: flex;
  align-items: center;
  gap: 0.25rem;
}

.status--idle { background: #f1f5f9; color: #94a3b8; }
.status--thinking { background: #EDE9FE; color: #7C3AED; }
.status--working { background: #DBEAFE; color: #2563EB; }
.status--reviewing { background: #FEF3C7; color: #D97706; }
.status--reworking { background: #FEE2E2; color: #DC2626; }
.status--patrolling { background: #D1FAE5; color: #059669; }
.status--done { background: #D1FAE5; color: #059669; }

.thinking-dots {
  display: inline-flex;
  gap: 2px;
}

.dot {
  width: 4px;
  height: 4px;
  border-radius: 50%;
  background: currentColor;
  animation: dotBounce 1.2s ease-in-out infinite;
}
.dot:nth-child(2) { animation-delay: 0.2s; }
.dot:nth-child(3) { animation-delay: 0.4s; }

.agent-bubble {
  position: absolute;
  bottom: calc(100% + 8px);
  left: 50%;
  transform: translateX(-50%);
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  padding: 0.4rem 0.7rem;
  font-size: 0.7rem;
  color: #475569;
  white-space: nowrap;
  max-width: 200px;
  overflow: hidden;
  text-overflow: ellipsis;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  z-index: 10;
}

.agent-bubble::after {
  content: '';
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  border: 6px solid transparent;
  border-top-color: white;
}

/* Transitions */
.bubble-enter-active { transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1); }
.bubble-leave-active { transition: all 0.2s ease; }
.bubble-enter-from { opacity: 0; transform: translateX(-50%) translateY(6px) scale(0.9); }
.bubble-leave-to { opacity: 0; transform: translateX(-50%) translateY(-4px); }

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  10%, 30%, 50%, 70%, 90% { transform: translateX(-2px); }
  20%, 40%, 60%, 80% { transform: translateX(2px); }
}

@keyframes dotBounce {
  0%, 80%, 100% { transform: translateY(0); opacity: 0.4; }
  40% { transform: translateY(-4px); opacity: 1; }
}
</style>
