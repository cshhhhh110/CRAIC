<template>
  <a-layout-header class="frontend-navbar">
    <div class="navbar-container">
      <!-- Logo和站点名称 -->
      <div class="navbar-logo">
        <router-link to="/home">
          <img :src="siteConfig.logo.icon" alt="Logo" class="logo-icon" />
          <span class="logo-text">{{ siteConfig.logo.text }}</span>
        </router-link>
      </div>

      <!-- 导航菜单 -->
      <a-menu
        v-model:selectedKeys="selectedKeys"
        mode="horizontal"
        class="navbar-menu"
        :style="{ lineHeight: '64px', borderBottom: 'none' }"
      >
        <a-menu-item key="home">
          <router-link to="/home">
            <HomeOutlined />
            <span>首页</span>
          </router-link>
        </a-menu-item>

        <a-menu-item key="heritage">
          <router-link to="/heritage">
            <i class="fas fa-landmark"></i>
            <span>非遗作品</span>
          </router-link>
        </a-menu-item>

        <a-menu-item key="inheritor">
          <router-link to="/inheritor">
            <i class="fas fa-user-tie"></i>
            <span>传承人</span>
          </router-link>
        </a-menu-item>

        <a-menu-item key="activity">
          <router-link to="/activity">
            <i class="fas fa-calendar-alt"></i>
            <span>活动中心</span>
          </router-link>
        </a-menu-item>

        <a-menu-item key="course">
          <router-link to="/course">
            <i class="fas fa-graduation-cap"></i>
            <span>在线课程</span>
          </router-link>
        </a-menu-item>

        <a-menu-item key="shop">
          <router-link to="/shop">
            <i class="fas fa-shopping-bag"></i>
            <span>手办商城</span>
          </router-link>
        </a-menu-item>

        <a-menu-item key="ai-chat">
          <router-link to="/ai-chat">
            <i class="fas fa-robot"></i>
            <span>AI助手</span>
          </router-link>
        </a-menu-item>

        <a-menu-item key="profile" v-if="isLoggedIn">
          <router-link to="/profile">
            <UserOutlined />
            <span>个人中心</span>
          </router-link>
        </a-menu-item>
      </a-menu>

      <!-- 右侧用户区域 -->
      <div class="navbar-user">
        <template v-if="isLoggedIn">
          <!-- 用户信息下拉菜单 -->
          <a-dropdown>
            <a class="user-info" @click.prevent>
              <a-avatar :size="32" :src="userStore.avatar">
                {{ userStore.userInfo.username?.charAt(0) || 'U' }}
              </a-avatar>
              <span class="user-name">{{ userStore.userInfo.username }}</span>
              <DownOutlined />
            </a>
            <template #overlay>
              <a-menu>
                <a-menu-item key="profile">
                  <router-link to="/profile">
                    <UserOutlined />
                    <span>个人中心</span>
                  </router-link>
                </a-menu-item>
        
                <a-menu-item key="my-orders">
                  <router-link to="/orders">
                    <i class="fas fa-shopping-cart"></i>
                    <span>我的订单</span>
                  </router-link>
                </a-menu-item>
             
                <a-menu-divider />
                <a-menu-item key="logout" @click="handleLogout">
                  <LogoutOutlined />
                  <span>退出登录</span>
                </a-menu-item>
              </a-menu>
            </template>
          </a-dropdown>
        </template>

        <template v-else>
          <!-- 未登录状态 -->
          <a-space :size="16">
            <a-button type="default" @click="router.push('/auth/login')">
              登录
            </a-button>
            <a-button type="primary" @click="router.push('/auth/register')">
              注册
            </a-button>
          </a-space>
        </template>
      </div>
    </div>
  </a-layout-header>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { useUserStore } from '@/store/user'
import { Modal } from 'ant-design-vue'
import siteConfig from '@/config/site'
import {
  HomeOutlined,
  UserOutlined,
  DownOutlined,
  LogoutOutlined
} from '@ant-design/icons-vue'

const router = useRouter()
const route = useRoute()
const userStore = useUserStore()

// 当前选中的菜单项
const selectedKeys = ref(['home'])

// 是否登录
const isLoggedIn = computed(() => userStore.isLoggedIn)

// 根据路由更新选中的菜单项
watch(() => route.path, (newPath) => {
  if (newPath === '/home') {
    selectedKeys.value = ['home']
  } else if (newPath.startsWith('/heritage')) {
    selectedKeys.value = ['heritage']
  } else if (newPath.startsWith('/inheritor')) {
    selectedKeys.value = ['inheritor']
  } else if (newPath.startsWith('/activity')) {
    selectedKeys.value = ['activity']
  } else if (newPath.startsWith('/course')) {
    selectedKeys.value = ['course']
  } else if (newPath.startsWith('/shop')) {
    selectedKeys.value = ['shop']
  } else if (newPath.startsWith('/profile')) {
    selectedKeys.value = ['profile']
  } else {
    selectedKeys.value = []
  }
}, { immediate: true })

// 退出登录
const handleLogout = () => {
  Modal.confirm({
    title: '确认退出',
    content: '确定要退出登录吗？',
    okText: '确定',
    cancelText: '取消',
    onOk: async () => {
      await userStore.logout()
      router.push('/auth/login')
    }
  })
}
</script>

<style scoped>
.frontend-navbar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  background: #fff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  padding: 0;
  height: 64px;
  line-height: 64px;
}

.navbar-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 24px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 100%;
}

.navbar-logo {
  flex-shrink: 0;
}

.navbar-logo a {
  display: flex;
  align-items: center;
  text-decoration: none;
  color: inherit;
}

.logo-icon {
  width: 32px;
  height: 32px;
  margin-right: 8px;
  object-fit: contain;
}

.logo-text {
  font-size: 20px;
  font-weight: 600;
  color: #1890ff;
}

.navbar-menu {
  flex: 1;
  border: none;
  background: transparent;
  margin: 0 24px;
}



.navbar-user {
  flex-shrink: 0;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 4px 12px;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
  color: rgba(0, 0, 0, 0.85);
  text-decoration: none;
}

.user-info:hover {
  background-color: rgba(0, 0, 0, 0.04);
}

.user-name {
  max-width: 120px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .navbar-container {
    padding: 0 16px;
  }

  .navbar-menu {
    display: none;
  }

  .logo-text {
    font-size: 16px;
  }

  .user-name {
    display: none;
  }
}
</style>
