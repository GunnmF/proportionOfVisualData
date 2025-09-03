<template>
  <div class="gauge-container">
    <div class="gauge-wrapper">
      <svg :width="svgSize" :height="svgSize / 2 + 40" class="gauge-svg">
        <!-- Background Arc -->
        <path
          :d="backgroundPath"
          fill="none"
          stroke="#f3f4f6"
          :stroke-width="strokeWidth"
          stroke-linecap="round"
        />
        
        <!-- Data Arc 1 (Primary) -->
        <path
          :d="primaryPath"
          fill="none"
          :stroke="primaryColor"
          :stroke-width="strokeWidth"
          stroke-linecap="round"
          class="data-arc"
          :stroke-dasharray="primaryDashArray"
          :stroke-dashoffset="primaryDashOffset"
        />
        
        <!-- Data Arc 2 (Secondary) -->
        <path
          :d="secondaryPath"
          fill="none"
          :stroke="secondaryColor"
          :stroke-width="strokeWidth"
          stroke-linecap="round"
          class="data-arc"
          :stroke-dasharray="secondaryDashArray"
          :stroke-dashoffset="secondaryDashOffset"
        />
        
        <!-- 连接点圆圈 -->
        <!-- 第一个数据段的尾部圆点 -->
        <circle
          v-if="primaryValue > 0"
          :cx="primaryEndPoint.x"
          :cy="primaryEndPoint.y"
          :r="strokeWidth / 3"
          fill="#fff"
          class="connection-dot">
          <animateTransform
            attributeName="transform"
            type="translate"
            :values="`0,0; ${primaryEndPoint.x - centerX},${primaryEndPoint.y - centerY}; 0,0`"
            dur="0.8s"
            repeatCount="1"
            calcMode="spline"
            keySplines="0.4,0,0.2,1"
            keyTimes="0;0.5;1"
          />
        </circle>
        
        <!-- 第二个数据段的头部圆点 -->
        <circle
          v-if="secondaryValue > 0"
          :cx="secondaryStartPoint.x"
          :cy="secondaryStartPoint.y"
          :r="strokeWidth / 3"
          fill="#fff"
          class="connection-dot">
          <animateTransform
            attributeName="transform"
            type="translate"
            :values="`0,0; ${secondaryStartPoint.x - centerX},${secondaryStartPoint.y - centerY}; 0,0`"
            dur="0.8s"
            repeatCount="1"
            calcMode="spline"
            keySplines="0.4,0,0.2,1"
            keyTimes="0;0.5;1"
          />
        </circle>
        
        <!-- 第二个数据段的尾部圆点 -->
        <circle
          v-if="secondaryValue > 0"
          :cx="secondaryEndPoint.x"
          :cy="secondaryEndPoint.y"
          :r="strokeWidth / 3"
          :fill="secondaryColor"
          class="connection-dot">
          <animateTransform
            attributeName="transform"
            type="translate"
            :values="`0,0; ${secondaryEndPoint.x - centerX},${secondaryEndPoint.y - centerY}; 0,0`"
            dur="0.8s"
            repeatCount="1"
            calcMode="spline"
            keySplines="0.4,0,0.2,1"
            keyTimes="0;0.5;1"
          />
        </circle>
        
        <!-- Center Text -->
        <g class="center-text">
          <text
            :x="svgSize / 2"
            :y="svgSize / 2 - 10"
            text-anchor="middle"
            class="value-text"
          >
            {{ displayValue }}
          </text>
          <text
            :x="svgSize / 2"
            :y="svgSize / 2 + 20"
            text-anchor="middle"
            class="label-text"
          >
            {{ label }}
          </text>
        </g>
      </svg>
    </div>
    
    <!-- Legend -->
    <div class="legend">
      <div class="legend-item">
        <div class="legend-color" :style="{ backgroundColor: primaryColor }"></div>
        <span>{{ primaryLabel }} ({{ primaryValue }})</span>
      </div>
      <div class="legend-item">
        <div class="legend-color" :style="{ backgroundColor: secondaryColor }"></div>
        <span>{{ secondaryLabel }} ({{ secondaryValue }})</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'

interface Props {
  primaryValue: number
  secondaryValue: number
  primaryLabel?: string
  secondaryLabel?: string
  primaryColor?: string
  secondaryColor?: string
  label?: string
  size?: number
  strokeWidth?: number
}

const props = withDefaults(defineProps<Props>(), {
  primaryLabel: '已使用',
  secondaryLabel: '可用',
  primaryColor: '#3b82f6',
  secondaryColor: '#22d3ee',
  label: '总可用机位',
  size: 200,
  strokeWidth: 16
})

const svgSize = computed(() => props.size)
const radius = computed(() => (props.size - props.strokeWidth * 2) / 2)
const centerX = computed(() => props.size / 2)
const centerY = computed(() => props.size / 2)

const totalValue = computed(() => props.primaryValue + props.secondaryValue)
const displayValue = computed(() => totalValue.value.toString())

// 计算圆弧的周长
const arcLength = computed(() => Math.PI * radius.value)

const primaryPercentage = computed(() => 
  totalValue.value > 0 ? props.primaryValue / totalValue.value : 0
)
const secondaryPercentage = computed(() => 
  totalValue.value > 0 ? props.secondaryValue / totalValue.value : 0
)

// 计算第一个数据段（从起点开始）
// 添加小间隙避免重叠
const gapPercentage = 0.02 // 2% 的间隙

const primaryDashArray = computed(() => {
  const adjustedPercentage = Math.max(0, primaryPercentage.value - gapPercentage / 2)
  const length = arcLength.value * adjustedPercentage
  return `${length} ${arcLength.value}`
})

