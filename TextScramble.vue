<template>
  <component 
    :is="tag"
    ref="containerRef" 
    :class="className"
  >
    <slot v-if="hasSlotContent && !animationStarted"></slot>
    <template v-else-if="!animationStarted && phrases && phrases.length > 0">
      <span v-for="(char, index) in initialTextChars" :key="index">{{ char }}</span>
    </template>
    <template v-else>
      <span v-for="(char, index) in displayedChars" :key="index" v-html="char"></span>
    </template>
  </component>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, nextTick, onUnmounted, computed, useSlots } from 'vue'

// 定义组件属性
interface Props {
  phrases: string[]
  startDelay?: number
  scrambleDuration?: number
  className?: string
  autoStart?: boolean
  stopAfterFirst?: boolean
  tag?: string
  showInitialText?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  startDelay: 5000,
  scrambleDuration: 4000,
  className: '',
  autoStart: true,
  stopAfterFirst: true, // 默认只执行一次转换
  tag: 'div',
  showInitialText: true
})

// 定义事件
const emit = defineEmits<{
  (e: 'scramble-start'): void
  (e: 'scramble-end'): void
}>()

// 获取插槽内容
const slots = useSlots()
const hasSlotContent = computed(() => {
  return slots.default && slots.default().length > 0
})

// 定义引用
const containerRef = ref<HTMLElement | null>(null)
const animationFrameId = ref<number | null>(null)
const isAnimating = ref(false)
const animationStarted = ref(false)
const displayedChars = ref<string[]>([])

// 计算初始文本字符数组
const initialTextChars = computed(() => {
  if (props.phrases && props.phrases.length > 0) {
    return props.phrases[0].split('')
  }
  return []
})

// 文字扰乱效果类
class TextScramble {
  chars: string
  queue!: { from: string; to: string; start: number; end: number; char?: string }[]
  frame!: number
  resolve!: (value: unknown) => void

  constructor() {
    this.chars = '!&lt;&gt;-_\\\\/[]{}—=+*^?#________'
  }

  async setText(fromText: string, toText: string) {
    const length = Math.max(fromText.length, toText.length)
    const promise = new Promise((resolve) => this.resolve = resolve)
    this.queue = []
    
    for (let i = 0; i < length; i++) {
      const from = fromText[i] || ''
      const to = toText[i] || ''
      const start = Math.floor(Math.random() * 40)
      const end = start + Math.floor(Math.random() * 40)
      this.queue.push({ from, to, start, end })
    }
    
    this.frame = 0
    this.update(fromText.split(''))
    return promise
  }

  update(chars: string[]) {
    let complete = 0
    
    for (let i = 0, n = this.queue.length; i < n; i++) {
      const { from, to, start, end } = this.queue[i]
      let char = this.queue[i].char
      
      if (this.frame >= end) {
        complete++
        chars[i] = to
      } else if (this.frame >= start) {
        if (!char || Math.random() < 0.28) {
          char = this.randomChar()
          this.queue[i].char = char
        }
        chars[i] = `<span class="dud">${char}</span>`
      } else {
        chars[i] = from
      }
    }
    
    // 更新显示的字符
    displayedChars.value = [...chars]
    
    if (complete === this.queue.length) {
      this.resolve(true)
    } else {
      this.frame++
      animationFrameId.value = requestAnimationFrame(() => this.update(chars))
    }
  }

  randomChar() {
    return this.chars[Math.floor(Math.random() * this.chars.length)]
  }
}

// 执行扰乱效果
const performScramble = async () => {
  if (!props.phrases || props.phrases.length < 2 || isAnimating.value) return

  animationStarted.value = true
  isAnimating.value = true
  emit('scramble-start')

  const fx = new TextScramble()
  
  // 获取初始文本
  const initialText = props.phrases[0]
  const targetText = props.phrases[1]
  
  // 执行从英文到中文的变化
  await fx.setText(initialText, targetText)
  
  // 如果设置为只执行一次，则标记动画完成
  if (props.stopAfterFirst) {
    // 显示最终文本
    displayedChars.value = targetText.split('')
  }
  
  emit('scramble-end')
  isAnimating.value = false
}

// 启动扰乱效果
const startScramble = () => {
  if (props.autoStart && !isAnimating.value) {
    setTimeout(() => {
      performScramble()
    }, props.startDelay)
  }
}

// 监听属性变化
watch(() => props.autoStart, (newVal) => {
  if (newVal && !isAnimating.value) {
    startScramble()
  }
})

// 组件挂载后启动效果
onMounted(() => {
  nextTick(() => {
    startScramble()
  })
})

// 组件卸载时清理动画帧
onUnmounted(() => {
  if (animationFrameId.value) {
    cancelAnimationFrame(animationFrameId.value)
  }
})

// 定义组件暴露的方法
defineExpose({
  startScramble,
  performScramble
})
</script>

<style scoped>
.dud {
  color: #777;
}
</style>