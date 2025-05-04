<template>
  <section class="about-section" id="about">
    <div class="section-header">
      <h2>关于我</h2>
    </div>
    <div class="about-content">
      <div class="about-card">
        <h3>个人简介</h3>
        <p>来自安徽六安的开发者，热爱创造和编程。目前专注于Vue开发，同时对游戏开发也有浓厚兴趣。</p>
      </div>
      
      <div class="about-card">
        <h3>项目经历</h3>
        <ul class="project-list">
          <li>
            <span class="project-title">鸣潮网站复刻</span>
            <span class="project-tech">Vue</span>
          </li>
          <li>
            <span class="project-title">Waline表情包开发</span>
            <span class="project-desc">为评论系统定制透明背景表情包</span>
          </li>
          <li>
            <span class="project-title">开源交流网站</span>
            <span class="project-tech">Vue</span>
          </li>
        </ul>
      </div>

      <div class="about-card">
        <h3>游戏爱好</h3>
        <p>Steam游戏收藏：158款</p>
        <p>最近游戏：死亡搁浅</p>
        <p>喜欢YuzuSoft的游戏，是柚子社粉丝群成员</p>
      </div>

      <div class="about-card">
        <h3>近期动态</h3>
        <ul class="activity-list">
          <li>完成爱丽丝生日贺图项目</li>
          <li>发布《小小世界》MC模组汉化</li>
          <li>更新个人博客内容</li>
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
  background: linear-gradient(135deg, rgba(255, 240, 245, 0.2), rgba(240, 248, 255, 0.2));
}

.about-content {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 2rem;
  max-width: 1400px;
  margin: 0 auto;
}

.about-card {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 105, 180, 0.2);
  border-radius: 16px;
  padding: 1.5rem;
  transition: all 0.3s ease;
}

.about-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 20px rgba(255, 105, 180, 0.15);
}

.about-card h3 {
  color: var(--primary-color);
  margin-bottom: 1rem;
  font-size: 1.3rem;
}

.project-list, .activity-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.project-list li, .activity-list li {
  margin-bottom: 1rem;
  padding: 0.5rem;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.05);
}

.project-title {
  font-weight: bold;
  color: var(--secondary-color);
  margin-right: 1rem;
}

.project-tech {
  font-size: 0.9rem;
  color: var(--accent-color);
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
  background: rgba(255, 215, 0, 0.1);
}

.project-desc {
  display: block;
  font-size: 0.9rem;
  color: var(--text-secondary);
  margin-top: 0.3rem;
}

@media (max-width: 768px) {
  .about-section {
    padding: 2rem 1rem;
  }
  
  .about-content {
    grid-template-columns: 1fr;
  }
}
</style>