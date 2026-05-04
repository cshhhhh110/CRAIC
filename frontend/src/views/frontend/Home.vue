<template>
  <div class="heritage-home">
    <!-- 1. 英雄横幅区 - 传统卷轴展开效果 -->
    <section class="hero-banner">
      <div class="scroll-container">
        <div class="scroll-content">
          <h1 class="main-title">非遗传承 文化瑰宝</h1>
          <p class="subtitle">探索中华五千年文化遗产 · 传承匠心技艺</p>
          <div class="hero-actions">
            <a-button type="primary" size="large" @click="exploreHeritage">
              <template #icon><CompassOutlined /></template>
              探索非遗
            </a-button>
            <a-button size="large" @click="learnCourses">
              <template #icon><BookOutlined /></template>
              学习课程
            </a-button>
          </div>
        </div>
        <div class="scroll-decoration">
          <!-- 传统装饰元素 -->
          <div class="corner-pattern top-left"></div>
          <div class="corner-pattern top-right"></div>
          <div class="corner-pattern bottom-left"></div>
          <div class="corner-pattern bottom-right"></div>
        </div>
      </div>
    </section>

    <!-- 2. 精选作品 -->
    <section class="featured-items">
      <div class="section-header">
        <h2 class="section-title">
          <span class="title-decoration">【</span>
          精选作品
          <span class="title-decoration">】</span>
        </h2>
        <a-button type="link" @click="viewAllItems">
          查看更多 <ArrowRightOutlined />
        </a-button>
      </div>

      <div class="items-grid">
        <div 
          v-for="item in featuredItems" 
          :key="item.id"
          class="item-card"
          @click="viewItemDetail(item.id)"
        >
          <div class="item-image">
            <img :src="getItemCoverUrl(item)" :alt="item.title" />
            <div class="item-category-tag">{{ item.category }}</div>
          </div>
          <div class="item-content">
            <h4 class="item-title">{{ item.title }}</h4>
            <p class="item-region">
              <EnvironmentOutlined />
              {{ item.region }}
            </p>
            <p class="item-summary">{{ item.summary }}</p>
          </div>
        </div>
      </div>
    </section>

    <!-- 4. 传承人风采 - 圆形头像墙 -->
    <section class="inheritor-showcase">
      <div class="section-header">
        <h2 class="section-title">
          <span class="title-decoration">【</span>
          传承人风采
          <span class="title-decoration">】</span>
        </h2>
        <a-button type="link" @click="viewAllInheritors">
          查看更多 <ArrowRightOutlined />
        </a-button>
      </div>

      <div class="inheritor-wall">
        <div 
          v-for="inheritor in inheritors" 
          :key="inheritor.id"
          class="inheritor-avatar-card"
          @click="viewInheritorDetail(inheritor.id)"
        >
          <div class="avatar-container">
            <div class="avatar-border">
              <img 
                :src="getInheritorAvatarUrl(inheritor)" 
                :alt="inheritor.name"
                class="avatar-image"
              />
            </div>
            <div class="avatar-title-badge">{{ inheritor.title }}</div>
          </div>
          <h4 class="inheritor-name">{{ inheritor.name }}</h4>
          <p class="inheritor-region">{{ inheritor.region }}</p>
        </div>
      </div>
    </section>

    <!-- 5. 近期活动 - 时间轴式展示 -->
    <section class="upcoming-activities">
      <div class="section-header">
        <h2 class="section-title">
          <span class="title-decoration">【</span>
          近期活动
          <span class="title-decoration">】</span>
        </h2>
        <a-button type="link" @click="viewAllActivities">
          查看更多 <ArrowRightOutlined />
        </a-button>
      </div>

      <div class="activity-timeline">
        <div
          v-for="(activity, index) in activities"
          :key="activity.id"
          class="activity-node"
          :class="{ 'node-alternate': index % 2 === 1 }"
        >
          <div class="timeline-dot">
            <div class="dot-inner"></div>
          </div>
          <div class="activity-card" @click="viewActivityDetail(activity.id)">
            <div class="activity-cover">
              <img :src="getActivityCoverUrl(activity)" :alt="activity.title" />
            </div>
            <div class="activity-info">
              <div class="activity-time">
                <CalendarOutlined />
                {{ formatDate(activity.startTime) }}
              </div>
              <h4 class="activity-title">{{ activity.title }}</h4>
              <div class="activity-meta">
                <a-tag :color="getActivityTypeColor(activity.type)">
                  {{ activity.type }}
                </a-tag>
                <span class="activity-location">
                  <EnvironmentOutlined />
                  {{ activity.location }}
                </span>
              </div>
              <p class="activity-desc">{{ activity.description?.substring(0, 80) }}...</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- 6. 在线课程 - 书架式布局 -->
    <section class="online-courses">
      <div class="section-header">
        <h2 class="section-title">
          <span class="title-decoration">【</span>
          在线课程
          <span class="title-decoration">】</span>
        </h2>
        <a-button type="link" @click="viewAllCourses">
          查看更多 <ArrowRightOutlined />
        </a-button>
      </div>

      <div class="bookshelf">
        <div class="shelf-row">
          <div 
            v-for="course in courses" 
            :key="course.id"
            class="book-spine"
            @click="viewCourseDetail(course.id)"
          >
            <div class="book-content">
              <h4 class="book-title">{{ course.title }}</h4>
              <div class="book-level">{{ getLevelText(course.level) }}</div>
            </div>
          </div>
        </div>
      </div>
    </section>

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { 
  CompassOutlined, 
  BookOutlined, 
  ArrowRightOutlined,
  EnvironmentOutlined,
  CalendarOutlined
} from '@ant-design/icons-vue'
import { getHeritageItemPage } from '@/api/HeritageApi'
import { getInheritorPage } from '@/api/InheritorApi'
import { getActivityPage } from '@/api/ActivityApi'
import { getCoursePage } from '@/api/CourseApi'
import { API_BASE_URL } from '@/config/api'

