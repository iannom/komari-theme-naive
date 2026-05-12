<script setup lang="ts">
import { useNow } from '@vueuse/core'
import { computed } from 'vue'
import { useAppStore } from '@/stores/app'
import { useNodesStore } from '@/stores/nodes'
import { formatBytesPerSecondWithConfig, formatBytesWithConfig } from '@/utils/helper'

const appStore = useAppStore()
const nodesStore = useNodesStore()

const now = useNow({ interval: 1000 })
const currentTime = computed(() => {
  return now.value.toLocaleTimeString('zh-CN', { hour12: false })
})

const onlineNodes = computed(() => nodesStore.nodes.filter(node => node.online))

const onlineNodeCount = computed(() => onlineNodes.value.length)

const onlineRegionCount = computed(() => {
  return new Set(
    onlineNodes.value
      .filter(node => node.region !== '')
      .map(node => node.region),
  ).size
})

const totalTraffic = computed(() => {
  return nodesStore.nodes.reduce(
    (total, node) => ({
      up: total.up + (node.net_total_up || 0),
      down: total.down + (node.net_total_down || 0),
    }),
    { up: 0, down: 0 },
  )
})

const totalSpeed = computed(() => {
  return onlineNodes.value.reduce(
    (total, node) => ({
      up: total.up + (node.net_out || 0),
      down: total.down + (node.net_in || 0),
    }),
    { up: 0, down: 0 },
  )
})

const stats = computed(() => [
  {
    label: '当前时间',
    lines: [currentTime.value, ''],
  },
  {
    label: '当前在线',
    lines: [`${onlineNodeCount.value} / ${nodesStore.nodes.length}`, ''],
  },
  {
    label: '点亮地区',
    lines: [String(onlineRegionCount.value), ''],
  },
  {
    label: '流量',
    lines: [
      `↑ ${formatBytesWithConfig(totalTraffic.value.up, appStore.byteDecimals)}`,
      `↓ ${formatBytesWithConfig(totalTraffic.value.down, appStore.byteDecimals)}`,
    ],
  },
  {
    label: '网速',
    lines: [
      `↑ ${formatBytesPerSecondWithConfig(totalSpeed.value.up, appStore.byteDecimals)}`,
      `↓ ${formatBytesPerSecondWithConfig(totalSpeed.value.down, appStore.byteDecimals)}`,
    ],
  },
])
</script>

<template>
  <div class="header-stats" :style="{ fontFamily: appStore.numberFontFamily }">
    <div v-for="item in stats" :key="item.label" class="header-stat-item">
      <span class="header-stat-label">{{ item.label }}</span>
      <span class="header-stat-values">
        <span
          v-for="(line, index) in item.lines"
          :key="`${item.label}-${index}`"
          class="header-stat-value"
          :class="{ 'header-stat-value--empty': !line }"
        >
          {{ line || '-' }}
        </span>
      </span>
    </div>
  </div>
</template>

<style scoped lang="scss">
.header-stats {
  display: flex;
  min-width: 0;
  flex: 1;
  gap: 20px;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  color: var(--n-text-color);
  font-size: 13px;
  line-height: 1;
  white-space: nowrap;
}

.header-stat-item {
  display: flex;
  height: 32px;
  min-width: 0;
  gap: 7px;
  align-items: flex-start;
  font-weight: 700;
}

.header-stat-label {
  flex-shrink: 0;
  line-height: 14px;
}

.header-stat-values {
  display: flex;
  min-width: 0;
  flex-direction: column;
  gap: 4px;
}

.header-stat-value {
  display: block;
  overflow: hidden;
  max-width: 120px;
  min-height: 14px;
  line-height: 14px;
  text-overflow: ellipsis;
}

.header-stat-value--empty {
  visibility: hidden;
}

@media (max-width: 1180px) {
  .header-stats {
    gap: 12px;
    font-size: 12px;
  }

  .header-stat-value {
    max-width: 92px;
  }
}
</style>
