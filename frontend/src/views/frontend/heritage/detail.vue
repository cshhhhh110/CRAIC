<template>
  <div class="heritage-detail-page">
    <!-- 加载状态 -->
    <div v-if="loading" class="loading-container">
      <a-skeleton :rows="10" active />
    </div>

    <!-- 作品详情 -->
    <div v-else-if="currentItem" class="detail-container">
      <!-- 页面头部 -->
      <div class="detail-header">
        <div class="header-content">
          <div class="header-left">
            <div class="header-breadcrumb">
              <a-breadcrumb>
                <a-breadcrumb-item href="/">首页</a-breadcrumb-item>
                <a-breadcrumb-item href="/heritage">非遗作品</a-breadcrumb-item>
                <a-breadcrumb-item>{{ currentItem.title }}</a-breadcrumb-item>
              </a-breadcrumb>
            </div>
            
            <div class="header-title-section">
              <h1 class="detail-title">{{ currentItem.title }}</h1>
              <div class="header-meta">
                <a-tag class="category-tag">{{ currentItem.category }}</a-tag>
                <span v-if="currentItem.region" class="region-tag">
                  <i class="fas fa-map-marker-alt"></i>
                  {{ currentItem.region }}
                </span>
                <span class="publish-time">
                  <i class="fas fa-clock"></i>
                  {{ formatDate(currentItem.publishTime || currentItem.createTime) }}
                </span>
              </div>
            </div>
          </div>
          
          <div class="header-cover">
            <img 
              v-if="currentItem.coverImage" 
              :src="currentItem.coverImage" 
              :alt="currentItem.title"
              class="cover-image"
            />
            <div v-else class="no-cover">
              <i class="fas fa-image"></i>
              <p>暂无封面</p>
            </div>
          </div>
        </div>
      </div>

      <!-- 主体内容 -->
      <div class="detail-body">
        <!-- 左侧：媒体展示 -->
        <div class="media-section">
          <!-- 主要媒体展示 -->
          <div class="main-media">
            <div v-if="mainMedia" class="media-display">
              <!-- 图片展示 -->
              <img 
                v-if="mainMedia.type === 'IMG'" 
                :src="mainMedia.filePath" 
                :alt="currentItem.title"
                class="main-image"
              />
              
              <!-- 视频展示 -->
              <video 
                v-else-if="mainMedia.type === 'VIDEO'" 
                :src="mainMedia.filePath" 
                controls 
                class="main-video"
              >
                您的浏览器不支持视频播放
              </video>
              
              <!-- 音频展示 -->
              <div v-else-if="mainMedia.type === 'AUDIO'" class="audio-player">
                <i class="fas fa-music"></i>
                <audio :src="mainMedia.filePath" controls class="main-audio">
                  您的浏览器不支持音频播放
                </audio>
              </div>
              
              <!-- 无媒体时显示默认图 -->
              <div v-else class="no-media">
                <i class="fas fa-image"></i>
                <p>暂无媒体文件</p>
              </div>
            </div>
            <div v-else class="no-media">
              <i class="fas fa-image"></i>
              <p>暂无媒体文件</p>
            </div>
          </div>

          <!-- 媒体缩略图列表 -->
          <div v-if="mediaList.length > 1" class="media-thumbnails">
            <div 
              v-for="(media, index) in mediaList" 
              :key="media.id"
              class="thumbnail-item"
              :class="{ active: currentMediaIndex === index }"
              @click="handleMediaChange(index)"
            >
              <img 
                v-if="media.type === 'IMG'" 
                :src="media.filePath" 
                :alt="media.originalName"
              />
              <div v-else class="thumbnail-icon">
                <i :class="getMediaIcon(media.type)"></i>
              </div>
            </div>
          </div>
        </div>

        <!-- 右侧：作品信息 -->
        <div class="info-section">
          <!-- 摘要 -->
          <div v-if="currentItem.summary" class="summary-box">
            <h3>作品摘要</h3>
            <p class="summary-text">{{ currentItem.summary }}</p>
          </div>

          <!-- 详细描述 -->
          <div class="description-box">
            <h3>详细介绍</h3>
            <div v-if="currentItem.description" class="description-content" v-html="formatDescription(currentItem.description)"></div>
            <div v-else class="no-content">暂无详细介绍</div>
          </div>

          <!-- 创建人信息 -->
          <div v-if="currentItem.creatorName" class="creator-box">
            <h3>创建人</h3>
            <div class="creator-info">
              <i class="fas fa-user-circle"></i>
              <span>{{ currentItem.creatorName }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- 关联传承人 -->
      <div v-if="currentItem.inheritorList && currentItem.inheritorList.length > 0" class="inheritor-section">
        <h2>相关传承人</h2>
        <div class="inheritor-grid">
          <div 
            v-for="inheritor in currentItem.inheritorList" 
            :key="inheritor.id"
            class="inheritor-card"
          >
            <div class="inheritor-avatar">
              <img v-if="inheritor.avatarPath" :src="inheritor.avatarPath" :alt="inheritor.name" />
              <i v-else class="fas fa-user"></i>
            </div>
            <div class="inheritor-info">
              <h4>{{ inheritor.name }}</h4>
              <p class="inheritor-title">{{ inheritor.title || '传承人' }}</p>
              <p class="inheritor-region">
                <i class="fas fa-map-marker-alt"></i>
                {{ inheritor.region || '未知地区' }}
              </p>
            </div>
            <a-button type="primary" size="small">
              了解更多
            </a-button>
          </div>
        </div>
      </div>

      <!-- 相关推荐 -->
      <div v-if="relatedItems.length > 0" class="related-section">
        <h2>相关作品推荐</h2>
        <div class="related-grid">
          <HeritageItemCard
            v-for="item in relatedItems"
            :key="item.id"
            :item="item"
            @click="handleRelatedItemClick"
          />
        </div>
      </div>
    </div>

    <!-- 未找到 -->
    <div v-else class="not-found">
      <i class="fas fa-exclamation-triangle"></i>
      <p>未找到该作品</p>
      <a-button type="primary" @click="handleBackToList">返回列表</a-button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { message } from 'ant-design-vue'
import { 
  getHeritageItemDetail, 
  getHeritageItemPage
} from '@/api/HeritageApi'
import HeritageItemCard from '@/components/common/HeritageItemCard.vue'
import { formatLocalDate } from '@/utils/dateUtils'

// ========== 路由 ==========
const route = useRoute()
const router = useRouter()

// ========== 响应式数据 ==========
const loading = ref(false)
const currentItem = ref(null)
const mediaList = ref([])
const currentMediaIndex = ref(0)
const relatedItems = ref([])

// ========== 计算属性 ==========
const mainMedia = computed(() => {
  return mediaList.value[currentMediaIndex.value] || null
})

// ========== 方法 ==========

/**
 * 格式化日期
 */
function formatDate(dateStr) {
  if (!dateStr) return ''
  try {
    return formatLocalDate(new Date(dateStr))
  } catch (error) {
    return ''
  }
}

/**
 * 格式化描述
 */
function formatDescription(description) {
  if (!description) return ''
  // 将换行符转换为HTML换行
  return description.replace(/\n/g, '<br>')
}

/**
 * 获取媒体图标
 */
function getMediaIcon(type) {
  const iconMap = {
    VIDEO: 'fas fa-play-circle',
    AUDIO: 'fas fa-music',
    PDF: 'fas fa-file-pdf',
    FILE: 'fas fa-file'
  }
  return iconMap[type] || 'fas fa-file'
}

/**
 * 获取作品详情
 */
function fetchItemDetail() {
  const itemId = route.params.id
  if (!itemId) {
    message.error('作品ID不能为空')
    handleBackToList()
    return
  }

  loading.value = true
  
  getHeritageItemDetail({ itemId }, {
    onSuccess: (res) => {
      currentItem.value = res
      loading.value = false
      
      // 直接从作品详情中获取媒体文件列表
      if (res.mediaList && res.mediaList.length > 0) {
        mediaList.value = res.mediaList
        currentMediaIndex.value = 0
        console.log(`成功加载 ${res.mediaList.length} 个媒体文件`)
      } else {
        mediaList.value = []
        currentMediaIndex.value = 0
      }
      
      // 获取相关推荐
      fetchRelatedItems(res.category)
    },
    onError: (error) => {
      message.error('获取作品详情失败')
      loading.value = false
    }
  })
}

// fetchMediaList 函数已移除，直接从作品详情中获取媒体文件

/**
 * 获取相关推荐
 */
function fetchRelatedItems(category) {
  if (!category) return
  
  getHeritageItemPage({ 
    category, 
    status: 2, 
    currentPage: 1, 
    size: 4 
  }, {
    onSuccess: (res) => {
      // 排除当前作品
      relatedItems.value = (res.records || []).filter(
        item => item.id !== currentItem.value.id
      )
    },
    onError: (error) => {
      console.error('获取相关作品失败:', error)
    }
  })
}

/**
 * 切换媒体
 */
function handleMediaChange(index) {
  currentMediaIndex.value = index
}

/**
 * 分享处理
 */
function handleShare(platform) {
  const url = window.location.href
  const title = currentItem.value.title
  
  switch (platform) {
    case 'wechat':
      message.info('请使用微信扫描分享')
      break
    case 'weibo':
      window.open(`http://service.weibo.com/share/share.php?url=${encodeURIComponent(url)}&title=${encodeURIComponent(title)}`)
      break
    case 'link':
      navigator.clipboard.writeText(url).then(() => {
        message.success('链接已复制到剪贴板')
      }).catch(() => {
        message.error('复制失败，请手动复制')
      })
      break
  }
}

/**
 * 返回列表
 */
function handleBackToList() {
  router.push('/heritage')
}

/**
 * 相关作品点击
 */
function handleRelatedItemClick(item) {
  router.push(`/heritage/${item.id}`)
  // 重新加载当前页面数据
  fetchItemDetail()
}

// ========== 生命周期 ==========
onMounted(() => {
  fetchItemDetail()
})
</script>

<style scoped>
/* ========== 页面整体 ========== */
.heritage-detail-page {
  min-height: 100vh;
  background: #faf8f3; /* 宣纸白 */
}

.loading-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 40px 20px;
  background: white;
  border: 2px solid #e8e8e8;
  border-radius: 8px;
}

