<template>
  <div class="flex flex-col gap-4">
    <!-- 输入区域 -->
    <div class="bg-white rounded-xl shadow-sm border border-gray-200 overflow-hidden">
      <div class="flex items-center gap-2 px-5 py-4 border-b border-gray-100 font-semibold text-gray-800">
        <svg class="w-5 h-5 text-emerald-600" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
          <path stroke-linecap="round" stroke-linejoin="round" d="M16.862 4.487l1.687-1.688a1.875 1.875 0 112.652 2.652L10.582 16.07a4.5 4.5 0 01-1.897 1.13L6 18l.8-2.685a4.5 4.5 0 011.13-1.897l8.932-8.931zm0 0L19.5 7.125M18 14v4.75A2.25 2.25 0 0115.75 21H5.25A2.25 2.25 0 013 18.75V8.25A2.25 2.25 0 015.25 6H10" />
        </svg>
        <span>持仓数据录入</span>
      </div>
      <div class="px-5 py-4 space-y-4">
        <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
          <div>
            <label class="block text-sm font-medium text-gray-600 mb-1.5">持有份额（份）</label>
            <input
              v-model.number="shares"
              type="number"
              step="0.01"
              min="0"
              placeholder="例：1000.50"
              class="w-full px-3 py-2.5 border border-gray-300 rounded-lg text-sm focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition"
            />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-600 mb-1.5">持仓成本价（元/份）</label>
            <input
              v-model.number="costPrice"
              type="number"
              step="0.0001"
              min="0"
              placeholder="例：1.2500"
              class="w-full px-3 py-2.5 border border-gray-300 rounded-lg text-sm focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition"
            />
          </div>
        </div>
        <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
          <div>
            <label class="block text-sm font-medium text-gray-600 mb-1.5">当前单位净值（元/份）</label>
            <input
              v-model.number="nav"
              type="number"
              step="0.0001"
              min="0"
              placeholder="例：1.4800"
              class="w-full px-3 py-2.5 border border-gray-300 rounded-lg text-sm focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition"
            />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-600 mb-1.5">赎回费率</label>
            <div class="relative w-28">
              <input
                v-model.number="feeRate"
                type="number"
                step="0.01"
                min="0"
                max="100"
                placeholder="0.50"
                class="w-full pl-3 pr-7 py-2.5 border border-gray-300 rounded-lg text-sm focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition"
              />
              <span class="absolute right-2.5 top-1/2 -translate-y-1/2 text-gray-400 text-sm">%</span>
            </div>
            <p class="mt-1.5 text-xs text-gray-400">不填或填 0 表示不计费率</p>
          </div>
        </div>
      </div>
    </div>

    <!-- 持仓概览 -->
    <div v-if="isValid" class="bg-white rounded-xl shadow-sm border border-gray-200 overflow-hidden">
      <div class="flex items-center gap-2 px-5 py-4 border-b border-gray-100 font-semibold text-gray-800">
        <svg class="w-5 h-5 text-emerald-600" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
          <path stroke-linecap="round" stroke-linejoin="round" d="M3 13.125C3 12.504 3.504 12 4.125 12h2.25c.621 0 1.125.504 1.125 1.125v6.75C7.5 20.496 6.996 21 6.375 21h-2.25A1.125 1.125 0 013 19.875v-6.75zM9.75 8.625c0-.621.504-1.125 1.125-1.125h2.25c.621 0 1.125.504 1.125 1.125v11.25c0 .621-.504 1.125-1.125 1.125h-2.25a1.125 1.125 0 01-1.125-1.125V8.625zM16.5 4.125c0-.621.504-1.125 1.125-1.125h2.25C20.496 3 21 3.504 21 4.125v15.75c0 .621-.504 1.125-1.125 1.125h-2.25a1.125 1.125 0 01-1.125-1.125V4.125z" />
        </svg>
        <span>持仓概览</span>
      </div>
      <div class="px-5 py-4">
        <div class="grid grid-cols-2 sm:grid-cols-4 gap-3">
          <div v-for="s in stats" :key="s.label" class="bg-gray-50 rounded-lg p-3 text-center border border-gray-100">
            <span class="block text-xs text-gray-400 mb-1">{{ s.label }}</span>
            <span :class="['text-base font-semibold', s.cls]">{{ s.value }}</span>
          </div>
        </div>
      </div>
    </div>

    <!-- 零成本结果 -->
    <div v-if="isValid" :class="['rounded-xl shadow-sm border overflow-hidden', canZeroCost ? 'bg-emerald-50 border-emerald-300' : 'bg-amber-50 border-amber-300']">
      <div :class="['flex items-center gap-2 px-5 py-4 border-b font-semibold', canZeroCost ? 'border-emerald-200 text-emerald-800' : 'border-amber-200 text-amber-800']">
        <svg v-if="canZeroCost" class="w-5 h-5 text-emerald-600" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
          <path stroke-linecap="round" stroke-linejoin="round" d="M9 12.75L11.25 15 15 9.75M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
        <svg v-else class="w-5 h-5 text-amber-600" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
          <path stroke-linecap="round" stroke-linejoin="round" d="M12 9v3.75m-9.303 3.376c-.866 1.5.217 3.374 1.948 3.374h14.71c1.73 0 2.813-1.874 1.948-3.374L13.949 3.378c-.866-1.5-3.032-1.5-3.898 0L2.697 16.126zM12 15.75h.007v.008H12v-.008z" />
        </svg>
        <span>{{ canZeroCost ? '已满足零成本条件' : '尚未达到零成本条件' }}</span>
      </div>
      <div class="px-5 py-4">
        <template v-if="canZeroCost">
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-2">
            <div v-for="item in sellPlan" :key="item.label" class="flex justify-between items-center bg-white/70 rounded-lg px-3.5 py-2.5">
              <span class="text-sm text-gray-500">{{ item.label }}</span>
              <span class="text-sm font-semibold text-gray-800">{{ item.value }}</span>
            </div>
          </div>
          <div class="mt-3 flex gap-2 items-start bg-emerald-100/80 rounded-lg px-3.5 py-2.5 text-sm text-emerald-800">
            <svg class="w-4 h-4 mt-0.5 shrink-0" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.857-9.809a.75.75 0 00-1.214-.882l-3.483 4.79-1.88-1.88a.75.75 0 10-1.06 1.061l2.5 2.5a.75.75 0 001.137-.089l4-5.5z" clip-rule="evenodd"/></svg>
            <span>赎回上述金额后，剩余份额即为零成本底仓，后续涨跌均为纯利润。</span>
          </div>
        </template>
        <template v-else>
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-2">
            <div v-for="item in gapInfo" :key="item.label" class="flex justify-between items-center bg-white/70 rounded-lg px-3.5 py-2.5">
              <span class="text-sm text-gray-500">{{ item.label }}</span>
              <span class="text-sm font-semibold text-gray-800">{{ item.value }}</span>
            </div>
          </div>
          <div class="mt-3 flex gap-2 items-start bg-amber-100/80 rounded-lg px-3.5 py-2.5 text-sm text-amber-800">
            <svg class="w-4 h-4 mt-0.5 shrink-0" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M8.485 2.495c.673-1.167 2.357-1.167 3.03 0l6.28 10.875c.673 1.167-.17 2.625-1.516 2.625H3.72c-1.347 0-2.189-1.458-1.515-2.625L8.485 2.495zM10 5a.75.75 0 01.75.75v3.5a.75.75 0 01-1.5 0v-3.5A.75.75 0 0110 5zm0 9a1 1 0 100-2 1 1 0 000 2z" clip-rule="evenodd"/></svg>
            <span>建议继续持有或定投，待市值覆盖本金后再操作。</span>
          </div>
        </template>
      </div>
    </div>

    <!-- 达标提示 -->
    <div v-if="isValid && !canZeroCost" class="bg-white rounded-xl shadow-sm border border-gray-200 overflow-hidden">
      <div class="flex items-center gap-2 px-5 py-4 border-b border-gray-100 font-semibold text-gray-800">
        <svg class="w-5 h-5 text-emerald-600" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
          <path stroke-linecap="round" stroke-linejoin="round" d="M9.879 7.519c1.171-1.025 3.071-1.025 4.242 0 1.172 1.025 1.172 2.687 0 3.712-.203.179-.43.326-.67.442-.745.361-1.45.999-1.45 1.827v.75M21 12a9 9 0 11-18 0 9 9 0 0118 0zm-9 5.25h.008v.008H12v-.008z" />
        </svg>
        <span>达到零成本条件</span>
      </div>
      <div class="px-5 py-4 space-y-2 text-sm text-gray-600">
        <p>使得 <code class="bg-gray-100 px-1.5 py-0.5 rounded text-emerald-700 font-mono text-xs">当前市值 ≥ 需卖出金额</code> 即可执行零成本操作。</p>
        <p>
          <span class="inline-block bg-amber-100 text-amber-800 text-xs font-medium px-2.5 py-1 rounded-full">净值需涨至 {{ breakEvenNav.toFixed(4) }} 元/份</span>
        </p>
        <p class="text-gray-400 text-xs">届时可一键赎回 {{ totalPrincipal.toFixed(2) }} 元本金，余仓全部为利润。</p>
      </div>
    </div>

    <!-- 空状态 -->
    <div v-if="!isValid" class="text-center py-16 text-gray-400 italic text-sm">
      请填写完整持仓数据以开始计算
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const shares = ref(null)
const costPrice = ref(null)
const nav = ref(null)
const feeRate = ref(null)