const router = useRouter()

// 数据
const featuredItems = ref([])
const inheritors = ref([])
const activities = ref([])
const courses = ref([])

// 加载数据
onMounted(async () => {
  try {
    // 加载精选作品 - 4个
    const itemsRes = await getHeritageItemPage({ 
      current: 1, 
      size: 4, 
      status: 2 
    }, {
      showDefaultMsg: false
    })
    featuredItems.value = itemsRes?.records || []

    // 加载传承人 - 4个
    const inheritorsRes = await getInheritorPage({ 
      current: 1, 
      size: 4 
    }, {
      showDefaultMsg: false
    })
    inheritors.value = inheritorsRes?.records || []

    // 加载活动 - 4个
    const activitiesRes = await getActivityPage({ 
      current: 1, 
      size: 4, 
      status: 1 
    }, {
      showDefaultMsg: false
    })
    activities.value = activitiesRes?.records || []

    // 加载课程 - 4个
    const coursesRes = await getCoursePage({ 
      current: 1, 
      size: 4, 
      status: 1 
    }, {
      showDefaultMsg: false
    })
    courses.value = coursesRes?.records || []
  } catch (error) {
    console.error('加载首页数据失败:', error)
  }
})

// 工具函数
const getImageUrl = (fileId) => {
  if (!fileId) return '/api/placeholder/400/300'
  return `${API_BASE_URL}/api/file/preview/${fileId}`
}

const getItemCoverUrl = (item) => {
  // 优先使用 coverImage（完整路径）
  if (item.coverImage) {
    return item.coverImage
  }
  // 其次使用 coverFileId
  if (item.coverFileId) {
    return `${API_BASE_URL}/api/file/preview/${item.coverFileId}`
  }
  // 默认占位图
  return '/api/placeholder/400/400'
}

const getInheritorAvatarUrl = (inheritor) => {
  // 优先使用 avatarPath（完整路径）
  if (inheritor.avatarPath) {
    return inheritor.avatarPath
  }
  // 其次使用 avatarFileId
  if (inheritor.avatarFileId) {
    return `${API_BASE_URL}/api/file/preview/${inheritor.avatarFileId}`
  }
  // 默认头像占位图
  return '/api/placeholder/160/160'
}

const getActivityCoverUrl = (activity) => {
  // 优先使用 coverFilePath（完整路径）
  if (activity.coverFilePath) {
    return activity.coverFilePath
  }
  // 其次使用 coverFileId
  if (activity.coverFileId) {
    return `${API_BASE_URL}/api/file/preview/${activity.coverFileId}`
  }
  // 默认占位图
  return '/api/placeholder/400/300'
}

const formatDate = (dateStr) => {
  if (!dateStr) return ''
  const date = new Date(dateStr)
  return `${date.getMonth() + 1}月${date.getDate()}日`
}

const getLevelText = (level) => {
  const map = {
    'beginner': '入门',
    'elementary': '初级',
    'intermediate': '中级',
    'advanced': '高级'
  }
  return map[level] || '入门'
}

const getActivityTypeColor = (type) => {
  const colorMap = {
    '展演': 'blue',
    '展览': 'green',
    '培训': 'orange',
    '比赛': 'red'
  }
  return colorMap[type] || 'default'
}

