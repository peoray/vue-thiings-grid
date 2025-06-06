<template>
  <div
    ref="containerRef"
    :class="className"
    class="absolute inset-0 overflow-hidden"
    :style="{ cursor: isDragging ? 'grabbing' : 'grab', touchAction: 'none' }"
    @mousedown="handleMouseDown"
    @mousemove="handleMouseMove"
    @mouseup="handleMouseUp"
    @mouseleave="handleMouseUp"
    @touchstart="handleTouchStart"
    @touchmove="handleTouchMove"
    @touchend="handleTouchEnd"
    @touchcancel="handleTouchEnd"
    @wheel="handleWheel"
  >
    <div
      class="absolute inset-0"
      :style="{
        transform: `translate3d(${offset.x}px, ${offset.y}px, 0)`,
        willChange: 'transform',
      }"
    >
      <div
        v-for="item in gridItems"
        :key="`${item.position.x}-${item.position.y}`"
        class="absolute flex items-center justify-center select-none"
        :style="{
          width: `${gridSize}px`,
          height: `${gridSize}px`,
          transform: `translate3d(${
            item.position.x * gridSize + containerWidth / 2
          }px, ${item.position.y * gridSize + containerHeight / 2}px, 0)`,
          marginLeft: `-${gridSize / 2}px`,
          marginTop: `-${gridSize / 2}px`,
          willChange: 'transform',
        }"
      >
        <slot
          :gridIndex="item.gridIndex"
          :position="item.position"
          :isMoving="isMoving"
        ></slot>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'

// Grid physics constants
const MIN_VELOCITY = 0.2
const UPDATE_INTERVAL = 16
const VELOCITY_HISTORY_SIZE = 5
const FRICTION = 0.9
const VELOCITY_THRESHOLD = 0.3

// Type definitions
interface Position {
  x: number
  y: number
}

interface GridItem {
  position: Position
  gridIndex: number
}

// interface ItemConfig {
//   isMoving: boolean
//   position: Position
//   gridIndex: number
// }

const props = withDefaults(
  defineProps<{
    gridSize: number
    className?: string
    initialPosition?: Position
  }>(),
  {
    initialPosition: () => ({ x: 0, y: 0 }),
  }
)

// Define reactive state
const offset = ref<Position>({ ...props.initialPosition })
const restPos = ref<Position>({ ...props.initialPosition })
const startPos = ref<Position>({ ...props.initialPosition })
const velocity = ref<Position>({ x: 0, y: 0 })
const isDragging = ref<boolean>(false)
const gridItems = ref<GridItem[]>([])
const isMoving = ref<boolean>(false)
const lastMoveTime = ref<number>(0)
const velocityHistory = ref<Position[]>([])
const containerRef = ref<HTMLElement | null>(null)
const lastPos = ref<Position>({ x: 0, y: 0 })
const animationFrame = ref<number | null>(null)
const isComponentMounted = ref<boolean>(false)
const lastUpdateTime = ref<number>(0)

// Custom debounce implementation
function debounce<T extends (...args: any[]) => void>(func: T, wait: number) {
  let timeoutId: number | undefined

  const debouncedFn = (...args: Parameters<T>) => {
    if (timeoutId) {
      clearTimeout(timeoutId)
    }
    timeoutId = setTimeout(() => {
      func(...args)
      timeoutId = undefined
    }, wait)
  }

  debouncedFn.cancel = () => {
    if (timeoutId) {
      clearTimeout(timeoutId)
      timeoutId = undefined
    }
  }

  return debouncedFn
}

// Custom throttle implementation
function throttle<T extends (...args: any[]) => void>(
  func: T,
  limit: number,
  options: { leading?: boolean; trailing?: boolean } = {}
) {
  let lastCall = 0
  let timeoutId: number | undefined
  const { leading = true, trailing = true } = options

  const throttledFn = (...args: Parameters<T>) => {
    const now = Date.now()

    if (!lastCall && !leading) {
      lastCall = now
    }

    const remaining = limit - (now - lastCall)

    if (remaining <= 0 || remaining > limit) {
      clearTimeout(timeoutId)
      timeoutId = undefined
      lastCall = now
      func(...args)
    } else if (!timeoutId && trailing) {
      timeoutId = setTimeout(() => {
        lastCall = leading ? Date.now() : 0
        timeoutId = undefined
        func(...args)
      }, remaining)
    }
  }

  throttledFn.cancel = () => {
    if (timeoutId) {
      clearTimeout(timeoutId)
      timeoutId = undefined
    }
  }

  return throttledFn
}

function getDistance(p1: Position, p2: Position): number {
  const dx = p2.x - p1.x
  const dy = p2.y - p1.y
  return Math.sqrt(dx * dx + dy * dy)
}

const debouncedStopMoving = debounce(() => {
  isMoving.value = false
  restPos.value = { ...offset.value }
}, 200)

const debouncedUpdateGridItems = throttle(updateGridItems, UPDATE_INTERVAL, {
  leading: true,
  trailing: true,
})

function calculateVisiblePositions(): Position[] {
  if (!containerRef.value) return []

  const rect = containerRef.value.getBoundingClientRect()
  const width = rect.width
  const height = rect.height

  const cellsX = Math.ceil(width / props.gridSize)
  const cellsY = Math.ceil(height / props.gridSize)

  const centerX = -Math.round(offset.value.x / props.gridSize)
  const centerY = -Math.round(offset.value.y / props.gridSize)

  const positions: Position[] = []
  const halfCellsX = Math.ceil(cellsX / 2)
  const halfCellsY = Math.ceil(cellsY / 2)

  for (let y = centerY - halfCellsY; y <= centerY + halfCellsY; y++) {
    for (let x = centerX - halfCellsX; x <= centerX + halfCellsX; x++) {
      positions.push({ x, y })
    }
  }

  return positions
}

