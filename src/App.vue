<template>
  <div class="app-container">
    <el-header class="app-header">
      <h1>
        <el-icon :size="28"><Coin /></el-icon>
        基金零成本计算器
      </h1>
      <p class="subtitle">输入当前持仓快照，一键测算零成本卖出方案</p>
    </el-header>

    <el-main class="app-main">
      <el-row :gutter="24" justify="center">
        <el-col :xs="24" :sm="22" :md="18" :lg="14" :xl="12">
          <ZeroFundCalculator />
        </el-col>
      </el-row>
    </el-main>

    <el-footer class="app-footer">
      <small>零成本策略：卖出部分份额收回全部本金，剩余份额即为零成本底仓</small>
    </el-footer>
  </div>
</template>

<script setup>
import { Coin } from '@element-plus/icons-vue'
import ZeroFundCalculator from './components/ZeroFundCalculator.vue'
</script>

<style>
:root {
  --primary-light: #ecfdf5;
  --primary-border: #34d399;
  --warning-light: #fffbeb;
  --warning-border: #fbbf24;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,
    'Helvetica Neue', Arial, 'Noto Sans SC', sans-serif;
  background: linear-gradient(135deg, #f0f9ff 0%, #e0f2fe 50%, #f0fdf4 100%);
  min-height: 100vh;
  color: #1f2937;
}

#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.app-header {
  text-align: center;
  padding: 32px 16px 16px !important;
  height: auto !important;
}

.app-header h1 {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  font-size: 1.6rem;
  font-weight: 700;
  color: #065f46;
}

.subtitle {
  color: #6b7280;
  margin-top: 6px;
  font-size: 0.95rem;
}

.app-main {
  flex: 1;
  padding: 16px;
}

.app-footer {
  text-align: center;
  padding: 16px !important;
  height: auto !important;
  color: #9ca3af;
}

@media (max-width: 480px) {
  .app-header h1 {
    font-size: 1.25rem;
  }
  .subtitle {
    font-size: 0.85rem;
  }
}
</style>
