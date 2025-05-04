<template>
  <div class="portfolio-app">
    <!-- 导航栏 -->
    <header class="header">
      <div class="logo">生日项目</div>
      <nav class="nav">
        <ul>
          <li><a href="#section1">主页</a></li>
          <li><a href="#section2">祝福</a></li>
          <li><a href="#section3">愿望</a></li>
        </ul>
      </nav>
    </header>

    <!-- 3D场景组件 (暂时注释掉) -->
    <ThreeScene />

    <!-- 主要内容区 -->
    <main class="content-sections">
      <!-- 标题区域 -->
      <section class="hero-section">
        <div class="hero-content">
          <h1 class="main-title">生日快乐</h1>
          <p class="subtitle">一个充满欢乐与惊喜的特别日子</p>
          <div class="birthday-message" ref="birthdayMessage"></div>
        </div>
      </section>

      <!-- 项目展示区域 -->
      <section class="projects-section">
        <div class="section-header">
          <h2>庆祝</h2>
          <p>最后更新于 {{ new Date().toISOString().split('T')[0] }}</p>
        </div>

        <!-- 项目卡片 -->
        <!-- 建议：如果需要单独控制每个卡片的动画，考虑将 project-card 也做成组件 -->
        <div class="project-grid">
          <div class="project-card" id="section1">
            <div class="project-image">
              <img :src="cakeImg" alt="生日蛋糕" class="card-img" />
            </div>
            <div class="project-info">
              <h3>生日快乐！</h3>
              <p>愿这特别的日子充满欢乐与惊喜</p>
              <div class="project-meta">
                <div class="meta-item">
                  <span class="meta-label">日期</span>
                  <span class="meta-value">{{ new Date().getFullYear() }}.{{ new Date().getMonth() + 1 }}</span>
                </div>
                <div class="meta-item">
                  <span class="meta-label">类型</span>
                  <span class="meta-value">庆祝</span>
                </div>
              </div>
            </div>
          </div>

          <div class="project-card" id="section2">
            <div class="project-image">
              <img :src="wishImg" alt="生日祝福" class="card-img" />
            </div>
            <div class="project-info">
              <h3>愿永远快乐</h3>
              <p>愿你的每一天都如阳光般灿烂</p>
              <div class="project-meta">
                <div class="meta-item">
                  <span class="meta-label">心情</span>
                  <span class="meta-value">欢乐</span>
                </div>
                <div class="meta-item">
                  <span class="meta-label">类型</span>
                  <span class="meta-value">祝福</span>
                </div>
              </div>
            </div>
          </div>

          <div class="project-card" id="section3">
            <div class="project-image">
              <img :src="dreamImg" alt="生日愿望" class="card-img" />
            </div>
            <div class="project-info">
              <h3>生日愿望成真</h3>
              <p>让所有美好的祝福都实现</p>
              <div class="project-meta">
                <div class="meta-item">
                  <span class="meta-label">梦想</span>
                  <span class="meta-value">无限</span>
                </div>
                <div class="meta-item">
                  <span class="meta-label">类型</span>
                  <span class="meta-value">愿望</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>
      <AboutMe />
    </main>

    <!-- 新增：生日动画区域 -->
    <section class="birthday-section">
      <BirthdayAnimation />
    </section>

    <!-- 装饰元素 -->
    <div class="decorations">
      <div class="decoration" v-for="n in 10" :key="n"></div> 
    </div>

    <!-- 粒子效果 -->
    <div class="particles"></div>

    <!-- 音乐控制 -->
    <audio ref="bgMusic" loop>
      <source src="/music/happy-birthday.mp3" type="audio/mpeg">
    </audio>
    <!-- 固定底部元素容器 (只保留音乐按钮) -->
    <div class="fixed-bottom-elements">
      <button class="music-toggle" @click="toggleMusic">
        <span>{{ isPlaying ? '🎵' : '🔇' }}</span>
      </button>
      <!-- BirthdayAnimation 已移出 -->
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, onUnmounted } from 'vue'
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import AboutMe from './components/AboutMe.vue'
// 导入新的场景组件 (名称和路径已修正)
import ThreeScene from './components/ThreeScene.vue'
import BirthdayAnimation from './components/BirthdayAnimation.vue'
// 导入图片 (修正路径)
import cakeImg from '/images/cake.png' // 使用 / 开头的绝对路径
import wishImg from '/images/wish.png'
import dreamImg from '/images/dream.png'
import decoration1 from '/images/decoration1.png' 
import decoration2 from '/images/decoration2.png'
import decoration3 from '/images/decoration3.png'