// 导航函数
const exploreHeritage = () => router.push('/heritage')
const learnCourses = () => router.push('/course')
const viewAllItems = () => router.push('/heritage')
const viewItemDetail = (id) => router.push(`/heritage/${id}`)
const viewAllInheritors = () => router.push('/inheritor')
const viewInheritorDetail = (id) => router.push(`/inheritor/${id}`)
const viewAllActivities = () => router.push('/activity')
const viewActivityDetail = (id) => router.push(`/activity/${id}`)
const viewAllCourses = () => router.push('/course')
const viewCourseDetail = (id) => router.push(`/course/${id}`)
</script>

<style scoped>
.heritage-home {
  width: 100%;
  background: #faf8f3;
}

/* ========== 1. 英雄横幅 ========== */
.hero-banner {
  height: 500px;
  background-image: url('@/assets/banner.jpeg');
  background-size: 100% 100%;
  position: relative;
  overflow: hidden;
}

.scroll-container {
  max-width: 1200px;
  margin: 0 auto;
  height: 100%;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.scroll-content {
  text-align: center;
  color: #fff;
  z-index: 2;
  position: relative;
}

.main-title {
  font-size: 66px;
  font-weight: bold;
  margin-bottom: 16px;
  letter-spacing: 8px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.7);
}

.subtitle {
  font-size: 30px;
  margin-bottom: 40px;
  letter-spacing: 2px;
  opacity: 0.95;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.7);
}

.hero-actions {
  display: flex;
  gap: 20px;
  justify-content: center;
}

.scroll-decoration .corner-pattern {
  position: absolute;
  width: 120px;
  height: 120px;
  border: 3px solid rgba(255, 255, 255, 0.2);
}

.corner-pattern.top-left {
  top: 20px;
  left: 20px;
  border-right: none;
  border-bottom: none;
}

.corner-pattern.top-right {
  top: 20px;
  right: 20px;
  border-left: none;
  border-bottom: none;
}

.corner-pattern.bottom-left {
  bottom: 20px;
  left: 20px;
  border-right: none;
  border-top: none;
}

.corner-pattern.bottom-right {
  bottom: 20px;
  right: 20px;
  border-left: none;
  border-top: none;
}

/* ========== 2. 精选作品 ========== */
.featured-items {
  max-width: 1200px;
  margin: 80px auto;
  padding: 0 24px;
}

.section-header {
  text-align: center;
  margin-bottom: 48px;
}

.section-title {
  font-size: 36px;
  font-weight: bold;
  color: #8b4513;
  margin-bottom: 8px;
}

.title-decoration {
  font-size: 28px;
  margin: 0 8px;
  opacity: 0.6;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 32px;
}

.items-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 24px;
}

.item-card {
  background: #fff;
  border-radius: 12px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.3s;
  border: 1px solid #e8e8e8;
}

.item-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.1);
}

.item-image {
  height: 200px;
  position: relative;
  overflow: hidden;
}

.item-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s;
}

.item-card:hover .item-image img {
  transform: scale(1.1);
}

.item-category-tag {
  position: absolute;
  top: 12px;
  left: 12px;
  background: rgba(139, 69, 19, 0.9);
  color: #fff;
  padding: 4px 12px;
  border-radius: 4px;
  font-size: 12px;
}

.item-content {
  padding: 20px;
}

.item-title {
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 8px;
  color: #333;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.item-region {
  font-size: 14px;
  color: #666;
  margin-bottom: 12px;
  display: flex;
  align-items: center;
  gap: 4px;
}

