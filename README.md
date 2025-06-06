# VueThiingsGrid

A high-performance, infinite scrolling grid component for Vue that provides smooth touch/mouse interactions with momentum-based scrolling. Perfect for displaying large datasets in a grid format with custom cell renderers.

## 🪩 [**Explore Thiings.co →**](https://thiings.co)

> This is the component that powers the interactive grid on [thiings.co](https://thiings.co/) - A growing collection of 1200+ free 3D icons, generated with AI.

## 🎮 [**Try the Live Playground →**](https://peoray.github.io/vue-thiings-grid)

> Experience ThiingsGrid in action with interactive examples and copy-paste ready code.

## ✨ Features

- 🚀 **High Performance**: Only renders visible cells with optimized viewport calculations
- 📱 **Touch & Mouse Support**: Smooth interactions on both desktop and mobile
- 🎯 **Momentum Scrolling**: Natural physics-based scrolling with inertia
- ♾️ **Infinite Grid**: Supports unlimited grid sizes with efficient rendering
- 🎨 **Custom Renderers**: Flexible cell rendering with your own components
- 🔧 **TypeScript Support**: Full type safety with comprehensive TypeScript definitions

## 🚀 Quick Start

### Installation

This component is currently part of this repository. To use it in your project:

1. Copy the `lib/ThiingsGrid.tsx` file to your project
2. Install the required dependencies:

```bash
npm install react react-dom
```

#### Basic Usage

```vue
<template>
	<div :style="{ width: '100vw', height: '100vh' }">
		<ThiingsGrid :grid-size="80">
			<template #default="{ gridIndex, position }">
				<div class="absolute inset-1 flex items-center justify-center">
					{{ gridIndex }}
				</div>
			</template>
		</ThiingsGrid>
	</div>
</template>

<script setup lang="ts">
import ThiingsGrid from './path/to/ThiingsGrid.vue';
</script>
```

## 📚 API Reference

### Props

| Prop              | Type     | Required | Default          | Description                      |
| ----------------- | -------- | -------- | ---------------- | -------------------------------- |
| `gridSize`        | number   | ✅       | -                | Size of each grid cell in pixels |
| `className`       | string   | ✅       | -                | CSS class name for the container |
| `initialPosition` | Position | ❌       | `{ x: 0, y: 0 }` | Initial scroll position          |

### Slot Props

The component provides the following slot props for custom item rendering:

| Prop        | Type     | Description                                                           |
| ----------- | -------- | --------------------------------------------------------------------- |
| `gridIndex` | number   | Unique index for each grid item                                       |
| `position`  | Position | Current position of the item in the grid - `{ x: number, y: number }` |
| `isMoving`  | boolean  | Whether the grid is currently in motion                               |

### Position

```typescript
interface Position {
	x: number;
	y: number;
}
```

## 🎨 Examples

### Simple Numbers

```vue
<template>
	<ThiingsGrid :gridSize="80" class="bg-gray-100">
		<template #default="{ gridIndex }">
			<div
				class="absolute inset-1 flex items-center justify-center bg-blue-50 border border-blue-500 rounded text-sm font-bold text-blue-800"
			>
				{{ gridIndex }}
			</div>
		</template>
	</ThiingsGrid>
</template>

<script setup lang="ts">
import ThiingsGrid, { type ItemConfig } from '../../lib/ThiingsGrid.vue';
</script>
```

## Colorful Grid

```vue
<template>
	<ThiingsGrid :grid-size="100">
		<template #default="{ gridIndex }">
			<div
				:class="`absolute inset-0 flex items-center justify-center ${getColorClass(
					gridIndex
				)} text-xs font-bold text-gray-800 shadow-sm`"
			>
				{{ gridIndex }}
			</div>
		</template>
	</ThiingsGrid>
</template>

<script setup lang="ts">
import ThiingsGrid, { type ItemConfig } from '../../lib/ThiingsGrid.vue';

const colors = [
	'bg-red-300',
	'bg-green-300',
	'bg-blue-300',
	'bg-yellow-300',
	'bg-pink-300',
	'bg-cyan-300',
];

const getColorClass = (gridIndex: number): string => {
	return colors[gridIndex % colors.length];
};
</script>
```

### Interactive Cards

```vue
<template>
	<ThiingsGrid :grid-size="150">
		<template #default="{ gridIndex, position, isMoving }">
			<div
				:class="[
					'absolute',
					'inset-1',
					'flex',
					'flex-col',
					'items-center',
					'justify-center',
					'bg-white',
					'border',
					'border-gray-200',
					'rounded-xl',
					'p-2',
					'text-xs',
					'text-gray-800',
					'transition-shadow',
					isMoving ? 'shadow-xl' : 'shadow-md',
				]"
			>
				<div class="text-base font-bold mb-1">#{{ gridIndex }}</div>
				<div class="text-[10px] text-gray-500">
					{{ position.x }}, {{ position.y }}
				</div>
			</div>
		</template>
	</ThiingsGrid>
</template>

<script setup lang="ts">
import ThiingsGrid, { type ItemConfig } from '../../lib/ThiingsGrid.vue';
</script>
```

## 🎯 Best Practices

### Cell Positioning

Always use absolute positioning within your cell components for optimal performance:

```vue
// ✅ Good
<template #default="{ gridIndex }">
	<div class="absolute inset-1 ...">
		{{ gridIndex }}
	</div>
</template>

// ❌ Avoid - can cause layout issues
<template #default="{ gridIndex }">
	<div class="w-full h-full ...">
		{{ gridIndex }}
	</div>
</template>
```

### Container Setup

Ensure the ThiingsGrid has a defined container size:

```tsx
// ✅ Good - explicit container size
<div style={ width: '100vw', height: '100vh' }>
  ...
</div>

// ✅ Good - CSS classes with defined dimensions
<div class="w-screen h-screen">
  ...
</div>
```

## 🔧 Advanced Usage

### Custom Grid Index Calculation

The `gridIndex` is calculated based on the grid position using a custom algorithm that provides unique indices for each cell position.

### Accessing Grid Position

You can access the current grid position programmatically:

```tsx
<template>
  <div>
    <ThiingsGrid
      ref="gridRef"
      :grid-size="80"
    >
      <template #default="{ gridIndex }">
        <div
        class="absolute inset-1 ...
      >
        {{ gridIndex }}
      </div>
      </template>
    </ThiingsGrid>
    <button @click="getCurrentPosition">Log Current Position</button>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import ThiingsGrid from '../../lib/ThiingsGrid.vue'; // Adjust path as needed

// Define the grid ref
const gridRef = ref<InstanceType<typeof ThiingsGrid> | null>(null);

// Method to get the current grid position
const getCurrentPosition = () => {
  if (gridRef.value) {
    const position = gridRef.value.publicGetCurrentPosition();
    console.log('Current position:', position);
  }
};
</script>
```

### Responsive Grid Sizes

```vue
// useResponsiveGridSize composable
export const useResponsiveGridSize = () => {
  const gridSize = ref(80);

  const handleResize = () => {
    const width = window.innerWidth;
    if (width < 768) {
      gridSize.value = 60; // Smaller on mobile
    } else if (width < 1024) {
      gridSize.value = 80; // Medium on tablet
    } else {
      gridSize.value = 100; // Larger on desktop
    }
  };

  onMounted(() => {
    handleResize();
    window.addEventListener('resize', handleResize);
  });

  onUnmounted(() => {
    window.removeEventListener('resize', handleResize);
  });

  return gridSize;
};
</script>

<!-- ResponsiveGrid Component -->
<template>
  <ThiingsGrid :grid-size="gridSize">
    <template #default="{ gridIndex, position, isMoving }">
      <div>{{ gridIndex }}</div>
    </template>
  </ThiingsGrid>
</template>

<script setup lang="ts">
import ThiingsGrid from './ThiingsGrid.vue';

const gridSize = useResponsiveGridSize();
</script>

```

## 🎮 Interaction

### Touch/Mouse Events

The component handles:

- **Mouse**: Click and drag to pan
- **Touch**: Touch and drag to pan
- **Wheel**: Scroll wheel for precise movements
- **Momentum**: Automatic momentum scrolling with physics

## 🔍 Development

### Running the Demo

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build
```

### Project Structure

```
src/
├── examples/           # Example implementations
│   ├── SimpleNumbers.vue
│   ├── ColorfulGrid.vue
│   ├── EmojiFun.vue
│   └── CardLayout.vue
├── components/         # Example implementations
│   ├── Playground.vue  # Example viewer
│   ├── SourceCode.vue  # Source code display
│   └── Sidebar.vue		  # Example navigation
├── App.vue             # Main demo application
|
lib/
└── ThiingsGrid.vue    # Main component
```

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

[MIT License](/License) - see the LICENSE file for details.

## 🙏 Acknowledgments

- Charlie Clark's [React version](https://github.com/charlieclark/thiings-grid)
- Built with Vue and TypeScript
- Styled with Tailwind CSS
- Bundled with Vite