gsap.registerPlugin(ScrollTrigger)
// 保留 App.vue 相关的变量和函数
const bgMusic = ref<HTMLAudioElement | null>(null)
const isPlaying = ref(false)
const birthdayMessage = ref<HTMLElement | null>(null) // 保留，用于 anime.js
// 移除了 decorations 和 particlesContainer refs

// 音乐控制
const toggleMusic = () => {
  if (!bgMusic.value) return
  if (isPlaying.value) {
    bgMusic.value.pause()
  } else {
    bgMusic.value.play().catch(error => {
      console.error("Music play failed:", error);
    });
  }
  isPlaying.value = !isPlaying.value
}

// 假设这些动画函数仍然需要，并且定义在文件的其他地方或将被添加
const animateText = () => {
  if (!birthdayMessage.value) return;
  // 使用 anime.js 实现打字效果或其他文本动画
  const text = "祝你生日快乐，愿你的所有愿望都能成真！";
  let i = 0;
  const typing = () => {
    if (i < text.length && birthdayMessage.value) {
      birthdayMessage.value.innerHTML += text.charAt(i);
      i++;
      setTimeout(typing, 100); // 控制打字速度
    } else if (birthdayMessage.value) {
      // 添加闪烁光标
      birthdayMessage.value.innerHTML += '<span class="cursor">|</span>';
      gsap.to('.cursor', { opacity: 0, repeat: -1, yoyo: true, duration: 0.5, ease: 'power1.inOut' });
    }
  };
  typing();
};

const animateCards = () => {
  // 使用 GSAP ScrollTrigger 为卡片添加入场动画
  gsap.utils.toArray('.project-card').forEach((card, index) => {
    gsap.from(card as HTMLElement, {
      opacity: 0,
      y: 50,
      duration: 0.8,
      delay: index * 0.2, // staggered effect
      scrollTrigger: {
        trigger: card as HTMLElement,
        start: 'top 80%', // 当卡片顶部进入视口80%时触发
        toggleActions: 'play none none none', // 触发一次动画
      }
    });
  });
};

const animateDecorations = () => {
  // 使用 anime.js 为装饰元素添加随机漂浮动画 (使用修正后的 anime 对象)
  anime({
    targets: '.decoration',
    translateY: () => anime.random(-20, 20),
    translateX: () => anime.random(-15, 15),
    rotate: () => anime.random(-30, 30),
    scale: () => anime.random(0.8, 1.2),
    duration: () => anime.random(2000, 4000),
    easing: 'easeInOutSine',
    loop: true,
    direction: 'alternate',
    delay: anime.stagger(100) // anime.stagger 也是 anime 对象的方法
  });

  // 设置装饰元素的初始背景图片 (如果需要动态设置)
  // 注意：如果装饰图片导入了但未使用，需要在这里使用或移除导入
  const decorImages = [decoration1, decoration2, decoration3];
  document.querySelectorAll('.decoration').forEach((el, index) => {
    (el as HTMLElement).style.backgroundImage = `url(${decorImages[index % decorImages.length]})`;
    // 随机初始位置 (使用修正后的 anime 对象)
    (el as HTMLElement).style.top = `${anime.random(5, 95)}%`;
    (el as HTMLElement).style.left = `${anime.random(5, 95)}%`;
  });
};

// 在 script setup 中添加以下代码
onMounted(() => {
// 监听滚动事件，添加导航栏滚动效果
const header = document.querySelector('.header');
window.addEventListener('scroll', () => {
if (window.scrollY > 50) {
header?.classList.add('scrolled');
} else {
header?.classList.remove('scrolled');
}
});

  // 初始化非 3D 动画
  animateText();
  animateCards();
  animateDecorations();

  // 可以在这里添加其他 App.vue 特有的初始化逻辑
})

