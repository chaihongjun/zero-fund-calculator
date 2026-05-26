<template>
  <div class="calculator">
    <el-card shadow="never" class="input-card">
      <template #header>
        <div class="card-header">
          <el-icon :size="20"><EditPen /></el-icon>
          <span>持仓数据录入</span>
        </div>
      </template>

      <el-form label-position="top" @submit.prevent>
        <el-row :gutter="16">
          <el-col :xs="24" :sm="12">
            <el-form-item label="持有份额（份）">
              <el-input
                v-model.number="shares"
                type="number"
                step="0.01"
                min="0"
                placeholder="例：1000.50"
                :prefix-icon="TrendCharts"
                clearable
              />
            </el-form-item>
          </el-col>
          <el-col :xs="24" :sm="12">
            <el-form-item label="持仓成本价（元/份）">
              <el-input
                v-model.number="costPrice"
                type="number"
                step="0.0001"
                min="0"
                placeholder="例：1.2500"
                :prefix-icon="Coin"
                clearable
              />
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="16">
          <el-col :xs="24" :sm="12">
            <el-form-item label="当前单位净值（元/份）">
              <el-input
                v-model.number="nav"
                type="number"
                step="0.0001"
                min="0"
                placeholder="例：1.4800"
                :prefix-icon="Money"
                clearable
              />
            </el-form-item>
          </el-col>
          <el-col :xs="24" :sm="12">
            <el-form-item label="赎回费率">
              <div class="fee-group">
                <el-switch v-model="considerFee" active-text="考虑费率" />
                <el-input
                  v-if="considerFee"
                  v-model.number="feeRate"
                  type="number"
                  step="0.01"
                  min="0"
                  max="100"
                  class="fee-input"
                >
                  <template #suffix>%</template>
                </el-input>
                <span v-else class="fee-disabled">不计费率</span>
              </div>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
    </el-card>

    <el-card v-if="isValid" shadow="never" class="result-card">
      <template #header>
        <div class="card-header">
          <el-icon :size="20"><DataAnalysis /></el-icon>
          <span>持仓概览</span>
        </div>
      </template>

      <el-row :gutter="12">
        <el-col :xs="12" :sm="6" v-for="stat in stats" :key="stat.label">
          <div :class="['stat-item', stat.cls]">
            <span class="stat-label">{{ stat.label }}</span>
            <span class="stat-value">{{ stat.value }}</span>
          </div>
        </el-col>
      </el-row>
    </el-card>

    <el-card v-if="isValid" :class="['action-card', canZeroCost ? 'is-zero' : 'not-zero']">
      <template #header>
        <div class="card-header">
          <el-icon :size="22" v-if="canZeroCost"><SuccessFilled /></el-icon>
          <el-icon :size="22" v-else><WarningFilled /></el-icon>
          <span>{{ canZeroCost ? '🎉 已满足零成本条件' : '⚠️ 尚未达到零成本条件' }}</span>
        </div>
      </template>

      <template v-if="canZeroCost">
        <el-row :gutter="12">
          <el-col :xs="24" :sm="12" v-for="item in sellPlan" :key="item.label">
            <div class="plan-item">
              <span class="plan-label">{{ item.label }}</span>
              <span class="plan-value">{{ item.value }}</span>
            </div>
          </el-col>
        </el-row>
        <el-alert
          title="操作建议：赎回上述金额后，剩余份额即为零成本底仓，后续涨跌均为纯利润。"
          type="success"
          :closable="false"
          show-icon
          class="alert-tip"
        />
      </template>

      <template v-else>
        <el-row :gutter="12">
          <el-col :xs="24" :sm="12" v-for="item in gapInfo" :key="item.label">
            <div class="plan-item">
              <span class="plan-label">{{ item.label }}</span>
              <span class="plan-value">{{ item.value }}</span>
            </div>
          </el-col>
        </el-row>
        <el-alert
          title="建议继续持有或定投，待市值覆盖本金后再操作。"
          type="warning"
          :closable="false"
          show-icon
          class="alert-tip"
        />
      </template>
    </el-card>

    <el-card v-if="isValid && !canZeroCost" shadow="never" class="tip-card">
      <template #header>
        <div class="card-header">
          <el-icon :size="20"><Help /></el-icon>
          <span>达到零成本条件</span>
        </div>
      </template>
      <div class="tip-content">
        <p>使得 <code>当前市值 ≥ 需卖出金额</code> 即可执行零成本操作。</p>
        <p><el-tag type="warning">净值需涨至 {{ breakEvenNav.toFixed(4) }} 元/份</el-tag></p>
        <p class="tip-sub">届时可一键赎回 {{ totalPrincipal.toFixed(2) }} 元本金，余仓全部为利润。</p>
      </div>
    </el-card>

    <el-empty v-if="!isValid" description="请填写完整持仓数据以开始计算" />
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import {
  EditPen, TrendCharts, Coin as CoinIcon,
  Money, DataAnalysis, SuccessFilled,
  WarningFilled, Help
} from '@element-plus/icons-vue'

