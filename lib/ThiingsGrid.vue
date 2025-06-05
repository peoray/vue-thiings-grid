<template>
  <div
    ref="containerRef"
    :class="className"
    :style="{
      position: 'absolute',
      inset: '0',
      touchAction: 'none',
      overflow: 'hidden',
      cursor: isDragging ? 'grabbing' : 'grab',
    }"
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
      :style="{
        position: 'absolute',
        inset: '0',
        transform: `translate3d(${offset.x}px, ${offset.y}px, 0)`,
        willChange: 'transform',
      }"
    >
      <div
        v-for="item in gridItems"
        :key="`${item.position.x}-${item.position.y}`"
        :style="{
          position: 'absolute',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          userSelect: 'none',
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
          :item-config="{
            gridIndex: item.gridIndex,
            position: item.position,
            isMoving,
          }"
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, onUnmounted, computed, watch } from 'vue'

// Grid physics constants
const MIN_VELOCITY = 0.2
const UPDATE_INTERVAL = 16
const VELOCITY_HISTORY_SIZE = 5
const FRICTION = 0.9
const VELOCITY_THRESHOLD = 0.3

// Props
interface Props {
  gridSize: number
  className?: string
  initialPosition?: Position
}

const props = withDefaults(defineProps<Props>(), {
  className: '',
  initialPosition: () => ({ x: 0, y: 0 }),
})

// Types
type Position = {
  x: number
  y: number
}

type GridItem = {
  position: Position
  gridIndex: number
}

export type ItemConfig = {
  isMoving: boolean
  position: Position
  gridIndex: number
}

// Custom debounce implementation
function debounce<T extends (...args: unknown[]) => unknown>(
  func: T,
  wait: number
) {
  let timeoutId: number | undefined = undefined

  const debouncedFn = function (...args: Parameters<T>) {
    if (timeoutId) {
      clearTimeout(timeoutId)
    }
    timeoutId = setTimeout(() => {
      func(...args)
      timeoutId = undefined
    }, wait)
  }

  debouncedFn.cancel = function () {
    clearTimeout(timeoutId)
    timeoutId = undefined
  }

  return debouncedFn
}