onUnmounted(() => {
  // 清理 GSAP ScrollTrigger
  ScrollTrigger.getAll().forEach(trigger => trigger.kill());
  // 停止 anime 动画 (使用修正后的 anime 对象)
  anime.remove('.decoration');
  anime.remove('.cursor');
  // 停止音乐
  if (bgMusic.value) {
    bgMusic.value.pause();
  }
})

  // 添加滚动视差效果
  window.addEventListener('scroll', () => {
    const scrolled = window.pageYOffset;
    const cards = document.querySelectorAll('.project-card');
    
    cards.forEach((card, index) => {
      const speed = 1 + index * 0.1;
      const yPos = -(scrolled * speed / 5);
      card.style.transform = `translateY(${yPos}px) rotateX(${yPos / 50}deg)`;
    });
  });
</script>
<style>
/* 在 :root 中添加字体变量 */
:root {
  /* 原有的颜色变量保持不变 */
  --primary-color: #FF69B4;     /* 主色调：亮粉色 */
  --secondary-color: #87CEEB;   /* 次要色：天蓝色 */
  --accent-color: #FFD700;      /* 强调色：金色 */
  --text-primary: #FFFFFF;      /* 主要文字：纯白色 */
  --text-secondary: #E0E0E0;    /* 次要文字：浅灰白色 */
  --text-accent: #FFB6C1;       /* 强调文字：浅粉色 */
  --background-primary: #FFF0F5; /* 主背景：淡粉色 */
  --background-secondary: #F0F8FF; /* 次要背景：淡蓝色 */
}


/* 修改渐变背景 */
.content-sections {
  background: linear-gradient(135deg, rgba(255, 240, 245, 0.3), rgba(240, 248, 255, 0.3));
}

/* 修改标题渐变 */
.main-title {
  background: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  font-weight: bold;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.subtitle {
  color: var(--text-primary);
  text-shadow: 0 0 8px rgba(255, 105, 180, 0.3);
  font-size: 1.2rem;
  margin-top: 1rem;
}

.project-info h3 {
  color: var(--text-primary);
  font-weight: bold;
  text-shadow: 0 0 8px rgba(255, 105, 180, 0.3);
}

.project-info p {
  color: var(--text-secondary);
  line-height: 1.6;
}

/* 修改导航栏样式 */
.header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 2rem;
  background: rgba(255, 192, 203, 0.15); /* 改为淡粉色半透明背景 */
  border-bottom: 1px solid rgba(255, 105, 180, 0.2); /* 减淡边框 */
  box-shadow: 0 4px 15px rgba(255, 105, 180, 0.1); /* 柔和的阴影 */
  backdrop-filter: blur(10px); /* 增加模糊效果 */
  transition: all 0.3s ease;
}

.header.scrolled {
  background: rgba(255, 192, 203, 0.25); /* 滚动时略微增加不透明度 */
  box-shadow: 0 4px 20px rgba(255, 105, 180, 0.15);
}

.nav ul {
  display: flex;
  gap: 2rem;
  list-style: none;
  margin: 0;
  padding: 0;
}

.nav a {
  color: var(--text-primary);
  text-decoration: none;
  font-weight: 500;
  position: relative;
  padding: 0.5rem 0;
}

.nav a:hover {
  color: var(--primary-color);
}

.nav a::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 2px;
  background: linear-gradient(90deg, var(--primary-color), var(--secondary-color));
  transform: scaleX(0);
  transform-origin: right;
  transition: transform 0.3s ease;
}

.nav a:hover::after {
  transform: scaleX(1);
  transform-origin: left;
}

/* Logo 样式 */
.logo {
  font-size: 1.5rem;
  font-weight: bold;
  background: linear-gradient(135deg, var(--primary-color), var(--accent-color));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  padding: 0.5rem 0;
}

/* 为了防止内容被固定导航栏遮挡，给主要内容添加上边距 */
.content-sections {
  padding-top: 80px; /* 根据导航栏高度调整 */
}

/* Logo 样式优化 */
.logo {
  font-size: 1.8rem;
  font-weight: bold;
  color: #FF1493; /* 直接使用纯色 */
  text-shadow: 2px 2px 4px rgba(255, 20, 147, 0.2); /* 添加文字阴影 */
  padding: 0.5rem 0;
  position: relative;
  z-index: 2;
  background: none; /* 移除渐变背景 */
  -webkit-text-fill-color: initial; /* 移除透明效果 */
}

/* 导航链接样式优化 */
.nav a {
  color: #FF1493; /* 更鲜艳的文字颜色 */
  font-weight: 600;
  text-shadow: none; /* 移除文字阴影以提高可读性 */
}

