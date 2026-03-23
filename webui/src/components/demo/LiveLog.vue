<script setup lang="ts">
import { ref, watch, nextTick } from 'vue'
import { useSimulationStore } from '@/composables/demo/useSimulationStore'
import type { TimelineEntry } from '@/composables/demo/types'

const store = useSimulationStore()
const logRef = ref<HTMLElement | null>(null)

// 逐字打出效果：追踪正在打字的条目
const typingEntryId = ref<string | null>(null)
const typingText = ref('')
let typingTimer: ReturnType<typeof setTimeout> | null = null

function getLogIcon(entry: TimelineEntry): string {
  switch (entry.event.type) {
    case 'review':
      return entry.event.result === 'approved' ? '✅' : '❌'
    case 'reflection':
      return '💭'
    case 'score_change':
      return (entry.event.delta ?? 0) > 0 ? '⭐' : '💔'
    case 'task_created':
      return '📋'
    default:
      return entry.agentAvatar || '💬'
  }
}

function getLogMessage(entry: TimelineEntry): string {
  const e = entry.event
  switch (e.type) {
    case 'log':
      return e.content || e.message || ''
    case 'agent_status':
      return e.message || `${entry.agentName} 状态变更为 ${e.status}`
    case 'task_created': {
      const taskName = typeof e.task === 'object' ? e.task.name : e.task
      return e.message || `创建任务：${taskName}`
    }
    case 'review':
      return `${e.result === 'approved' ? '通过' : '驳回'}（${e.score}/5）${e.comment || ''}`
    case 'reflection':
      return `反思：${e.content}`
    case 'score_change':
      return `${(e.delta ?? 0) > 0 ? '+' : ''}${e.delta} ${e.reason || ''}`
    default:
      return e.message || e.content || ''
  }
}

function getLogClass(entry: TimelineEntry): string {
  const e = entry.event
  if (e.type === 'review' && e.result === 'rejected') return 'log-entry--rejected'
  if (e.type === 'review' && e.result === 'approved') return 'log-entry--approved'
  if (e.type === 'reflection') return 'log-entry--reflection'
  if (e.type === 'score_change' && (e.delta ?? 0) < 0) return 'log-entry--negative'
  if (e.type === 'score_change' && (e.delta ?? 0) > 0) return 'log-entry--positive'
  return ''
}

// 监听新日志，触发打字效果
watch(
  () => store.state.timeline.length,
  () => {
    const latest = store.state.timeline[0]
    if (!latest) return

    // 只对 log / reflection / review 做打字效果
    const typableTypes = ['log', 'reflection', 'review']
    if (typableTypes.includes(latest.event.type)) {
      startTyping(latest)
    }

    nextTick(() => {
      if (logRef.value) {
        logRef.value.scrollTop = 0
      }
    })
  }
)

function startTyping(entry: TimelineEntry) {
  if (typingTimer) clearTimeout(typingTimer)

  const fullText = getLogMessage(entry)
  typingEntryId.value = entry.id
  typingText.value = ''

  let i = 0
  function typeChar() {
    if (i < fullText.length) {
      typingText.value += fullText[i]
      i++
      typingTimer = setTimeout(typeChar, 30 + Math.random() * 20)
    } else {
      typingEntryId.value = null
    }
  }
  typeChar()
}

function getDisplayText(entry: TimelineEntry): string {
  if (typingEntryId.value === entry.id) {
    return typingText.value
  }
  return getLogMessage(entry)
}
</script>

<template>
  <div class="live-log" ref="logRef">
    <div class="log-header">
      <span class="log-title">💬 实时日志</span>
      <span class="log-count">{{ store.state.timeline.length }} 条</span>
    </div>

    <TransitionGroup name="log-entry" tag="div" class="log-list">
      <div
        v-for="entry in store.state.timeline.slice(0, 30)"
        :key="entry.id"
        class="log-entry"
        :class="getLogClass(entry)"
      >
        <span class="log-icon">{{ getLogIcon(entry) }}</span>
        <span v-if="entry.agentName" class="log-agent">{{ entry.agentName }}：</span>
        <span class="log-text">
          {{ getDisplayText(entry) }}
          <span v-if="typingEntryId === entry.id" class="typing-cursor">▌</span>
        </span>
      </div>
    </TransitionGroup>

    <div v-if="store.state.timeline.length === 0" class="log-empty">
      等待 AI 团队开始工作...
    </div>
  </div>
</template>

<style scoped>
.live-log {
  background: white;
  border-radius: 16px;
  padding: 1rem;
  max-height: 280px;
  overflow-y: auto;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.04);
}

.log-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
  padding-bottom: 0.5rem;
  border-bottom: 1px solid #f1f5f9;
}

.log-title {
  font-size: 0.9rem;
  font-weight: 700;
  color: #334155;
}

.log-count {
  font-size: 0.7rem;
  color: #94a3b8;
  background: #f8fafc;
  padding: 0.15rem 0.5rem;
  border-radius: 999px;
}

.log-list {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
  position: relative;
}

.log-entry {
  display: flex;
  align-items: flex-start;
  gap: 0.4rem;
  font-size: 0.8rem;
  line-height: 1.5;
  padding: 0.35rem 0.5rem;
  border-radius: 8px;
  background: #fafbfc;
}

.log-entry--rejected {
  background: #FFF5F5;
  color: #DC2626;
}

.log-entry--approved {
  background: #F0FFF4;
  color: #059669;
}

.log-entry--reflection {
  background: #FFF7ED;
  color: #9A3412;
  font-style: italic;
}

.log-entry--negative {
  color: #DC2626;
}

.log-entry--positive {
  color: #059669;
}

.log-icon {
  flex-shrink: 0;
  font-size: 0.85rem;
}

.log-agent {
  font-weight: 700;
  color: #475569;
  flex-shrink: 0;
}

.log-text {
  color: inherit;
}

.typing-cursor {
  animation: blink 0.8s step-end infinite;
  color: #818CF8;
}

.log-empty {
  text-align: center;
  color: #94a3b8;
  padding: 2rem 0;
  font-size: 0.85rem;
}

/* Transitions */
.log-entry-enter-active {
  transition: all 0.35s cubic-bezier(0.34, 1.56, 0.64, 1);
}
.log-entry-enter-from {
  opacity: 0;
  transform: translateY(-8px) scale(0.97);
}
.log-entry-move {
  transition: transform 0.3s ease;
}

@keyframes blink {
  50% { opacity: 0; }
}
</style>
