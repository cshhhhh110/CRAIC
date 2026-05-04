<template>
  <div class="sidebar-container" :class="{ 'is-collapsed': isCollapsed }">
    <div class="logo">
      <img :src="siteConfig.admin.logo.icon" alt="Logo" class="logo-icon" />
      <span class="logo-text" v-show="!isCollapsed">{{ siteConfig.admin.logo.text }}</span>
    </div>
    <div class="menu-wrapper">
      <a-menu 
        v-model:selectedKeys="selectedKeys" 
        :inline-collapsed="isCollapsed" 
        mode="inline" 
        class="sidebar-menu"
       
      >
        <!-- 固定菜单项 -->
        <a-menu-item key="/back/dashboard" @click="handleMenuClick('/back/dashboard')">
          <template #icon>
            <i class="fas fa-house"></i>
          </template>
          <span>首页</span>
        </a-menu-item>
        
        <a-menu-item key="/back/user" @click="handleMenuClick('/back/user')">
          <template #icon>
            <i class="fas fa-user"></i>
          </template>
          <span>用户管理</span>
        </a-menu-item>
        
        <a-menu-item key="/back/heritage" @click="handleMenuClick('/back/heritage')">
          <template #icon>
            <i class="fas fa-landmark"></i>
          </template>
          <span>非遗作品管理</span>
        </a-menu-item>
        
        <a-menu-item key="/back/inheritor" @click="handleMenuClick('/back/inheritor')">
          <template #icon>
            <i class="fas fa-user-tie"></i>
          </template>
          <span>传承人管理</span>
        </a-menu-item>
        
        <a-menu-item key="/back/activity" @click="handleMenuClick('/back/activity')">
          <template #icon>
            <i class="fas fa-calendar-alt"></i>
          </template>
          <span>活动管理</span>
        </a-menu-item>
        
        <a-menu-item key="/back/course" @click="handleMenuClick('/back/course')">
          <template #icon>
            <i class="fas fa-graduation-cap"></i>
          </template>
          <span>课程管理</span>
        </a-menu-item>
        
        <!-- 商城管理子菜单 -->
        <a-sub-menu key="shop">
          <template #icon>
            <i class="fas fa-store"></i>
          </template>
          <template #title>商城管理</template>
          
          <a-menu-item key="/back/shop/category" @click="handleMenuClick('/back/shop/category')">
            <template #icon>
              <i class="fas fa-tags"></i>
            </template>
            <span>商品分类</span>
          </a-menu-item>
          
          <a-menu-item key="/back/shop/product" @click="handleMenuClick('/back/shop/product')">
            <template #icon>
              <i class="fas fa-box"></i>
            </template>
            <span>商品管理</span>
          </a-menu-item>
          
          <a-menu-item key="/back/shop/orders" @click="handleMenuClick('/back/shop/orders')">
            <template #icon>
              <i class="fas fa-shopping-cart"></i>
            </template>
            <span>订单管理</span>
          </a-menu-item>
        </a-sub-menu>
        
        <a-menu-item key="/back/profile" @click="handleMenuClick('/back/profile')">
          <template #icon>
            <i class="fas fa-user"></i>
          </template>
          <span>个人信息</span>
        </a-menu-item>
      </a-menu>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useAppStore } from '@/store/app'
import siteConfig from '@/config/site'

const route = useRoute()
const router = useRouter()
const appStore = useAppStore()

const isCollapsed = computed(() => appStore.sidebarCollapsed)

// 当前激活的菜单
const selectedKeys = ref([route.path])

// 监听路由变化更新选中的菜单
watch(() => route.path, (newPath) => {
  selectedKeys.value = [newPath]
})

// 处理菜单点击
const handleMenuClick = (path) => {
  router.push(path)
}
</script>

<style lang="scss" scoped>
.sidebar-container {
  height: 100%; 
  min-height: 100vh;
  background:rgb(255, 255, 255);
  display: flex;
  flex-direction: column;
  width: 220px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  
  &.is-collapsed {
    width: 64px;
    
    .logo {
      padding: 0;
      justify-content: center;
      
      .logo-icon {
        margin: 0;
      }
    }

    :deep(.ant-menu) {
      // 只隐藏文字，不隐藏图标
      .ant-menu-submenu-title > span:not(.anticon),
      .ant-menu-item > span:not(.anticon) {
        opacity: 0;
        transition: opacity 0.2s;
      }
    }
  }
  
  .logo {
    height: 60px;
    flex-shrink: 0;
    line-height: 60px;
    text-align: center;
    background: rgba(255, 255, 255, 0.05);
    backdrop-filter: blur(10px);
    border-bottom: 1px solid rgba(255, 255, 255, 0.05);
    display: flex;
    align-items: center;
    padding: 0 16px;
    overflow: hidden;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    
    .logo-icon {
      width: 32px;
      height: 32px;
      margin-right: 8px;
      object-fit: contain;
      transition: margin 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    }
    
    .logo-text {
      color:rgb(0, 0, 0);
      font-size: 18px;
      font-weight: 600;
      white-space: nowrap;
      opacity: 1;
      transition: opacity 0.2s;
    }
  }

  .menu-wrapper {
    flex: 1;
    overflow-y: auto;
    overflow-x: hidden;
    width: 100%;

    &::-webkit-scrollbar {
      width: 6px;
    }

    &::-webkit-scrollbar-thumb {
      background: rgba(0, 0, 0, 0.2);
      border-radius: 3px;
    }

    &::-webkit-scrollbar-track {
      background: transparent;
    }
  }

  :deep(.sidebar-menu) {
    border: none;
    width: 100% !important;
  }
}
</style> 