.nav a:hover {
  color: #FF69B4;
  text-shadow: 0 0 8px rgba(255, 105, 180, 0.3);
}

/* 音乐控制按钮样式优化 */
.music-toggle {
  position: fixed;
  bottom: 20px;
  right: 20px;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: rgba(255, 240, 245, 0.1);
  backdrop-filter: blur(8px);
  border: 2px solid rgba(255, 105, 180, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  cursor: pointer;
  transition: all 0.3s ease;
  z-index: 1000;
}

.music-toggle:hover {
  transform: scale(1.1) rotate(10deg); /* 添加旋转效果 */
  border-color: rgba(255, 20, 147, 0.7);
  background: rgba(255, 20, 147, 0.25);
  box-shadow: 0 4px 20px rgba(255, 20, 147, 0.3);
}

/* 项目网格布局样式 */
.project-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
  padding: 2rem;
  max-width: 1400px;
  margin: 0 auto;
}

/* 响应式布局 */
@media (max-width: 768px) {
  .project-grid {
    grid-template-columns: 1fr; /* 在小屏幕上改为单列 */
    padding: 1rem;
  }

  .project-card {
    margin: 1rem 0;
  }

  .music-toggle {
    width: 40px; /* 在移动端稍微缩小按钮 */
    height: 40px;
    font-size: 20px;
    bottom: 15px;
    right: 15px;
  }
}

/* 确保卡片样式完整 */
.project-card {
  background: rgba(255, 240, 245, 0.1);
  border: 1px solid rgba(255, 105, 180, 0.2);
  backdrop-filter: blur(4px);
  border-radius: 24px;
  overflow: hidden;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  transform-style: preserve-3d;
  perspective: 1000px;
  padding-bottom: 1rem; /* 添加内边距 */
}

.project-info {
  padding: 1.5rem;
}

.project-meta {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 1rem;
  margin-top: 1rem;
}

.meta-item {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}
/* 固定底部元素容器 */
.fixed-bottom-elements {
  position: fixed;
  bottom: 2rem;
  right: 2rem;
  display: flex;
  align-items: flex-end;
  gap: 1.5rem;
  z-index: 999;
}

/* 移除或注释掉 .fixed-bottom-elements 容器样式 */
/*
.fixed-bottom-elements {
  position: fixed;
  bottom: 2rem;
  right: 2rem;
  display: flex;
  align-items: flex-end;
  gap: 1.5rem;
  z-index: 999;
}
*/

/* 音乐控制按钮样式优化 - 恢复 fixed 定位 */
.music-toggle {
  /* position: static; */ /* 移除 static 定位 */
  position: fixed; /* 恢复 fixed 定位 */
  bottom: 20px;
  right: 20px;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: rgba(255, 240, 245, 0.1);
  backdrop-filter: blur(8px);
  border: 2px solid rgba(255, 105, 180, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  cursor: pointer;
  transition: all 0.3s ease;
  z-index: 1000;
}

.music-toggle:hover {
  transform: scale(1.1) rotate(10deg);
  border-color: rgba(255, 20, 147, 0.7);
  background: rgba(255, 20, 147, 0.25);
  box-shadow: 0 4px 20px rgba(255, 20, 147, 0.3);
}

@media (max-width: 768px) {
  .fixed-bottom-elements {
    bottom: 1rem;
    right: 1rem;
    gap: 1rem;
  }

  .music-toggle {
    width: 40px;
    height: 40px;
    font-size: 20px;
  }
}


/* 新增：为生日动画区域添加样式 */
.birthday-section {
  padding: 4rem 0; /* 上下留白 */
  text-align: center; /* 使内部 inline-block 或 inline 元素居中 */
  position: relative; /* 为可能的绝对定位子元素提供上下文 */
  background-color: rgba(255, 255, 255, 0.05); /* 可选：添加一点背景区分 */
}

.fixed-bottom-elements {
  position: fixed;
  bottom: 1rem;
  right: 1rem; /* 只剩下音乐按钮，可以放在右下角 */ 
  z-index: 1000;
  display: flex;
  gap: 1rem;
}
.meta-label {
  color: var(--text-accent);
  font-size: 0.9rem;
  font-weight: 500;
}

.meta-value {
  color: var(--text-primary);
  font-weight: 600;
  text-shadow: 0 0 5px rgba(255, 105, 180, 0.2);
}
</style>
