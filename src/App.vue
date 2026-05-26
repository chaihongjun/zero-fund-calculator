<template>
  <div class="min-h-screen flex flex-col bg-gradient-to-br from-sky-50 via-cyan-50 to-emerald-50">
    <header class="text-center pt-8 pb-4 px-4">
      <h1 class="flex items-center justify-center gap-2 text-2xl font-bold text-emerald-800 sm:text-3xl">
        <span class="text-3xl">📊</span>
        基金零成本计算器
      </h1>
      <p class="text-gray-400 mt-1 text-sm sm:text-base">输入当前持仓快照，一键测算零成本卖出方案</p>
    </header>

    <main class="flex-1 px-4 pb-8">
      <div class="max-w-2xl mx-auto">
        <ZeroFundCalculator />
      </div>
    </main>

    <footer class="text-center py-4 text-gray-400 text-xs">
      零成本策略：卖出部分份额收回全部本金，剩余份额即为零成本底仓
    </footer>
  </div>
</template>

<script setup>
import ZeroFundCalculator from './components/ZeroFundCalculator.vue'
</script>