.item-summary {
  font-size: 14px;
  color: #999;
  line-height: 1.6;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* ========== 4. 传承人风采 ========== */
.inheritor-showcase {
  max-width: 1200px;
  margin: 80px auto;
  padding: 0 24px;
}

.inheritor-wall {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 40px;
}

.inheritor-avatar-card {
  text-align: center;
  cursor: pointer;
  transition: transform 0.3s;
}

.inheritor-avatar-card:hover {
  transform: translateY(-8px);
}

.avatar-container {
  position: relative;
  margin-bottom: 16px;
}

.avatar-border {
  width: 160px;
  height: 160px;
  margin: 0 auto;
  border-radius: 50%;
  padding: 6px;
  background: linear-gradient(135deg, #8b4513, #d2691e);
  position: relative;
}

.avatar-image {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  object-fit: cover;
  border: 4px solid #fff;
}

.avatar-title-badge {
  position: absolute;
  bottom: -12px;
  left: 50%;
  transform: translateX(-50%);
  background: #8b4513;
  color: #fff;
  padding: 6px 16px;
  border-radius: 16px;
  font-size: 12px;
  white-space: nowrap;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}

.inheritor-name {
  font-size: 18px;
  font-weight: bold;
  color: #333;
  margin-bottom: 4px;
}

.inheritor-region {
  font-size: 14px;
  color: #666;
}

/* ========== 5. 近期活动 ========== */
.upcoming-activities {
  max-width: 1200px;
  margin: 80px auto;
  padding: 0 24px;
}

.activity-timeline {
  position: relative;
  padding: 24px 0;
}

.activity-timeline::before {
  content: '';
  position: absolute;
  left: 50%;
  top: 0;
  bottom: 0;
  width: 2px;
  background: #e8e8e8;
  transform: translateX(-50%);
}

.activity-node {
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  gap: 32px;
  margin-bottom: 48px;
  position: relative;
}

.activity-node:last-child {
  margin-bottom: 0;
}

.timeline-dot {
  width: 20px;
  height: 20px;
  background: #fff;
  border: 4px solid #8b4513;
  border-radius: 50%;
  position: relative;
  z-index: 2;
}

.dot-inner {
  width: 8px;
  height: 8px;
  background: #8b4513;
  border-radius: 50%;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.activity-card {
  background: #fff;
  border: 1px solid #e8e8e8;
  border-radius: 12px;
  padding: 0;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  overflow: hidden;
}

.activity-node:not(.node-alternate) .activity-card {
  grid-column: 1;
}

.activity-node.node-alternate .activity-card {
  grid-column: 3;
}

.activity-card:hover {
  border-color: #8b4513;
  box-shadow: 0 8px 16px rgba(139, 69, 19, 0.1);
  transform: scale(1.02);
}

.activity-cover {
  width: 180px;
  height: 180px;
  flex-shrink: 0;
  overflow: hidden;
}

.activity-cover img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s;
}

.activity-card:hover .activity-cover img {
  transform: scale(1.1);
}

.activity-info {
  flex: 1;
  padding: 24px;
  display: flex;
  flex-direction: column;
}

.activity-time {
  font-size: 14px;
  color: #8b4513;
  margin-bottom: 12px;
  display: flex;
  align-items: center;
  gap: 6px;
  font-weight: bold;
}

.activity-title {
  font-size: 18px;
  font-weight: bold;
  color: #333;
  margin-bottom: 12px;
}

.activity-meta {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 12px;
}

.activity-location {
  font-size: 14px;
  color: #666;
  display: flex;
  align-items: center;
  gap: 4px;
}

.activity-desc {
  font-size: 14px;
  color: #999;
  line-height: 1.6;
}

/* ========== 6. 在线课程 ========== */
.online-courses {
  max-width: 1200px;
  margin: 80px auto;
  padding: 0 24px;
}

.bookshelf {
  background: #3e2723;
  border-radius: 12px;
  padding: 40px 32px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.2);
}

.shelf-row {
  display: flex;
  gap: 16px;
  justify-content: center;
  flex-wrap: wrap;
}

.book-spine {
  width: 160px;
  height: 280px;
  background: linear-gradient(to right, #8b4513, #a0522d);
  border-radius: 8px 8px 4px 4px;
  padding: 24px 16px;
  cursor: pointer;
  transition: all 0.3s;
  position: relative;
  box-shadow: 
    inset -4px 0 8px rgba(0, 0, 0, 0.3),
    4px 4px 8px rgba(0, 0, 0, 0.2);
}

.book-spine:hover {
  transform: translateY(-12px);
  box-shadow: 
    inset -4px 0 8px rgba(0, 0, 0, 0.3),
    4px 12px 16px rgba(0, 0, 0, 0.3);
}

.book-content {
  writing-mode: vertical-rl;
  text-orientation: mixed;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #fff;
}

.book-title {
  font-size: 18px;
  font-weight: bold;
  letter-spacing: 4px;
  margin-bottom: 16px;
}

.book-level {
  font-size: 14px;
  opacity: 0.8;
  margin-top: auto;
}

/* ========== 响应式设计 ========== */
@media (max-width: 1024px) {
  .items-grid {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .inheritor-wall {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 768px) {
  .main-title {
    font-size: 36px;
  }
  
  .items-grid {
    grid-template-columns: 1fr;
  }
  
  .inheritor-wall {
    grid-template-columns: 1fr;
  }
  
  .activity-timeline::before {
    left: 20px;
  }
  
  .activity-node {
    grid-template-columns: auto 1fr;
    padding-left: 20px;
  }
  
  .activity-node .activity-card {
    grid-column: 2 !important;
  }
  
  .activity-card {
    flex-direction: column;
  }
  
  .activity-cover {
    width: 100%;
    height: 150px;
  }
}

@media (max-width: 480px) {
  .items-grid {
    grid-template-columns: 1fr;
  }
  
  .inheritor-wall {
    grid-template-columns: 1fr;
  }
  
  .shelf-row {
    gap: 8px;
  }
  
  .book-spine {
    width: 100px;
    height: 200px;
  }
}
</style>

