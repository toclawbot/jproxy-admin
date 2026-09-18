<template>
  <div class="navbar">
    <div class="left-side">
      <a-space>
        <img
          alt="logo"
          src="@/assets/images/logo-128x128.png"
          style="width: 33px"
        />
        <a-typography-title
          :style="{
            margin: '0 0 0 10px',
            fontSize: '20px',
            fontWeight: 600,
            letterSpacing: '2px',
          }"
          :heading="5"
        >
          JProxy
        </a-typography-title>
        <a-link
          style="margin-top: 15px; font-size: 13px; color: var(--color-text-1)"
          href="https://github.com/LuckyPuppy514/jproxy/blob/main/changelog.md"
          target="_blank"
        >
          v{{ version }}
        </a-link>
        <icon-menu-fold
          v-if="!topMenu && appStore.device === 'mobile'"
          style="font-size: 22px; cursor: pointer"
          @click="toggleDrawerMenu"
        />
      </a-space>
    </div>
    <div class="center-side">
      <Menu v-if="topMenu" />
    </div>
    <div class="right-side">
      <!-- 主要操作按钮：始终显示 -->
      <div class="nav-primary-actions">
        <div class="nav-item nav-dropdown">
          <a-tooltip :content="$t('navbar.language')" :disabled="appStore.device === 'mobile'">
            <a-button
              class="nav-btn"
              type="outline"
              :shape="'circle'"
              @click="setDropDownVisible"
            >
              <template #icon>
                <icon-language />
              </template>
            </a-button>
          </a-tooltip>
          <a-dropdown trigger="click" @select="changeLocale as any">
            <div ref="triggerBtn" class="trigger-btn"></div>
            <template #content>
              <a-doption
                v-for="item in locales"
                :key="item.value"
                :value="item.value"
              >
                <template #icon>
                  <icon-check v-show="item.value === currentLocale" />
                </template>
                {{ item.label }}
              </a-doption>
            </template>
          </a-dropdown>
        </div>
        <div class="nav-item">
          <a-tooltip
            :content="theme === 'light' ? $t('navbar.dark') : $t('navbar.light')"
            :disabled="appStore.device === 'mobile'"
          >
            <a-button
              class="nav-btn"
              type="outline"
              :shape="'circle'"
              @click="handleToggleTheme"
            >
              <template #icon>
                <icon-moon-fill v-if="theme === 'dark'" />
                <icon-sun-fill v-else />
              </template>
            </a-button>
          </a-tooltip>
        </div>
        <div class="nav-item nav-dropdown">
          <a-dropdown :trigger="appStore.device === 'mobile' ? 'click' : 'hover'">
            <a-avatar
              :size="30"
              :style="{
                marginRight: '8px',
                cursor: 'pointer',
                backgroundColor: '#3370ff',
              }"
            >
              <IconUser />
            </a-avatar>
            <template #content>
              <a-doption>
                <a-space @click="$router.push({ name: 'systemUser' })">
                  <icon-settings />
                  <span>
                    {{ $t('navbar.user') }}
                  </span>
                </a-space>
              </a-doption>
              <a-doption>
                <a-space @click="handleLogout">
                  <icon-export />
                  <span>
                    {{ $t('navbar.logout') }}
                  </span>
                </a-space>
              </a-doption>
            </template>
          </a-dropdown>
        </div>
      </div>
      
      <!-- 次要操作：移动端折叠到更多菜单 -->
      <div class="nav-secondary-actions" v-if="!isMobile">
        <div class="nav-item">
          <a-tooltip :content="$t('navbar.github')">
            <a-button
              href="https://github.com/LuckyPuppy514/jproxy"
              target="_blank"
              class="nav-btn"
              type="outline"
              :shape="'circle'"
            >
              <icon-github />
            </a-button>
          </a-tooltip>
        </div>
        <div class="nav-item">
          <a-tooltip :content="$t('navbar.issue')">
            <a-button
              href="https://github.com/LuckyPuppy514/jproxy/issues/new/choose"
              target="_blank"
              class="nav-btn"
              type="outline"
              :shape="'circle'"
            >
              <icon-bug />
            </a-button>
          </a-tooltip>
        </div>
      </div>
      
      <!-- 移动端更多菜单 -->
      <div class="nav-mobile-more" v-else>
        <a-dropdown trigger="click" placement="bottom-end">
          <a-button class="nav-btn" type="outline" :shape="'circle'">
            <template #icon>
              <icon-more />
            </template>
          </a-button>
          <template #content>
            <a-doption>
              <a-space @click="openGithub">
                <icon-github />
                <span>{{ $t('navbar.github') }}</span>
              </a-space>
            </a-doption>
            <a-doption>
              <a-space @click="openIssue">
                <icon-bug />
                <span>{{ $t('navbar.issue') }}</span>
              </a-space>
            </a-doption>
          </template>
        </a-dropdown>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
  import { computed, ref, inject, onMounted, onUnmounted } from 'vue';
  import { useDark, useToggle } from '@vueuse/core';
  import { useAppStore } from '@/store';
  import { LOCALE_OPTIONS } from '@/locale';
  import useLocale from '@/hooks/locale';
  import useUser from '@/hooks/user';
  import Menu from '@/components/menu/index.vue';
  import { systemConfigVersion } from '@/api/system';

  const version = ref('3.0.0');
  systemConfigVersion().then((res) => {
    version.value = res.data;
  });
  const appStore = useAppStore();
  const { logout } = useUser();
  const { changeLocale, currentLocale } = useLocale();
  const locales = [...LOCALE_OPTIONS];
  const theme = computed(() => {
    return appStore.theme;
  });
  const topMenu = computed(() => appStore.topMenu && appStore.menu);
  const isDark = useDark({
    selector: 'body',
    attribute: 'arco-theme',
    valueDark: 'dark',
    valueLight: 'light',
    storageKey: 'arco-theme',
    onChanged(dark: boolean) {
      appStore.toggleTheme(dark);
    },
  });
  const toggleTheme = useToggle(isDark);
  const handleToggleTheme = () => {
    toggleTheme();
  };
  const triggerBtn = ref();
  const handleLogout = () => {
    logout();
  };
  const setDropDownVisible = () => {
    const event = new MouseEvent('click', {
      view: window,
      bubbles: true,
      cancelable: true,
    });
    triggerBtn.value.dispatchEvent(event);
  };
  const toggleDrawerMenu = inject('toggleDrawerMenu') as () => void;
  
  // 移动端检测
  const isMobile = ref(false);
  const checkMobile = () => {
    isMobile.value = window.innerWidth < 768;
  };
  onMounted(() => {
    checkMobile();
    window.addEventListener('resize', checkMobile);
  });
  onUnmounted(() => {
    window.removeEventListener('resize', checkMobile);
  });
  
  const openGithub = () => {
    window.open('https://github.com/LuckyPuppy514/jproxy', '_blank');
  };
  const openIssue = () => {
    window.open('https://github.com/LuckyPuppy514/jproxy/issues/new/choose', '_blank');
  };
