<script setup lang="ts">
import { computed, onUnmounted, ref, watch } from 'vue'
import { useIsSlideActive } from '@slidev/client'

const props = withDefaults(defineProps<{ interval?: number }>(), { interval: 2500 })

type Phase = 'observe' | 'diff' | 'act'
type VmState = 'running' | 'crashed' | 'provisioning'
interface Step {
  phase: Phase
  desired: number
  vms: VmState[]
  msg: string
  tone: 'ok' | 'warn' | 'act' | 'info'
}

const R = 'running'
const steps: Step[] = [
  { phase: 'observe', desired: 3, vms: [R, R, R], msg: '👀 Observe: 3 worker VMs running', tone: 'info' },
  { phase: 'diff', desired: 3, vms: [R, R, R], msg: '✅ In sync: desired 3 = actual 3', tone: 'ok' },
  { phase: 'observe', desired: 3, vms: [R, 'crashed', R], msg: '💥 wp-2 crashed! (host died)', tone: 'warn' },
  { phase: 'diff', desired: 3, vms: [R, 'crashed', R], msg: '⚠️ Drift: desired 3, actual 2', tone: 'warn' },
  { phase: 'act', desired: 3, vms: [R, 'provisioning', R], msg: '🔧 Act: create Machine wp-2 (KubeVirt VM)', tone: 'act' },
  { phase: 'observe', desired: 3, vms: [R, R, R], msg: '👀 Observe: wp-2 booted, joined the cluster', tone: 'info' },
  { phase: 'diff', desired: 3, vms: [R, R, R], msg: '✅ In sync again: self-healed', tone: 'ok' },
  { phase: 'observe', desired: 5, vms: [R, R, R], msg: '✏️ kubectl apply: replicas 3 → 5', tone: 'info' },
  { phase: 'diff', desired: 5, vms: [R, R, R], msg: '⚠️ Drift: desired 5, actual 3', tone: 'warn' },
  { phase: 'act', desired: 5, vms: [R, R, R, 'provisioning', 'provisioning'], msg: '🔧 Act: create Machines wp-4, wp-5', tone: 'act' },
  { phase: 'observe', desired: 5, vms: [R, R, R, R, R], msg: '👀 Observe: 5 worker VMs running', tone: 'info' },
  { phase: 'diff', desired: 5, vms: [R, R, R, R, R], msg: '✅ In sync: status updated', tone: 'ok' },
]

const i = ref(0)
const step = computed(() => steps[i.value])

let timer: ReturnType<typeof setInterval> | undefined
const isActive = useIsSlideActive()
function stop() {
  if (timer) clearInterval(timer)
  timer = undefined
}
watch(isActive, (active) => {
  stop()
  if (active) {
    i.value = 0
    timer = setInterval(() => (i.value = (i.value + 1) % steps.length), props.interval)
  }
}, { immediate: true })
onUnmounted(stop)

// Loop geometry
const cx = 170
const cy = 165
const r = 120
const nodes: Record<Phase, { x: number, y: number, label: string, icon: string }> = {
  observe: { x: cx, y: cy - r, label: 'Observe', icon: '👀' },
  diff: { x: cx + r * Math.cos(Math.PI / 6), y: cy + r * Math.sin(Math.PI / 6), label: 'Diff', icon: '⚖️' },
  act: { x: cx - r * Math.cos(Math.PI / 6), y: cy + r * Math.sin(Math.PI / 6), label: 'Act', icon: '🔧' },
}
const orb = computed(() => nodes[step.value.phase])

const toneColor: Record<Step['tone'], string> = {
  ok: '#4ade80',
  warn: '#fbbf24',
  act: '#60a5fa',
  info: '#cbd5e1',
}
</script>

