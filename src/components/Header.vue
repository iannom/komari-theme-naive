<script setup lang="ts">
import { NAvatar, NButton, NFlex, NH3, NPopover } from 'naive-ui'
import { computed, h, inject, ref } from 'vue'
import { useRouter } from 'vue-router'
import { useAppStore } from '@/stores/app'
import HeaderStats from './HeaderStats.vue'
import LoginDialog from './LoginDialog.vue'

const router = useRouter()
const appStore = useAppStore()

// 从 Provider 注入滚动状态
const isScrolled = inject<ReturnType<typeof ref<boolean>>>('isScrolled', ref(false))

const siteFavicon = ref('/favicon.ico')

// 计算页面容器的样式
const containerStyle = computed(() => {
  if (appStore.fullWidth) {
    return {}
  }
  return {
    maxWidth: appStore.maxPageWidth,
    marginInline: 'auto',
  }
})

const actionButtons = computed(() => {
  const buttons = [
    {
      title: appStore.themeMode === 'auto' ? '自动主题' : appStore.themeMode === 'light' ? '浅色主题' : '深色主题',
      icon: appStore.themeMode === 'auto' ? 'i-icon-park-outline-dark-mode' : appStore.themeMode === 'light' ? 'i-icon-park-outline-sun-one' : 'i-icon-park-outline-moon',
      action: 'toggleTheme',
      disabled: false,
    },
  ]

  // 已登录时显示设置按钮，未登录时根据配置决定是否显示登录按钮
  if (appStore.isLoggedIn) {
    buttons.push({
      title: '后台管理',
      icon: 'i-icon-park-outline-setting',
      action: 'jumpToSetting',
      disabled: false,
    })
  }
  else if (appStore.showLoginButton) {
    buttons.push({
      title: '登录',
      icon: 'i-icon-park-outline-login',
      action: 'openLoginDialog',
      disabled: false,
    })
  }

  return buttons
})

function handleButtonClick(action: string) {
  switch (action) {
    case 'toggleTheme':
      appStore.updateThemeMode()
      break
    case 'jumpToSetting':
      // 设置页由 Server 提供，不能使用无极路由
      location.href = '/admin'
      break
    case 'openLoginDialog':
      window.$modal.create({
        title: '登录',
        preset: 'dialog',
        showIcon: false,
        content: () => h(LoginDialog),
      })
      break
  }
}
</script>

<template>
  <div class="transition-all duration-200 top-0 position-sticky z-10" :class="isScrolled ? 'bg-$n-color shadow-sm backdrop-blur-md' : 'bg-transparent'">
    <div class="header-container px-4 h-16" :style="containerStyle">
      <NFlex class="header-brand flex-center cursor-pointer" @click="router.push('/')">
        <NAvatar :src="siteFavicon" round />
        <NH3 class="m-0 truncate">
          {{ appStore.publicSettings?.sitename || 'Komari Monitor' }}
        </NH3>
      </NFlex>
      <HeaderStats v-if="appStore.showHeaderStats" class="header-stats-panel" />
      <NFlex class="header-actions flex gap-4">
        <NPopover v-for="button in actionButtons" :key="button.action" :disabled="button.disabled">
          <template #trigger>
            <NButton :disabled="button.disabled" class="p-2 h-8 w-8" text @click="handleButtonClick(button.action)">
              <div :class="button.icon" />
            </NButton>
          </template>
          <template #default>
            {{ button.title }}
          </template>
        </NPopover>
      </NFlex>
    </div>
  </div>
</template>

<style scoped lang="scss">
.header-container {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(0, 2fr) minmax(max-content, 1fr);
  gap: 20px;
  align-items: center;
}

.header-brand {
  min-width: 0;
  justify-self: start;
}

.header-actions {
  justify-self: end;
}

.header-stats-panel {
  min-width: 0;
}

@media (max-width: 960px) {
  .header-container {
    grid-template-columns: minmax(0, 1fr) max-content;
  }

  .header-stats-panel {
    display: none;
  }
}
</style>