// Custom throttle implementation
function throttle<T extends (...args: unknown[]) => unknown>(
  func: T,
  limit: number,
  options: { leading?: boolean; trailing?: boolean } = {}
) {
  let lastCall = 0
  let timeoutId: number | undefined = undefined
  const { leading = true, trailing = true } = options

  const throttledFn = function (...args: Parameters<T>) {
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

  throttledFn.cancel = function () {
    if (timeoutId) {
      clearTimeout(timeoutId)
      timeoutId = undefined
    }
  }

  return throttledFn
}

function getDistance(p1: Position, p2: Position) {
  const dx = p2.x - p1.x
  const dy = p2.y - p1.y
  return Math.sqrt(dx * dx + dy * dy)
}

// Refs
const containerRef = ref<HTMLElement | null>(null)

// Reactive state
const state = reactive({
  offset: { ...props.initialPosition },
  isDragging: false,
  startPos: { ...props.initialPosition },
  restPos: { ...props.initialPosition },
  velocity: { x: 0, y: 0 },
  gridItems: [] as GridItem[],
  isMoving: false,
  lastMoveTime: 0,
  velocityHistory: [] as Position[],
})

// Destructure reactive properties for template
const { offset, isDragging, gridItems, isMoving } = state

// Other refs
const lastPos = ref<Position>({ x: 0, y: 0 })
const animationFrame = ref<number | null>(null)
const isComponentMounted = ref(false)
const lastUpdateTime = ref(0)

// Computed properties
const containerWidth = computed(() => {
  return containerRef.value?.getBoundingClientRect().width || 0
})

const containerHeight = computed(() => {
  return containerRef.value?.getBoundingClientRect().height || 0
})

// Methods
const calculateVisiblePositions = (): Position[] => {
  if (!containerRef.value) return []

  const rect = containerRef.value.getBoundingClientRect()
  const width = rect.width
  const height = rect.height

  // Calculate grid cells needed to fill container
  const cellsX = Math.ceil(width / props.gridSize)
  const cellsY = Math.ceil(height / props.gridSize)

  // Calculate center position based on offset
  const centerX = -Math.round(state.offset.x / props.gridSize)
  const centerY = -Math.round(state.offset.y / props.gridSize)

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

const getItemIndexForPosition = (x: number, y: number): number => {
  // Special case for center
  if (x === 0 && y === 0) return 0

  // Determine which layer of the spiral we're in
  const layer = Math.max(Math.abs(x), Math.abs(y))

  // Calculate the size of all inner layers
  const innerLayersSize = Math.pow(2 * layer - 1, 2)

  // Calculate position within current layer
  let positionInLayer = 0

  if (y === 0 && x === layer) {
    // Starting position (middle right)
    positionInLayer = 0
  } else if (y < 0 && x === layer) {
    // Right side, bottom half
    positionInLayer = -y
  } else if (y === -layer && x > -layer) {
    // Bottom side
    positionInLayer = layer + (layer - x)
  } else if (x === -layer && y < layer) {
    // Left side
    positionInLayer = 3 * layer + (layer + y)
  } else if (y === layer && x < layer) {
    // Top side
    positionInLayer = 5 * layer + (layer + x)
  } else {
    // Right side, top half (y > 0 && x === layer)
    positionInLayer = 7 * layer + (layer - y)
  }

  const index = innerLayersSize + positionInLayer
  return index
}

const debouncedStopMoving = debounce(() => {
  state.isMoving = false
  state.restPos = { ...state.offset }
}, 200)

const updateGridItems = () => {
  if (!isComponentMounted.value) return

  const positions = calculateVisiblePositions()
  const newItems = positions.map((position) => {
    const gridIndex = getItemIndexForPosition(position.x, position.y)
    return {
      position,
      gridIndex,
    }
  })

  const distanceFromRest = getDistance(state.offset, state.restPos)

  state.gridItems = newItems
  state.isMoving = distanceFromRest > 5

  debouncedStopMoving()
}

const debouncedUpdateGridItems = throttle(updateGridItems, UPDATE_INTERVAL, {
  leading: true,
  trailing: true,
})

const animate = () => {
  if (!isComponentMounted.value) return

  const currentTime = performance.now()
  const deltaTime = currentTime - lastUpdateTime.value

  if (deltaTime >= UPDATE_INTERVAL) {
    const { velocity } = state
    const speed = Math.sqrt(velocity.x * velocity.x + velocity.y * velocity.y)

    if (speed < MIN_VELOCITY) {
      state.velocity = { x: 0, y: 0 }
      return
    }

    // Apply non-linear deceleration based on speed
    let deceleration = FRICTION
    if (speed < VELOCITY_THRESHOLD) {
      // Apply stronger deceleration at lower speeds for more natural stopping
      deceleration = FRICTION * (speed / VELOCITY_THRESHOLD)
    }

    state.offset = {
      x: state.offset.x + state.velocity.x,
      y: state.offset.y + state.velocity.y,
    }
    state.velocity = {
      x: state.velocity.x * deceleration,
      y: state.velocity.y * deceleration,
    }

    debouncedUpdateGridItems()
    lastUpdateTime.value = currentTime
  }

  animationFrame.value = requestAnimationFrame(animate)
}

const handleDown = (p: Position) => {
  if (animationFrame.value) {
    cancelAnimationFrame(animationFrame.value)
  }

  state.isDragging = true
  state.startPos = {
    x: p.x - state.offset.x,
    y: p.y - state.offset.y,
  }
  state.velocity = { x: 0, y: 0 }

  lastPos.value = { x: p.x, y: p.y }
}

const handleMove = (p: Position) => {
  if (!state.isDragging) return

  const currentTime = performance.now()
  const timeDelta = currentTime - state.lastMoveTime

  // Calculate raw velocity based on position and time
  const rawVelocity = {
    x: (p.x - lastPos.value.x) / (timeDelta || 1),
    y: (p.y - lastPos.value.y) / (timeDelta || 1),
  }

  // Add to velocity history and maintain fixed size
  const velocityHistory = [...state.velocityHistory, rawVelocity]
  if (velocityHistory.length > VELOCITY_HISTORY_SIZE) {
    velocityHistory.shift()
  }

  // Calculate smoothed velocity using moving average
  const smoothedVelocity = velocityHistory.reduce(
    (acc, vel) => ({
      x: acc.x + vel.x / velocityHistory.length,
      y: acc.y + vel.y / velocityHistory.length,
    }),
    { x: 0, y: 0 }
  )

  state.velocity = smoothedVelocity
  state.offset = {
    x: p.x - state.startPos.x,
    y: p.y - state.startPos.y,
  }
  state.lastMoveTime = currentTime
  state.velocityHistory = velocityHistory

  updateGridItems()

  lastPos.value = { x: p.x, y: p.y }
}

const handleUp = () => {
  state.isDragging = false
  animationFrame.value = requestAnimationFrame(animate)
}

const handleMouseDown = (e: MouseEvent) => {
  handleDown({
    x: e.clientX,
    y: e.clientY,
  })
}

const handleMouseMove = (e: MouseEvent) => {
  e.preventDefault()
  handleMove({
    x: e.clientX,
    y: e.clientY,
  })
}

const handleMouseUp = () => {
  handleUp()
}

const handleTouchStart = (e: TouchEvent) => {
  const touch = e.touches[0]

  if (!touch) return

  handleDown({
    x: touch.clientX,
    y: touch.clientY,
  })
}

const handleTouchMove = (e: TouchEvent) => {
  const touch = e.touches[0]

  if (!touch) return

  handleMove({
    x: touch.clientX,
    y: touch.clientY,
  })
}

const handleTouchEnd = () => {
  handleUp()
}

const handleWheel = (e: WheelEvent) => {
  // Get the scroll deltas
  const deltaX = e.deltaX
  const deltaY = e.deltaY

  state.offset = {
    x: state.offset.x - deltaX,
    y: state.offset.y - deltaY,
  }
  state.velocity = { x: 0, y: 0 } // Reset velocity when scrolling

  debouncedUpdateGridItems()
}

// Public method equivalent
const getCurrentPosition = () => {
  return state.offset
}

// Lifecycle
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

// Expose public methods
defineExpose({
  getCurrentPosition,
})
</script>
