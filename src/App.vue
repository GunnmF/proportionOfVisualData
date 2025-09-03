<template>
  <div class="app">
    <div class="header">
      <h1 class="title">数据可视化</h1>
      <p class="subtitle">半圆形数据占比图表</p>
    </div>

    <!-- 控制面板 -->
    <div class="controls">
      <h3 class="controls-title">自定义图表数据</h3>
      <div class="controls-grid">
        <div class="control-group">
          <label>第一组数据:</label>
          <input
            v-model.number="customData.value1"
            type="number"
            min="0"
            class="control-input" />
        </div>
        <div class="control-group">
          <label>第二组数据:</label>
          <input
            v-model.number="customData.value2"
            type="number"
            min="0"
            class="control-input" />
        </div>
      </div>

      <div class="custom-chart">
        <SemiCircleGauge
          :primary-value="customData.value1"
          :secondary-value="customData.value2"
          primary-label="数据A"
          secondary-label="数据B"
          label="自定义数据"
          primary-color="#f97316"
          secondary-color="#84cc16"
          :size="180" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import SemiCircleGauge from './components/SemiCircleGauge.vue'

const customData = ref({
  value1: 120,
  value2: 80,
})
</script>

<style scoped>
.app {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 40px 20px;
}

.header {
  text-align: center;
  margin-bottom: 48px;
  color: white;
}

.title {
  font-size: 3rem;
  font-weight: 700;
  margin: 0 0 12px 0;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.subtitle {
  font-size: 1.2rem;
  font-weight: 400;
  margin: 0;
  opacity: 0.9;
}

.charts-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 32px;
  max-width: 1200px;
  margin: 0 auto 48px;
}

.chart-card {
  background: white;
  border-radius: 16px;
  padding: 32px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.chart-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.15);
}

.chart-title {
  font-size: 1.4rem;
  font-weight: 600;
  color: #1f2937;
  margin: 0 0 24px;
  text-align: center;
}

.controls {
  max-width: 600px;
  margin: 0 auto;
  background: white;
  border-radius: 16px;
  padding: 32px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
}

.controls-title {
  font-size: 1.4rem;
  font-weight: 600;
  color: #1f2937;
  margin: 0 0 24px;
  text-align: center;
}

.controls-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin-bottom: 32px;
}

.control-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.control-group label {
  font-weight: 500;
  color: #374151;
  font-size: 14px;
}

.control-input {
  padding: 12px;
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  font-size: 16px;
  transition: all 0.2s ease;
  background: #f9fafb;
}

.control-input:focus {
  outline: none;
  border-color: #3b82f6;
  background: white;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.custom-chart {
  display: flex;
  justify-content: center;
  padding: 20px;
  background: #f8fafc;
  border-radius: 12px;
  margin-top: 8px;
}

@media (max-width: 768px) {
  .app {
    padding: 20px 16px;
  }

  .title {
    font-size: 2.2rem;
  }

  .charts-grid {
    grid-template-columns: 1fr;
    gap: 24px;
  }

  .chart-card {
    padding: 24px;
  }

  .controls {
    padding: 24px;
  }

  .controls-grid {
    grid-template-columns: 1fr;
    gap: 16px;
  }
}
</style>