const primaryDashOffset = computed(() => 0)

// 计算第二个数据段（紧接着第一个数据段）
const secondaryDashArray = computed(() => {
  const adjustedPercentage = Math.max(0, secondaryPercentage.value - gapPercentage / 2)
  const length = arcLength.value * adjustedPercentage
  return `${length} ${arcLength.value}`
})

const secondaryDashOffset = computed(() => {
  // 第二个数据段从第一个数据段结束的地方开始，加上间隙
  const primaryLength = arcLength.value * Math.max(0, primaryPercentage.value - gapPercentage / 2)
  const gap = arcLength.value * gapPercentage
  return -(primaryLength + gap)
})

// 计算连接点的位置
const primaryEndPoint = computed(() => {
  // 第一个数据段的结束点
  const adjustedPercentage = Math.max(0, primaryPercentage.value - gapPercentage / 2)
  const angle = Math.PI - (Math.PI * adjustedPercentage)
  return polarToCartesian(centerX.value, centerY.value, radius.value, angle)
})

const secondaryStartPoint = computed(() => {
  // 第二个数据段的开始点（在第一个数据段结束后，加上间隙）
  const primaryEndPercentage = Math.max(0, primaryPercentage.value - gapPercentage / 2)
  const secondaryStartPercentage = primaryEndPercentage + gapPercentage
  const angle = Math.PI - (Math.PI * secondaryStartPercentage)
  return polarToCartesian(centerX.value, centerY.value, radius.value, angle)
})

const secondaryEndPoint = computed(() => {
  // 第二个数据段的结束点
  const primaryEndPercentage = Math.max(0, primaryPercentage.value - gapPercentage / 2)
  const secondaryStartPercentage = primaryEndPercentage + gapPercentage
  const secondaryEndPercentage = secondaryStartPercentage + Math.max(0, secondaryPercentage.value - gapPercentage / 2)
  const angle = Math.PI - (Math.PI * secondaryEndPercentage)
  return polarToCartesian(centerX.value, centerY.value, radius.value, angle)
  return polarToCartesian(centerX.value, centerY.value, radius.value, angle)
})

// Create path for background semicircle
const backgroundPath = computed(() => {
  const startAngle = Math.PI  // 从左侧开始
  const endAngle = 0          // 到右侧结束
  return createArcPath(centerX.value, centerY.value, radius.value, startAngle, endAngle)
})

// Create path for primary data arc
const primaryPath = computed(() => {
  // 第一个数据段的完整路径
  const startAngle = Math.PI
  const endAngle = 0
  return createArcPath(centerX.value, centerY.value, radius.value, startAngle, endAngle)
})

// Create path for secondary data arc
const secondaryPath = computed(() => {
  // 第二个数据段的完整路径
  const startAngle = Math.PI
  const endAngle = 0
  return createArcPath(centerX.value, centerY.value, radius.value, startAngle, endAngle)
})

function createArcPath(x: number, y: number, radius: number, startAngle: number, endAngle: number): string {
  // 如果角度差太小，返回空路径避免变形
  const angleDiff = Math.abs(endAngle - startAngle)
  if (angleDiff < 0.01) {
    return ""
  }
  
  const start = polarToCartesian(x, y, radius, startAngle)
  const end = polarToCartesian(x, y, radius, endAngle)
  
  // 修正大弧标志逻辑
  const largeArcFlag = angleDiff > Math.PI ? "1" : "0"
  
  // 确定扫描方向：从大角度到小角度用逆时针(0)，从小角度到大角度用顺时针(1)
  const sweepFlag = startAngle > endAngle ? "0" : "1"
  
  return [
    "M", start.x, start.y,
    "A", radius, radius, 0, largeArcFlag, sweepFlag, end.x, end.y
  ].join(" ")
}

function polarToCartesian(centerX: number, centerY: number, radius: number, angleInRadians: number) {
  return {
    x: centerX + (radius * Math.cos(Math.PI - angleInRadians)),  // 翻转X坐标
    y: centerY - (radius * Math.sin(Math.PI - angleInRadians))   // 翻转Y坐标，让半圆在上方
  }
}
</script>

<style scoped>
.gauge-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  padding: 20px;
}

.gauge-wrapper {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
}

.gauge-svg {
  filter: drop-shadow(0 4px 6px rgba(0, 0, 0, 0.1));
}

.data-arc {
  transition: stroke-dasharray 0.8s cubic-bezier(0.4, 0, 0.2, 1),
              stroke-dashoffset 0.8s cubic-bezier(0.4, 0, 0.2, 1);
}

.connection-dot {
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.2));
  transform-origin: center;
}

.center-text {
  animation: fadeIn 1s ease-out 1.2s both;
}

.value-text {
  font-size: 28px;
  font-weight: 700;
  fill: #1f2937;
  font-family: 'Inter', sans-serif;
}

.label-text {
  font-size: 14px;
  font-weight: 500;
  fill: #6b7280;
  font-family: 'Inter', sans-serif;
}

.legend {
  display: flex;
  gap: 24px;
  flex-wrap: wrap;
  justify-content: center;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  background: #f9fafb;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  color: #374151;
  transition: all 0.2s ease;
}

.legend-item:hover {
  background: #f3f4f6;
  transform: translateY(-1px);
}

.legend-color {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  flex-shrink: 0;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (max-width: 640px) {
  .gauge-container {
    padding: 16px;
  }
  
  .legend {
    gap: 16px;
  }
  
  .legend-item {
    font-size: 13px;
  }
}
</style>