const isValid = computed(() =>
  +shares.value > 0 && +costPrice.value > 0 && +nav.value > 0
)

const totalPrincipal = computed(() => shares.value * costPrice.value)
const marketValue = computed(() => shares.value * nav.value)
const profit = computed(() => marketValue.value - totalPrincipal.value)
const profitRate = computed(() =>
  totalPrincipal.value ? (profit.value / totalPrincipal.value) * 100 : 0
)

const feeDecimal = computed(() => (feeRate.value ?? 0) / 100)

const targetSellCash = computed(() => {
  if (!isValid.value) return 0
  return totalPrincipal.value / (1 - feeDecimal.value)
})

const canZeroCost = computed(() => marketValue.value >= targetSellCash.value)

const sellShares = computed(() => isValid.value ? targetSellCash.value / nav.value : 0)
const remainingShares = computed(() => shares.value - sellShares.value)
const remainingValue = computed(() => remainingShares.value * nav.value)
const breakEvenNav = computed(() => {
  if (!isValid.value) return 0
  return totalPrincipal.value / (shares.value * (1 - feeDecimal.value))
})
const sellFee = computed(() => targetSellCash.value * feeDecimal.value)
const gap = computed(() => Math.max(0, targetSellCash.value - marketValue.value))

const fmt = (v, d = 2) => (v ?? 0).toFixed(d)

