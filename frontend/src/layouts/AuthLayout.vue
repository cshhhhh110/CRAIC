<template>
  <div class="auth-layout">
    <div class="left-section">
      <div class="image-wrapper">
        <img src="@/assets/10.png" class="background-image" />
        <div class="image-overlay">
          <h2>非遗传承 文化瑰宝</h2>
          <p>探索中华五千年文化遗产 · 传承匠心技艺</p>
        </div>
      </div>
    </div>
    <div class="right-section">
    <div class="auth-container">
      <!-- 传统装饰边框 -->
      <div class="decoration-border">
        <div class="corner corner-tl"></div>
        <div class="corner corner-tr"></div>
        <div class="corner corner-bl"></div>
        <div class="corner corner-br"></div>
      </div>

      <div class="auth-header">
        <div class="logo-wrapper">
          <div class="logo-ornament"></div>
          <h1 class="site-title">{{ siteName }}</h1>
          <div class="logo-ornament"></div>
        </div>
        <p class="site-description">{{ siteDescription }}</p>
        <div class="decorative-line"></div>
      </div>

      <div class="auth-content">
        <router-view />
      </div>

      <div class="auth-footer">
        <div class="footer-divider"></div>
        <p class="copyright">{{ siteConfig.copyright.text }}</p>
      </div>
    </div>
    </div>
  </div>
</template>

<script setup>
import siteConfig from '@/config/site'

// 使用统一的站点配置
const siteName = siteConfig.name
const siteDescription = siteConfig.description
</script>

<style scoped>
/* 引入思源字体 */
@import url('https://fonts.googleapis.com/css2?family=Noto+Serif+SC:wght@400;500;700&family=Noto+Sans+SC:wght@400;500&display=swap');

.auth-layout {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background-image: url('@/assets/222.png');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  position: relative;
  overflow: hidden;
  font-family: 'Noto Sans SC', '思源黑体', sans-serif;
}

/* 背景遮罩层，增加文字可读性 */
.auth-layout::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(248, 245, 240, 0.2);
  z-index: 0;
}

.auth-container {
  max-width: 460px;
  width: 100%;
  margin: 0 auto;
  padding: 3rem 2.5rem;
  background: rgba(255, 255, 255, 0.6);
  backdrop-filter: blur(10px);
  border-radius: 0;
  box-shadow: 
    0 8px 32px rgba(44, 44, 44, 0.08),
    0 2px 8px rgba(44, 44, 44, 0.04);
  position: relative;
  z-index: 1;
  animation: fadeIn 0.5s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.left-section {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  background: none;
  padding: 5px;
}

.image-wrapper {
  width: 420px;
  height: 540px;
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #fff;
  border-radius: 15px;
  box-shadow: 0 8px 24px rgba(0,0,0,0.08);
  overflow: hidden;
  transition: all 0.3s ease;
}

.image-wrapper:hover {
  transform: translateY(-5px);
  box-shadow: 0 12px 28px rgba(0,0,0,0.12);
}

.background-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.image-wrapper:hover .background-image {
  transform: scale(1.05);
}

.image-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 20px;
  background: linear-gradient(to top, rgba(0,0,0,0.7), transparent);
  color: white;
  text-align: center;
}

.image-overlay h2 {
  margin: 0;
  font-size: 24px;
  font-weight: 600;
}

.image-overlay p {
  margin: 5px 0 0;
  font-size: 16px;
  opacity: 0.9;
}

.right-section {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  background: none;
  padding: 20px;
}
/* 传统装饰边框 */
.decoration-border {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.corner {
  position: absolute;
  width: 60px;
  height: 60px;
  border-color: #C5A572;
  border-style: solid;
  border-width: 0;
}

.corner-tl {
  top: 0;
  left: 0;
  border-top-width: 2px;
  border-left-width: 2px;
}

.corner-tr {
  top: 0;
  right: 0;
  border-top-width: 2px;
  border-right-width: 2px;
}

.corner-bl {
  bottom: 0;
  left: 0;
  border-bottom-width: 2px;
  border-left-width: 2px;
}

.corner-br {
  bottom: 0;
  right: 0;
  border-bottom-width: 2px;
  border-right-width: 2px;
}

.auth-header {
  text-align: center;
  margin-bottom: 2.5rem;
}

.logo-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 0.8rem;
}

.logo-ornament {
  width: 40px;
  height: 2px;
  background: linear-gradient(90deg, transparent, #C5A572, transparent);
}

.site-title {
  margin: 0;
  font-family: 'Noto Serif SC', '思源宋体', serif;
  font-size: 2rem;
  font-weight: 700;
  color: #2C2C2C;
  letter-spacing: 0.3rem;
  position: relative;
}

.site-description {
  margin: 0 0 1rem 0;
  color: #666666;
  font-size: 0.95rem;
  letter-spacing: 0.1rem;
  line-height: 1.6;
}

.decorative-line {
  width: 80px;
  height: 3px;
  margin: 0 auto;
  background: linear-gradient(90deg, transparent, #D4282D, transparent);
  border-radius: 2px;
}

.auth-content {
  margin-bottom: 2rem;
  animation: slideUp 0.5s ease-out 0.2s both;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.auth-footer {
  text-align: center;
  padding-top: 1.5rem;
}

.footer-divider {
  width: 100%;
  height: 1px;
  background: linear-gradient(90deg, transparent, #E0E0E0, transparent);
  margin-bottom: 1rem;
}

.copyright {
  margin: 0;
  color: #999999;
  font-size: 0.85rem;
  letter-spacing: 0.05rem;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .auth-container {
    max-width: 90%;
    padding: 2rem 1.5rem;
  }

  .site-title {
    font-size: 1.5rem;
    letter-spacing: 0.2rem;
  }

  .corner {
    width: 40px;
    height: 40px;
  }
}
</style>
