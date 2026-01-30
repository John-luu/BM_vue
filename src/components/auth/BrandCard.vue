<template>
  <div class="brand-card">
    <div class="card-decoration">
      <div class="decoration-content">
        <div class="library-icon">
          <div class="book-icon">📚</div>
        </div>
        <h3>图书馆座位预约系统</h3>
        <p class="subtitle">{{ subtitle }}</p>

        <div class="features-list">
          <div v-for="feature in features" :key="feature" class="feature-item">
            <div class="check-icon">✓</div>
            <span>{{ feature }}</span>
          </div>
        </div>

        <div class="additional-info">
          <slot name="additional-info">
            <p class="welcome-text">{{ welcomeText }}</p>
          </slot>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "BrandCard",
  props: {
    subtitle: {
      type: String,
      default: "智慧学习空间 · 高效资源管理",
    },
    features: {
      type: Array,
      default: () => ["智能预约", "实时状态", "学习统计"],
    },
    welcomeText: {
      type: String,
      default: "欢迎使用图书馆座位预约系统",
    },
    showStats: {
      type: Boolean,
      default: true,
    },
    userCount: {
      type: [String, Number],
      default: "2,847",
    },
  },
};
</script>

<style scoped>
.brand-card {
  width: 100%;
  height: 100%;
}

.card-decoration {
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #4a90e2 0%, #2b6cb0 100%);
  padding: 50px 40px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  color: white;
  position: relative;
  overflow: hidden;
}

.card-decoration:before {
  content: "";
  position: absolute;
  top: -50%;
  right: -50%;
  width: 100%;
  height: 200%;
  background: radial-gradient(
    circle,
    rgba(255, 255, 255, 0.1) 1px,
    transparent 1px
  );
  background-size: 30px 30px;
  opacity: 0.3;
  animation: float 20s linear infinite;
}

.library-icon {
  margin-bottom: 30px;
  display: flex;
  justify-content: center;
}

.book-icon {
  font-size: 60px;
  line-height: 1;
}

.decoration-content h3 {
  font-size: 28px;
  margin-bottom: 15px;
  font-weight: 600;
  text-align: center;
  line-height: 1.3;
}

.subtitle {
  font-size: 18px;
  opacity: 0.9;
  margin-bottom: 40px;
  text-align: center;
  font-weight: 300;
}

.features-list {
  margin: 40px 0;
}

.feature-item {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
  font-size: 16px;
}

.check-icon {
  width: 24px;
  height: 24px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 15px;
  font-weight: bold;
}

.feature-item span {
  opacity: 0.9;
}

.additional-info {
  margin-top: 30px;
}

.welcome-text {
  text-align: center;
  font-size: 16px;
  opacity: 0.9;
  margin-bottom: 10px;
}

.stats {
  margin-top: 20px;
  text-align: center;
}

.stats-text {
  font-size: 14px;
  opacity: 0.7;
}

.stats-text strong {
  color: #ffd700;
}

@keyframes float {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-20px);
  }
}
</style>