const stats = computed(() => [
  { label: '总投入本金', value: `¥ ${fmt(totalPrincipal.value)}`, cls: '' },
  { label: '当前市值', value: `¥ ${fmt(marketValue.value)}`, cls: '' },
  {
    label: '持仓收益',
    value: `${profit.value >= 0 ? '+' : ''}¥ ${fmt(profit.value)}`,
    cls: profit.value >= 0 ? 'text-red-500' : 'text-emerald-500'
  },
  {
    label: '持仓收益率',
    value: `${profitRate.value >= 0 ? '+' : ''}${fmt(profitRate.value)}%`,
    cls: profitRate.value >= 0 ? 'text-red-500' : 'text-emerald-500'
  }
])

const sellPlan = computed(() => [
  { label: '需卖出金额', value: `¥ ${fmt(targetSellCash.value)}` },
  { label: '预估赎回费', value: `¥ ${fmt(sellFee.value)}` },
  { label: '对应卖出份额', value: `${fmt(sellShares.value)} 份` },
  { label: '剩余零成本份额', value: `${fmt(remainingShares.value)} 份` },
  { label: '零成本底仓市值', value: `¥ ${fmt(remainingValue.value)}` }
])

const gapInfo = computed(() => [
  { label: '当前缺口', value: `¥ ${fmt(gap.value)}` },
  { label: '需卖出金额', value: `¥ ${fmt(targetSellCash.value)}` },
  { label: '净值需涨至', value: `${fmt(breakEvenNav.value, 4)} 元/份` },
  { label: '当前净值差距', value: `${fmt(breakEvenNav.value - nav.value, 4)} 元/份` }
])
</script>
