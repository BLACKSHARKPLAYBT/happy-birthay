<template>
  <section class="about-section" id="about">
    <div class="section-header">
      <h2 class="rainbow-text">关于我</h2>
    </div>
    <div class="about-content">
      <div class="about-card">
        <h3 class="gradient-text-primary">个人简介</h3>
        <p class="readable-text">来自安徽六安的开发者，热爱创造和编程。目前专注于Vue开发，同时对游戏开发也有浓厚兴趣。</p>
      </div>
      
      <div class="about-card">
        <h3 class="gradient-text-secondary">项目经历</h3>
        <ul class="project-list">
          <li>
            <span class="project-title glow-text">鸣潮网站复刻</span>
            <span class="project-tech">Vue</span>
          </li>
          <li>
            <span class="project-title glow-text">Waline表情包开发</span>
            <span class="project-desc">为评论系统定制透明背景表情包</span>
          </li>
          <li>
            <span class="project-title glow-text">开源交流网站</span>
            <span class="project-tech">Vue</span>
          </li>
        </ul>
      </div>

      <div class="about-card">
        <h3 class="gradient-text-accent">游戏爱好</h3>
        <p class="readable-text">Steam游戏收藏：158款</p>
        <p class="readable-text">最近游戏：死亡搁浅</p>
        <p class="readable-text">喜欢YuzuSoft的游戏，是柚子社粉丝群成员</p>
      </div>

      <div class="about-card">
        <h3 class="gradient-text-primary">近期动态</h3>
        <ul class="activity-list">
          <li class="highlight-text">完成爱丽丝生日贺图项目</li>
          <li class="highlight-text">发布《小小世界》MC模组汉化</li>
          <li class="highlight-text">更新个人博客内容</li>
        </ul>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
// 组件逻辑
</script>

<style scoped>
.about-section {
  padding: 4rem 2rem;
  background: linear-gradient(135deg, rgba(255, 240, 245, 0.3), rgba(240, 248, 255, 0.3));
}

.about-content {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 2rem;
  max-width: 1400px;
  margin: 0 auto;
}

.about-card {
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 105, 180, 0.3);
  border-radius: 16px;
  padding: 1.5rem;
  transition: all 0.3s ease;
}

.about-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 25px rgba(255, 105, 180, 0.2);
}

/* 彩虹文字效果 */
.rainbow-text {
  font-size: 2.5rem;
  background: linear-gradient(45deg, #ff6b6b, #ffd93d, #6c5ce7, #a8e6cf);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  animation: rainbow 8s ease infinite;
}

/* 渐变文字效果 */
.gradient-text-primary {
  background: linear-gradient(45deg, #ff6b6b, #ffd93d);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  font-size: 1.4rem;
  margin-bottom: 1rem;
}

.gradient-text-secondary {
  background: linear-gradient(45deg, #6c5ce7, #a8e6cf);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  font-size: 1.4rem;
  margin-bottom: 1rem;
}

.gradient-text-accent {
  background: linear-gradient(45deg, #ff9a9e, #fad0c4);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  font-size: 1.4rem;
  margin-bottom: 1rem;
}

/* 发光文字效果 */
.glow-text {
  color: #fff;
  text-shadow: 0 0 10px rgba(255, 255, 255, 0.5);
  font-weight: bold;
}

/* 高亮文字效果 */
.highlight-text {
  color: #fff;
  background: rgba(255, 255, 255, 0.1);
  padding: 0.8rem;
  border-radius: 8px;
  margin-bottom: 0.8rem;
  transition: all 0.3s ease;
}

.highlight-text:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateX(5px);
}

/* 提高可读性的文本样式 */
.readable-text {
  color: rgba(255, 255, 255, 0.9);
  line-height: 1.6;
  font-size: 1.05rem;
  letter-spacing: 0.3px;
}

.project-list, .activity-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.project-list li {
  margin-bottom: 1.2rem;
  padding: 0.8rem;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.08);
  transition: all 0.3s ease;
}

.project-list li:hover {
  background: rgba(255, 255, 255, 0.12);
}

.project-title {
  font-weight: bold;
  margin-right: 1rem;
}

.project-tech {
  font-size: 0.9rem;
  color: #ffd93d;
  padding: 0.3rem 0.6rem;
  border-radius: 4px;
  background: rgba(255, 217, 61, 0.15);
}

.project-desc {
  display: block;
  font-size: 0.95rem;
  color: rgba(255, 255, 255, 0.7);
  margin-top: 0.5rem;
}

@keyframes rainbow {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

@media (max-width: 768px) {
  .about-section {
    padding: 2rem 1rem;
  }
  
  .about-content {
    grid-template-columns: 1fr;
  }

  .rainbow-text {
    font-size: 2rem;
  }
}
</style>