.detail-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 32px 24px 40px;
}

/* ========== 页面头部 ========== */
.detail-header {
  background: #fff;
  padding: 36px 40px;
  margin-bottom: 32px;
  border: 2px solid #e8e8e8;
  border-radius: 8px;
  box-shadow: 0 4px 16px rgba(139, 69, 19, 0.08);
  position: relative;
}

/* 传统边框装饰 */
.detail-header::before {
  content: '';
  position: absolute;
  top: 8px;
  left: 8px;
  right: 8px;
  bottom: 8px;
  border: 1px solid rgba(139, 69, 19, 0.1);
  border-radius: 4px;
  pointer-events: none;
}

.header-content {
  display: flex;
  gap: 40px;
  align-items: flex-start;
}

.header-left {
  flex: 1;
  min-width: 0;
}

.header-cover {
  width: 200px;
  height: 200px;
  flex-shrink: 0;
  border: 3px solid #8b4513;
  border-radius: 8px;
  overflow: hidden;
  background: #faf8f3;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.header-cover::after {
  content: '';
  position: absolute;
  top: 4px;
  left: 4px;
  right: 4px;
  bottom: 4px;
  border: 1px solid rgba(139, 69, 19, 0.2);
  border-radius: 4px;
  pointer-events: none;
}

.cover-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.no-cover {
  text-align: center;
  color: #999;
}

