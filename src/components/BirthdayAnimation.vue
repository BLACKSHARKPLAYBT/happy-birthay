<template>
  <div class="birthday-animation-container">
    <!-- 头像保持在右下角，作为触发器 -->
    <div class="avatar" ref="avatar" @click="triggerCelebration">
      <img src="/images/avatar.png" alt="头像" class="avatar-img" />
      <div class="avatar-glow"></div>
    </div>
    <!-- 新增：提示文字 -->
    <p class="click-prompt" style="color: white;">点击头像有惊喜！</p>

    <!-- 效果容器，直接放在这里 -->
    <div class="effects-container" ref="effectsContainer">
      <!-- 使用 emoji 元素 -->
      <div class="emoji" v-for="n in 25" :key="'emoji-' + n"></div>
    </div>
  </div>
</template>

<script setup lang="ts">
  import { ref, onMounted } from 'vue'
import anime from 'animejs/lib/anime.min.js'

const avatar = ref<HTMLElement | null>(null)
const effectsContainer = ref<HTMLElement | null>(null)

// 生日主题 Emoji 列表
const birthdayEmojis = ['🎂', '🎉', '🎁', '🎈', '🥳', '🎊', '🍰', '⭐'];

// 头像的持续呼吸效果
const animateAvatarIdle = () => {
  if (!avatar.value) return
  anime.remove(avatar.value) // 确认这里能正确调用
  anime({
    targets: avatar.value,
    scale: [1, 1.05],
    rotate: [-3, 3],
    duration: 1500,
    direction: 'alternate',
    loop: true,
    easing: 'easeInOutQuad'
  })
  anime({
    targets: '.avatar-glow',
    scale: [1, 1.2],
    opacity: [0.5, 0.2],
    duration: 2000,
    loop: true,
    direction: 'alternate',
    easing: 'easeInOutSine'
  })
}

// 触发庆祝动画 - Emoji 掉落
const triggerCelebration = () => {
  console.log('triggerCelebration called');

  // 停止头像的闲置动画，播放点击效果
  if (avatar.value) { // 增加检查确保 avatar.value 存在
    anime.remove(avatar.value) // 确认这里能正确调用
    anime({
      targets: avatar.value,
      scale: [1, 1.2, 1],
      rotate: [-10, 10, 0],
      duration: 500,
      easing: 'easeInOutBack',
      complete: () => {
        animateAvatarIdle() // 点击动画结束后恢复闲置动画
      }
    });
  } else {
    console.error('Avatar element not found when trying to remove animation');
  }

  // --- Emoji 掉落动画 ---
  const containerEl = effectsContainer.value
  if (!containerEl) {
    console.error('effectsContainer not found'); // <-- 添加日志
    return
  }
  console.log('effectsContainer found:', containerEl); // <-- 添加日志

  const avatarRect = avatar.value?.getBoundingClientRect()
  const avatarSize = avatarRect ? Math.max(avatarRect.width, avatarRect.height) : 90
  const effectRangeX = avatarSize * 1.2 // X轴散开范围
  const fallDistance = window.innerHeight * 0.6; // Emoji 掉落距离

  const emojiElements = containerEl.querySelectorAll('.emoji');
  console.log(`Found ${emojiElements.length} emoji elements`); // <-- 添加日志
  if (emojiElements.length === 0) {
    console.warn('No .emoji elements found inside effectsContainer'); // <-- 添加日志
  }

  // 为每个 emoji 元素设置随机 Emoji 内容
  emojiElements.forEach(el => {
    (el as HTMLElement).textContent = birthdayEmojis[anime.random(0, birthdayEmojis.length - 1)];
  });

  // 重置效果元素的位置和透明度
  anime.set(emojiElements, {
    translateX: () => anime.random(-effectRangeX / 2, effectRangeX / 2),
    translateY: () => anime.random(-avatarSize * 0.5, avatarSize * 0.2),
    scale: 0,
    opacity: 1,
    rotate: 0
  });
  console.log('Emoji elements reset'); // <-- 添加日志

  // Emoji 掉落动画
  anime({
    targets: emojiElements,
    translateY: [`+=${anime.random(-50, 0)}`, `+=${fallDistance + anime.random(0, 100)}`],
    translateX: [`+=${anime.random(-effectRangeX, effectRangeX)}`],
    scale: [0, () => anime.random(1, 1.5), 0.5],
    opacity: [1, 0.8, 0],
    rotate: () => anime.random(-180, 180),
    duration: () => anime.random(2000, 3500),
    delay: anime.stagger(40),
    easing: 'easeInQuad',
    begin: () => console.log('Emoji animation started'), // <-- 添加日志
    complete: () => console.log('Emoji animation completed') // <-- 添加日志
  })
}

onMounted(() => {
  animateAvatarIdle()
})

</script>

<style scoped>
.birthday-animation-container {
  position: relative;
  display: inline-block;
  text-align: center; /* 让提示文字居中 */
}

.avatar {
  position: relative;
  margin: 0 auto; /* 保持头像居中 */
  width: 90px;
  height: 90px;
  border-radius: 50%;
  overflow: hidden;
  box-shadow: 0 4px 20px rgba(255, 105, 180, 0.3);
  background: rgba(255, 255, 255, 0.25);
  backdrop-filter: blur(10px);
  border: 2px solid rgba(255, 105, 180, 0.4);
  cursor: pointer;
  transition: all 0.3s ease;
  z-index: 2; /* 确保在效果元素之上 */
}

/* 新增：提示文字样式 */
.click-prompt {
  margin-top: 0.8rem;
  font-size: 0.9rem;
  color: white; /* 改为白色以适应深色背景 */
  /* text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5); */ /* 可以添加轻微阴影增加可读性 */
}

.avatar-glow {
  position: absolute;
  top: -20%;
  left: -20%;
  width: 140%;
  height: 140%;
  background: radial-gradient(circle, rgba(255,105,180,0.3) 0%, rgba(255,105,180,0) 70%);
  pointer-events: none;
}

.avatar:hover {
   transform: scale(1.1);
  border-color: rgba(255, 105, 180, 0.6);
  box-shadow: 0 6px 25px rgba(255, 105, 180, 0.5);
}

.avatar-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.effects-container {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  /* width: 1px; */ /* 移除 */
  /* height: 1px; */ /* 移除 */
  pointer-events: none;
  z-index: 5; /* 保持较高值 */
}

/* 新增：Emoji 元素样式 */
.emoji {
  position: absolute;
  font-size: 24px; /* 调整 Emoji 大小 */
  opacity: 0;
  user-select: none; /* 防止用户选中 Emoji */
}

/* 移除旧的 .confetti, .sparkle, .heart, .balloon 样式 */

/* 响应式调整 */
@media (max-width: 768px) {
  .avatar {
    width: 70px;
    height: 70px;
  }
  .emoji {
    font-size: 20px; /* 移动端适当减小 Emoji 大小 */
  }
}
</style>