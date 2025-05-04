<template>
  <div class="birthday-animation-container">
    <!-- 头像容器 -->
    <div
      v-motion
      class="avatar"
      :initial="{ scale: 1 }"
      :enter="{ scale: 1.05 }"
      :transition="{
        duration: 1.5,
        repeat: Infinity,
        repeatType: 'reverse',
        ease: 'easeInOut'
      }"
      @click="triggerCelebration"
    >
      <img src="/images/avatar.png" alt="头像" class="avatar-img" />
      <div
        v-motion
        class="avatar-glow"
        :initial="{ scale: 1, opacity: 0.5 }"
        :enter="{ scale: 1.2, opacity: 0.2 }"
        :transition="{
          duration: 2,
          repeat: Infinity,
          repeatType: 'reverse',
          ease: 'easeInOut'
        }"
      />
    </div>

    <!-- 提示文字 -->
    <p class="click-prompt">点击头像有惊喜！</p>

    <!-- Emoji 容器 -->
    <div class="effects-container" ref="effectsContainer">
      <div
        v-for="n in 25"
        :key="'emoji-' + n"
        v-motion
        class="emoji"
        :initial="{ scale: 0, opacity: 0 }"
        :animate="isAnimating ? emojiVariants.visible(n) : {}"
      >
        {{ getRandomEmoji() }}
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

// 控制动画状态
const isAnimating = ref(false)

// 生日主题 Emoji 列表
const birthdayEmojis = ['🎂', '🎉', '🎁', '🎈', '🥳', '🎊', '🍰', '⭐']

// 获取随机 Emoji
const getRandomEmoji = () => {
  const randomIndex = Math.floor(Math.random() * birthdayEmojis.length)
  return birthdayEmojis[randomIndex]
}

// Emoji 动画变体
const emojiVariants = {
  visible: (i: number) => ({
    scale: [0, 1.5, 0.5],
    opacity: [0, 1, 0],
    y: [0, window.innerHeight * 0.6],
    x: [(i - 12) * 20, (i - 12) * 40],
    rotate: Math.random() * 360,
    transition: {
      duration: 2 + Math.random(),
      ease: 'easeOut',
      delay: i * 0.04
    }
  })
}

// 触发庆祝动画
const triggerCelebration = () => {
  isAnimating.value = true
  // 动画结束后重置状态，以便可以再次触发
  setTimeout(() => {
    isAnimating.value = false
  }, 3000)
}
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

.click-prompt {
  margin-top: 0.8rem;
  font-size: 0.9rem;
  color: #ffffff;
  text-shadow: 0 0 10px rgba(255, 105, 180, 0.8);
  font-weight: bold;
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
  pointer-events: none;
  z-index: 5;
}

.emoji {
  position: absolute;
  font-size: 24px;
  opacity: 0;
  user-select: none;
}

@media (max-width: 768px) {
  .avatar {
    width: 70px;
    height: 70px;
  }
  .emoji {
    font-size: 20px;
  }
}
</style>