</script>

<style scoped lang="less">
  .navbar {
    display: flex;
    justify-content: space-between;
    height: 100%;
    background-color: var(--color-bg-2);
    border-bottom: 1px solid var(--color-border);
  }

  .left-side {
    display: flex;
    align-items: center;
    padding-left: 20px;
    min-width: 0;
  }

  .center-side {
    flex: 1;
    min-width: 0;
  }

  .right-side {
    display: flex;
    align-items: center;
    padding-right: 20px;
    list-style: none;
    gap: 8px;
    min-width: 0;
    
    :deep(.locale-select) {
      border-radius: 20px;
    }
    
    .nav-item {
      display: flex;
      align-items: center;
    }

    a {
      color: var(--color-text-1);
      text-decoration: none;
    }
    .nav-btn {
      border-color: rgb(var(--gray-2));
      color: rgb(var(--gray-8));
      font-size: 16px;
      min-width: 40px;
      height: 40px;
    }
    .nav-btn:hover {
      color: rgb(var(--primary-5));
      background-color: transparent;
      border-color: rgb(var(--primary-5));
    }
    .trigger-btn,
    .ref-btn {
      position: absolute;
      bottom: 14px;
    }
    .trigger-btn {
      margin-left: 14px;
    }
  }
  
  .nav-primary-actions {
    display: flex;
    align-items: center;
    gap: 8px;
  }
  
  .nav-secondary-actions {
    display: flex;
    align-items: center;
    gap: 8px;
  }
  
  .nav-mobile-more {
    display: none;
  }

  /* 移动端优化 */
  @media (max-width: 768px) {
    .navbar {
      padding: 0 8px;
    }
    
    .left-side {
      padding-left: 8px;
    }
    
    .right-side {
      padding-right: 8px;
      gap: 4px;
    }
    
    .nav-item {
      padding: 0;
    }
    
    .nav-btn {
      min-width: 36px;
      height: 36px;
      font-size: 14px;
    }
    
    .a-avatar {
      width: 28px !important;
      height: 28px !important;
    }
    
    /* 隐藏版本号以节省空间 */
    .left-side a-link {
      display: none;
    }
    
    /* 次要操作隐藏，改为更多菜单 */
    .nav-secondary-actions {
      display: none;
    }
    
    .nav-mobile-more {
      display: flex;
    }
    
    .nav-primary-actions {
      gap: 4px;
    }
  }
  
  @media (max-width: 480px) {
    .navbar {
      padding: 0 4px;
    }
    
    .left-side {
      padding-left: 4px;
    }
    
    .right-side {
      padding-right: 4px;
      gap: 2px;
    }
    
    .nav-btn {
      min-width: 32px;
      height: 32px;
      font-size: 13px;
    }
    
    .a-avatar {
      width: 26px !important;
      height: 26px !important;
    }
    
    :deep(.arco-dropdown) {
      :deep(.arco-dropdown-menu) {
        min-width: 160px;
      }
    }
  }
</style>

<style lang="less">
  .message-popover {
    .arco-popover-content {
      margin-top: 0;
    }
  }
</style>