const shares = ref(null)
const costPrice = ref(null)
const nav = ref(null)
const considerFee = ref(true)
const feeRate = ref(0.5)

const isValid = computed(() =>
  +shares.value > 0 && +costPrice.value > 0 && +nav.value > 0
)

const totalPrincipal = computed(() => shares.value * costPrice.value)
const marketValue = computed(() => shares.value * nav.value)
const profit = computed(() => marketValue.value - totalPrincipal.value)
const profitRate = computed(() =>
  totalPrincipal.value ? (profit.value / totalPrincipal.value) * 100 : 0
)

const feeDecimal = computed(() => considerFee.value ? feeRate.value / 100 : 0)

const targetSellCash = computed(() => {
  if (!isValid.value) return 0
  return totalPrincipal.value / (1 - feeDecimal.value)
})

const canZeroCost = computed(() => marketValue.value >= targetSellCash.value)

const sellShares = computed(() => isValid.value ? targetSellCash.value / nav.value : 0)
const remainingShares = computed(() => shares.value - sellShares.value)
const remainingValue = computed(() => remainingShares.value * nav.value)
const breakEvenNav = computed(() => totalPrincipal.value / shares.value)
const gap = computed(() => Math.max(0, targetSellCash.value - marketValue.value))

const fmt = (v, d = 2) => (v ?? 0).toFixed(d)

const stats = computed(() => [
  { label: '总投入本金', value: `¥ ${fmt(totalPrincipal.value)}`, cls: '' },
  { label: '当前市值', value: `¥ ${fmt(marketValue.value)}`, cls: '' },
  {
    label: '持仓收益',
    value: `${profit.value >= 0 ? '+' : ''}¥ ${fmt(profit.value)}`,
    cls: profit.value >= 0 ? 'up' : 'down'
  },
  {
    label: '持仓收益率',
    value: `${profitRate.value >= 0 ? '+' : ''}${fmt(profitRate.value)}%`,
    cls: profitRate.value >= 0 ? 'up' : 'down'
  }
])

const sellPlan = computed(() => [
  { label: '需卖出金额', value: `¥ ${fmt(targetSellCash.value)}` },
  { label: '对应卖出份额', value: `${fmt(sellShares.value)} 份` },
  { label: '剩余零成本份额', value: `${fmt(remainingShares.value)} 份` },
  { label: '零成本底仓市值', value: `¥ ${fmt(remainingValue.value)}` }
])

const gapInfo = computed(() => [
  { label: '当前缺口', value: `¥ ${fmt(gap.value)}` },
  { label: '需卖出金额', value: `¥ ${fmt(targetSellCash.value)}` },
  { label: '净值需涨至', value: `${fmt(breakEvenNav.value, 4)} 元/份` },
  { label: '当前净值差距', value: `${fmt((breakEvenNav.value / (1 - feeDecimal.value)) - nav.value, 4)} 元/份` }
])
</script>

<style scoped>
.calculator {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.card-header {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 600;
  font-size: 1rem;
}

.input-card :deep(.el-card__body) {
  padding-top: 8px;
}

.fee-group {
  display: flex;
  align-items: center;
  gap: 12px;
  padding-top: 2px;
}

.fee-input {
  width: 100px;
}

.fee-disabled {
  color: #9ca3af;
  font-size: 0.9rem;
}

.stat-item {
  background: #f9fafb;
  border-radius: 8px;
  padding: 12px 8px;
  text-align: center;
  border: 1px solid #e5e7eb;
  margin-bottom: 8px;
}

.stat-label {
  display: block;
  font-size: 0.75rem;
  color: #6b7280;
  margin-bottom: 4px;
}

.stat-value {
  font-size: 1.05rem;
  font-weight: 600;
}

.up .stat-value {
  color: #ef4444;
}

.down .stat-value {
  color: #10b981;
}

.action-card :deep(.el-card__header) {
  border-bottom: none;
  padding-bottom: 0;
}

.action-card.is-zero :deep(.el-card__header) {
  color: #065f46;
}

.action-card.not-zero :deep(.el-card__header) {
  color: #92400e;
}

.plan-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  background: #f9fafb;
  border-radius: 6px;
  margin-bottom: 8px;
}

.plan-label {
  font-size: 0.85rem;
  color: #6b7280;
}

.plan-value {
  font-weight: 600;
  font-size: 0.95rem;
  color: #1f2937;
}

.alert-tip {
  margin-top: 12px;
}

.tip-card :deep(.el-card__body) {
  padding-top: 4px;
}

.tip-content {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.tip-content p {
  margin: 0;
  font-size: 0.9rem;
  color: #374151;
}

.tip-sub {
  color: #9ca3af;
  font-size: 0.85rem !important;
}
</style>