function getItemIndexForPosition(x: number, y: number): number {
  if (x === 0 && y === 0) return 0

  const layer = Math.max(Math.abs(x), Math.abs(y))
  const innerLayersSize = Math.pow(2 * layer - 1, 2)
  let positionInLayer = 0

  if (y === 0 && x === layer) {
    positionInLayer = 0
  } else if (y < 0 && x === layer) {
    positionInLayer = -y
  } else if (y === -layer && x > -layer) {
    positionInLayer = layer + (layer - x)
  } else if (x === -layer && y < layer) {
    positionInLayer = 3 * layer + (layer + y)
  } else if (y === layer && x < layer) {
    positionInLayer = 5 * layer + (layer + x)
  } else {
    positionInLayer = 7 * layer + (layer - y)
  }

  return innerLayersSize + positionInLayer
}

function updateGridItems() {
  if (!isComponentMounted.value) return

  const positions = calculateVisiblePositions()
  const newItems = positions.map((position) => ({
    position,
    gridIndex: getItemIndexForPosition(position.x, position.y),
  }))

  const distanceFromRest = getDistance(offset.value, restPos.value)
  gridItems.value = newItems
  isMoving.value = distanceFromRest > 5

  debouncedStopMoving()
}

function animate() {
  if (!isComponentMounted.value) return

  const currentTime = performance.now()
  const deltaTime = currentTime - lastUpdateTime.value

  if (deltaTime >= UPDATE_INTERVAL) {
    const speed = Math.sqrt(
      velocity.value.x * velocity.value.x + velocity.value.y * velocity.value.y
    )

    if (speed < MIN_VELOCITY) {
      velocity.value = { x: 0, y: 0 }
      return
    }

    let deceleration = FRICTION
    if (speed < VELOCITY_THRESHOLD) {
      deceleration = FRICTION * (speed / VELOCITY_THRESHOLD)
    }

    offset.value = {
      x: offset.value.x + velocity.value.x,
      y: offset.value.y + velocity.value.y,
    }
    velocity.value = {
      x: velocity.value.x * deceleration,
      y: velocity.value.y * deceleration,
    }

    debouncedUpdateGridItems()
    lastUpdateTime.value = currentTime
  }

  animationFrame.value = requestAnimationFrame(animate)
}

function handleDown(p: Position) {
  if (animationFrame.value) {
    cancelAnimationFrame(animationFrame.value)
  }

  isDragging.value = true
  startPos.value = {
    x: p.x - offset.value.x,
    y: p.y - offset.value.y,
  }
  velocity.value = { x: 0, y: 0 }
  lastPos.value = { x: p.x, y: p.y }
}

function handleMove(p: Position) {
  if (!isDragging.value) return

  const currentTime = performance.now()
  const timeDelta = currentTime - lastMoveTime.value

  const rawVelocity: Position = {
    x: (p.x - lastPos.value.x) / (timeDelta || 1),
    y: (p.y - lastPos.value.y) / (timeDelta || 1),
  }

  velocityHistory.value = [...velocityHistory.value, rawVelocity]
  if (velocityHistory.value.length > VELOCITY_HISTORY_SIZE) {
    velocityHistory.value.shift()
  }

  const smoothedVelocity = velocityHistory.value.reduce(
    (acc, vel) => ({
      x: acc.x + vel.x / velocityHistory.value.length,
      y: acc.y + vel.y / velocityHistory.value.length,
    }),
    { x: 0, y: 0 }
  )

  velocity.value = smoothedVelocity
  offset.value = {
    x: p.x - startPos.value.x,
    y: p.y - startPos.value.y,
  }
  lastMoveTime.value = currentTime
  velocityHistory.value = velocityHistory.value
  updateGridItems()
  lastPos.value = { x: p.x, y: p.y }
}

function handleUp() {
  isDragging.value = false
  animationFrame.value = requestAnimationFrame(animate)
}

function handleMouseDown(e: MouseEvent) {
  handleDown({ x: e.clientX, y: e.clientY })
}

function handleMouseMove(e: MouseEvent) {
  e.preventDefault()
  handleMove({ x: e.clientX, y: e.clientY })
}

function handleMouseUp() {
  handleUp()
}

function handleTouchStart(e: TouchEvent) {
  const touch = e.touches[0]
  if (!touch) return
  handleDown({ x: touch.clientX, y: touch.clientY })
}

function handleTouchMove(e: TouchEvent) {
  const touch = e.touches[0]
  if (!touch) return
  handleMove({ x: touch.clientX, y: touch.clientY })
}

function handleTouchEnd() {
  handleUp()
}

function handleWheel(e: WheelEvent) {
  const deltaX = e.deltaX
  const deltaY = e.deltaY

  offset.value = {
    x: offset.value.x - deltaX,
    y: offset.value.y - deltaY,
  }
  velocity.value = { x: 0, y: 0 }
  debouncedUpdateGridItems()
}

// Computed properties for container dimensions
const containerWidth = computed(
  () => containerRef.value?.getBoundingClientRect().width || 0
)
const containerHeight = computed(
  () => containerRef.value?.getBoundingClientRect().height || 0
)

// Expose public method
defineExpose({
  publicGetCurrentPosition: () => offset.value,
})

onMounted(() => {
  isComponentMounted.value = true
  updateGridItems()
})

onUnmounted(() => {
  isComponentMounted.value = false
  if (animationFrame.value) {
    cancelAnimationFrame(animationFrame.value)
  }
  debouncedUpdateGridItems.cancel()
})
</script>

<style scoped>
/* No additional styles needed as Tailwind classes are used in the template */
</style>