.no-cover i {
  font-size: 48px;
  margin-bottom: 8px;
  display: block;
  color: #d4d4d4;
}

.no-cover p {
  font-size: 14px;
  margin: 0;
  letter-spacing: 1px;
}

.header-breadcrumb {
  margin-bottom: 28px;
  padding-bottom: 16px;
  border-bottom: 1px solid #f0f0f0;
}

.header-breadcrumb :deep(.ant-breadcrumb) {
  font-size: 14px;
}

.header-breadcrumb :deep(.ant-breadcrumb-link) {
  color: #666;
  transition: color 0.3s;
}

.header-breadcrumb :deep(.ant-breadcrumb-link:hover) {
  color: #8b4513;
}

.header-breadcrumb :deep(.ant-breadcrumb-separator) {
  color: #999;
}

.header-title-section {
  position: relative;
}

.detail-title {
  font-size: 42px;
  font-weight: 700;
  color: #2c2c2c;
  margin: 0 0 24px 0;
  line-height: 1.4;
  letter-spacing: 4px;
  font-family: 'SimSun', '宋体', serif;
}

.header-meta {
  display: flex;
  align-items: center;
  gap: 20px;
  flex-wrap: wrap;
}

.category-tag {
  background: #8b4513 !important;
  color: #fff !important;
  border: none !important;
  padding: 6px 18px !important;
  font-size: 15px !important;
  border-radius: 4px !important;
  letter-spacing: 1px;
  font-weight: 500;
}