<template>
  <div class="grid grid-cols-[340px_1fr] gap-10 items-center">
    <!-- The loop -->
    <svg viewBox="0 0 340 330" class="w-full">
      <defs>
        <marker id="cl-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto">
          <path d="M0 0L10 5L0 10z" fill="#64748b" />
        </marker>
      </defs>
      <circle :cx="cx" :cy="cy" :r="r" fill="none" stroke="#334155" stroke-width="10" />
      <circle :cx="cx" :cy="cy" :r="r" fill="none" stroke="#64748b" stroke-width="2" stroke-dasharray="6 10" class="cl-flow" />
      <!-- in-sync shortcut -->
      <path
        :d="`M ${nodes.diff.x - 20} ${nodes.diff.y - 30} Q ${cx + 15} ${cy - 10} ${nodes.observe.x + 18} ${nodes.observe.y + 34}`"
        fill="none" stroke="#4ade80" stroke-opacity="0.5" stroke-width="2" stroke-dasharray="4 4" marker-end="url(#cl-arrow)"
      />
      <text :x="cx + 28" :y="cy - 2" fill="#4ade80" fill-opacity="0.8" :style="{ fontSize: '14px' }">in sync</text>

      <circle :cx="orb.x" :cy="orb.y" r="46" fill="#60a5fa" fill-opacity="0.35" class="cl-orb" />

      <g v-for="(n, key) in nodes" :key="key">
        <circle
          :cx="n.x" :cy="n.y" r="34"
          :fill="step.phase === key ? '#1e3a5f' : '#1e293b'"
          :stroke="step.phase === key ? '#60a5fa' : '#475569'"
          stroke-width="3"
          class="cl-node"
          :class="{ 'cl-active': step.phase === key }"
        />
        <text :x="n.x" :y="n.y - 2" text-anchor="middle" :style="{ fontSize: '20px' }">{{ n.icon }}</text>
        <text :x="n.x" :y="n.y + 18" text-anchor="middle" :style="{ fontSize: '12px' }" fill="#e2e8f0">{{ n.label }}</text>
      </g>

    </svg>

    <!-- The example -->
    <div class="flex flex-col gap-5 text-left">
      <div>
        <div class="text-xs uppercase tracking-widest opacity-60 mb-1">🎯 Desired state · spec</div>
        <div class="font-mono text-lg bg-[#1e293b] rounded px-4 py-2 inline-block">
          nodePools[wp].replicas:
          <span :key="step.desired" class="cl-pop inline-block text-[#fbbf24] font-bold">{{ step.desired }}</span>
        </div>
      </div>

      <div>
        <div class="text-xs uppercase tracking-widest opacity-60 mb-2">🌍 Actual state · the real world</div>
        <div class="flex gap-3 h-22">
          <TransitionGroup name="cl-vm">
            <div
              v-for="(vm, idx) in step.vms" :key="idx"
              class="cl-vm w-18 h-20 rounded-lg border-2 flex flex-col items-center justify-center text-xs"
              :class="{
                'border-green-400 bg-green-900/30': vm === 'running',
                'border-red-500 bg-red-900/40 cl-shake': vm === 'crashed',
                'border-blue-400 bg-blue-900/30 border-dashed cl-pulse': vm === 'provisioning',
              }"
            >
              <span class="text-2xl">{{ vm === 'running' ? '🖥️' : vm === 'crashed' ? '💥' : '⏳' }}</span>
              <span class="opacity-80 mt-1">wp-{{ idx + 1 }}</span>
            </div>
          </TransitionGroup>
        </div>
      </div>

      <div
        :key="i"
        class="cl-msg text-lg font-semibold rounded px-4 py-2 border-l-4 bg-[#1e293b]"
        :style="{ borderColor: toneColor[step.tone], color: toneColor[step.tone] }"
      >
        {{ step.msg }}
      </div>
    </div>
  </div>
</template>

<style scoped>
.cl-orb {
  transition: cx 0.7s ease-in-out, cy 0.7s ease-in-out;
  filter: blur(4px);
}
.cl-node {
  transition: fill 0.3s, stroke 0.3s;
}
.cl-active {
  filter: drop-shadow(0 0 10px #60a5fa);
}
.cl-flow {
  animation: cl-dash 1.2s linear infinite;
}
@keyframes cl-dash {
  to { stroke-dashoffset: -32; }
}
.cl-vm {
  transition: all 0.4s;
}
.cl-vm-enter-from {
  opacity: 0;
  transform: scale(0.5);
}
.cl-vm-leave-to {
  opacity: 0;
}
.cl-shake {
  animation: cl-shake 0.4s ease-in-out 2;
}
@keyframes cl-shake {
  25% { transform: translateX(-4px) rotate(-3deg); }
  75% { transform: translateX(4px) rotate(3deg); }
}
.cl-pulse {
  animation: cl-pulse 0.9s ease-in-out infinite;
}
@keyframes cl-pulse {
  50% { opacity: 0.5; }
}
.cl-pop {
  animation: cl-pop 0.5s ease-out;
}
@keyframes cl-pop {
  0% { transform: scale(1.8); }
  100% { transform: scale(1); }
}
.cl-msg {
  animation: cl-fade 0.4s ease-out;
}
@keyframes cl-fade {
  from { opacity: 0; transform: translateY(6px); }
}
</style>