.region-tag,
.publish-time {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #666;
  font-size: 15px;
}

.region-tag i,
.publish-time i {
  font-size: 14px;
  color: #8b4513;
}

/* ========== 主体内容 ========== */
.detail-body {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 24px;
  margin-bottom: 40px;
}

/* ========== 媒体区域 ========== */
.media-section {
  background: #fff;
  padding: 28px;
  border: 2px solid #e8e8e8;
  box-shadow: 0 4px 16px rgba(139, 69, 19, 0.08);
  border-radius: 8px;
  position: relative;
}

/* 传统边框装饰 */
.media-section::before {
  content: '';
  position: absolute;
  top: 8px;
  left: 8px;
  right: 8px;
  bottom: 8px;
  border: 1px solid rgba(139, 69, 19, 0.1);
  border-radius: 4px;
  pointer-events: none;
}

.main-media {
  margin-bottom: 20px;
}

.media-display {
  width: 100%;
  overflow: hidden;
  background: #000;
  border: 2px solid #e8e8e8;
  border-radius: 4px;
}

.main-image {
  width: 100%;
  height: auto;
  display: block;
  max-height: 500px;
  object-fit: contain;
}

.main-video {
  width: 100%;
  max-height: 500px;
  border-radius: 4px;
}

/* 音频播放器 */
.audio-player {
  padding: 60px 32px;
  text-align: center;
  background: linear-gradient(135deg, #2c2c2c 0%, #3a3a3a 100%);
  color: #faf8f3;
  border: 3px solid #8b4513;
  border-radius: 4px;
  position: relative;
}

.audio-player::before {
  content: '';
  position: absolute;
  top: 8px;
  left: 8px;
  right: 8px;
  bottom: 8px;
  border: 1px solid rgba(139, 69, 19, 0.3);
  border-radius: 2px;
}

.audio-player i {
  font-size: 56px;
  margin-bottom: 28px;
  color: #c5a572;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

.main-audio {
  width: 100%;
  position: relative;
  z-index: 1;
}

.no-media {
  text-align: center;
  padding: 100px 20px;
  color: #999;
  background: linear-gradient(to bottom, #fdfcfa 0%, #faf8f3 100%);
  border: 2px dashed #d4d4d4;
  border-radius: 4px;
}

.no-media i {
  font-size: 72px;
  margin-bottom: 20px;
  display: block;
  color: #d4d4d4;
}

.no-media p {
  font-size: 16px;
  letter-spacing: 2px;
}

/* 缩略图列表 */
.media-thumbnails {
  display: flex;
  gap: 12px;
  overflow-x: auto;
  padding: 8px 4px;
}

/* 自定义滚动条 */
.media-thumbnails::-webkit-scrollbar {
  height: 6px;
}

.media-thumbnails::-webkit-scrollbar-track {
  background: rgba(139, 69, 19, 0.05);
  border-radius: 3px;
}

.media-thumbnails::-webkit-scrollbar-thumb {
  background: rgba(139, 69, 19, 0.3);
  border-radius: 3px;
}

.thumbnail-item {
  width: 88px;
  height: 88px;
  overflow: hidden;
  cursor: pointer;
  border: 3px solid #e8e8e8;
  transition: all 0.3s ease;
  flex-shrink: 0;
  background: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  position: relative;
}

.thumbnail-item::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border: 2px solid transparent;
  transition: border-color 0.3s;
  pointer-events: none;
}

.thumbnail-item:hover {
  border-color: #c5a572;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(139, 69, 19, 0.15);
}

.thumbnail-item.active {
  border-color: #8b4513;
  box-shadow: 0 4px 12px rgba(139, 69, 19, 0.25);
}

.thumbnail-item.active::after {
  border-color: rgba(139, 69, 19, 0.2);
  top: 4px;
  left: 4px;
  right: 4px;
  bottom: 4px;
}

.thumbnail-item img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.thumbnail-icon {
  color: #8b4513;
  font-size: 28px;
}

/* ========== 信息区域 ========== */
.info-section {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.summary-box,
.description-box,
.creator-box {
  background: #fff;
  padding: 28px;
  border: 2px solid #e8e8e8;
  box-shadow: 0 4px 16px rgba(139, 69, 19, 0.08);
  border-radius: 8px;
  position: relative;
}

/* 传统边框装饰 */
.summary-box::before,
.description-box::before,
.creator-box::before {
  content: '';
  position: absolute;
  top: 8px;
  left: 8px;
  right: 8px;
  bottom: 8px;
  border: 1px solid rgba(139, 69, 19, 0.1);
  border-radius: 4px;
  pointer-events: none;
}

.summary-box h3,
.description-box h3,
.creator-box h3 {
  font-size: 19px;
  font-weight: 600;
  color: #2c2c2c;
  margin: 0 0 20px 0;
  padding-bottom: 14px;
  border-bottom: 3px solid #8b4513;
  letter-spacing: 3px;
  font-family: 'SimSun', '宋体', serif;
  position: relative;
}

/* 标题装饰点 */
.summary-box h3::after,
.description-box h3::after,
.creator-box h3::after {
  content: '';
  position: absolute;
  bottom: -3px;
  left: 0;
  width: 60px;
  height: 3px;
  background: #c5a572;
}

.summary-text {
  font-size: 16px;
  line-height: 2.2;
  color: #555;
  margin: 0;
  text-align: justify;
}

.description-content {
  font-size: 15px;
  line-height: 2.2;
  color: #555;
  text-align: justify;
}

.no-content {
  color: #999;
  font-style: italic;
  text-align: center;
  padding: 32px 20px;
  background: rgba(139, 69, 19, 0.02);
  border-radius: 4px;
}

.creator-info {
  display: flex;
  align-items: center;
  gap: 12px;
  font-size: 17px;
  color: #555;
  padding: 12px 16px;
  background: rgba(139, 69, 19, 0.03);
  border-radius: 6px;
  border-left: 4px solid #8b4513;
}

.creator-info i {
  font-size: 22px;
  color: #8b4513;
}

/* ========== 传承人区域 ========== */
.inheritor-section {
  background: #fff;
  padding: 48px 40px;
  margin-bottom: 40px;
  border: 2px solid #e8e8e8;
  border-radius: 8px;
  box-shadow: 0 4px 16px rgba(139, 69, 19, 0.08);
  position: relative;
}

/* 传统边框装饰 */
.inheritor-section::before {
  content: '';
  position: absolute;
  top: 8px;
  left: 8px;
  right: 8px;
  bottom: 8px;
  border: 1px solid rgba(139, 69, 19, 0.1);
  border-radius: 4px;
  pointer-events: none;
}

.inheritor-section h2 {
  font-size: 32px;
  font-weight: 600;
  color: #2c2c2c;
  margin: 0 0 40px 0;
  text-align: center;
  letter-spacing: 8px;
  position: relative;
  padding-bottom: 20px;
  font-family: 'SimSun', '宋体', serif;
}

/* 标题下方装饰线 */
.inheritor-section h2::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 80px;
  height: 4px;
  background: linear-gradient(to right, transparent, #8b4513, transparent);
}

.inheritor-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 24px;
}

.inheritor-card {
  border: 2px solid #e8e8e8;
  border-radius: 8px;
  padding: 28px;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  transition: all 0.3s ease;
  background: #fff;
  cursor: pointer;
}

.inheritor-card:hover {
  border-color: #c5a572;
  box-shadow: 0 8px 20px rgba(139, 69, 19, 0.15);
  transform: translateY(-4px);
}

.inheritor-avatar {
  width: 110px;
  height: 110px;
  border-radius: 50%;
  overflow: hidden;
  background: #faf8f3;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
  border: 4px solid #8b4513;
  position: relative;
}

/* 头像外圈装饰 */
.inheritor-avatar::after {
  content: '';
  position: absolute;
  width: 120px;
  height: 120px;
  border: 2px solid rgba(139, 69, 19, 0.2);
  border-radius: 50%;
  top: -5px;
  left: -5px;
}

.inheritor-avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.inheritor-avatar i {
  font-size: 44px;
  color: #999;
}

.inheritor-info {
  margin-bottom: 20px;
  flex: 1;
}

.inheritor-info h4 {
  font-size: 19px;
  font-weight: 600;
  color: #2c2c2c;
  margin: 0 0 10px 0;
  letter-spacing: 2px;
  font-family: 'SimSun', '宋体', serif;
}

.inheritor-title {
  font-size: 14px;
  color: #8b4513;
  margin: 0 0 6px 0;
  font-weight: 500;
}

.inheritor-region {
  font-size: 14px;
  color: #666;
  margin: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
}

.inheritor-region i {
  color: #8b4513;
}

.inheritor-card :deep(.ant-btn) {
  background: linear-gradient(135deg, #8b4513 0%, #a0522d 100%);
  border: none;
  color: #fff;
  font-weight: 500;
  letter-spacing: 1px;
  transition: all 0.3s;
}

.inheritor-card :deep(.ant-btn:hover) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(139, 69, 19, 0.4);
}

/* ========== 相关推荐区域 ========== */
.related-section {
  background: #fff;
  padding: 48px 40px;
  border: 2px solid #e8e8e8;
  border-radius: 8px;
  box-shadow: 0 4px 16px rgba(139, 69, 19, 0.08);
  position: relative;
}

/* 传统边框装饰 */
.related-section::before {
  content: '';
  position: absolute;
  top: 8px;
  left: 8px;
  right: 8px;
  bottom: 8px;
  border: 1px solid rgba(139, 69, 19, 0.1);
  border-radius: 4px;
  pointer-events: none;
}

.related-section h2 {
  font-size: 32px;
  font-weight: 600;
  color: #2c2c2c;
  margin: 0 0 40px 0;
  text-align: center;
  letter-spacing: 8px;
  position: relative;
  padding-bottom: 20px;
  font-family: 'SimSun', '宋体', serif;
}

.related-section h2::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 80px;
  height: 4px;
  background: linear-gradient(to right, transparent, #8b4513, transparent);
}

.related-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 24px;
}

/* ========== 未找到 ========== */
.not-found {
  text-align: center;
  padding: 140px 20px;
  color: #999;
  background: #fff;
  border-radius: 8px;
  border: 2px dashed #e8e8e8;
}

.not-found i {
  font-size: 88px;
  margin-bottom: 28px;
  color: #d4d4d4;
}

.not-found p {
  font-size: 22px;
  margin-bottom: 32px;
  color: #666;
  letter-spacing: 2px;
}

.not-found :deep(.ant-btn) {
  background: linear-gradient(135deg, #8b4513 0%, #a0522d 100%);
  border: none;
  color: #fff;
  font-size: 16px;
  padding: 12px 32px;
  height: auto;
  font-weight: 500;
  letter-spacing: 2px;
}

.not-found :deep(.ant-btn:hover) {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(139, 69, 19, 0.4);
}

/* 响应式设计 */
@media (max-width: 1024px) {
  .detail-body {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 768px) {
  .detail-container {
    padding: 12px;
  }
  
  .detail-header {
    padding: 20px;
  }
  
  .header-content {
    flex-direction: column;
    gap: 20px;
  }
  
  .header-cover {
    width: 100%;
    height: 200px;
    order: -1;
  }
  
  .detail-title {
    font-size: 26px;
    letter-spacing: 2px;
  }
  
  .media-section,
  .inheritor-section,
  .related-section {
    padding: 20px 16px;
  }
  
  .summary-box,
  .description-box,
  .creator-box {
    padding: 20px;
  }
  
  .inheritor-grid,
  .related-grid {
    grid-template-columns: 1fr;
  }
  
  .inheritor-section h2,
  .related-section h2 {
    font-size: 24px;
    letter-spacing: 4px;
  }
}
</style>
