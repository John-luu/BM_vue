<template>
  <div class="login-container">
    <!-- 使用背景组件 -->
    <BackgroundElements
      :circle-color="'rgba(255, 255, 255, 0.1)'"
      :element-color="'rgba(255, 255, 255, 0.15)'"
    />

    <!-- 主登录卡片 -->
    <div class="login-card">
      <!-- 左侧表单区域 -->
      <div class="card-form">
        <div class="form-header">
          <h2>用户登录</h2>
          <p>使用您的学工号登录系统</p>
        </div>

        <form @submit.prevent="handleSubmit">
          <!-- 学工号输入 -->
          <InputField
            v-model="number"
            label="学工号"
            placeholder="请输入学工号"
            :error="numberError"
            icon="id"
            :required="true"
          />

          <!-- 密码输入 -->
          <InputField
            v-model="password"
            label="密码"
            :type="showPassword ? 'text' : 'password'"
            placeholder="请输入密码"
            :error="passwordError"
            icon="password"
            :required="true"
            :show-password="showPassword"
            @toggle-password="showPassword = !showPassword"
          >
            <template #suffix>
              <a
                href="#"
                class="forgot-password"
                @click.prevent="handleForgotPassword"
                >忘记密码？</a
              >
            </template>
          </InputField>

          <!-- 记住我选项 -->
          <div class="remember-me">
            <label class="checkbox-label">
              <input
                type="checkbox"
                v-model="rememberMe"
                class="checkbox-input"
              />
              <span class="checkbox-custom"></span>
              <span class="checkbox-text">记住我</span>
            </label>
          </div>

          <!-- 登录按钮 -->
          <button
            type="submit"
            :disabled="loading"
            class="login-btn"
            :class="{ loading: loading }"
          >
            <span v-if="loading" class="loading-spinner"></span>
            <span v-else class="button-text">登录系统 →</span>
          </button>

          <!-- 注册链接 -->
          <div class="register-prompt">
            <p>
              还没有账户？
              <a @click="signUp_asd" class="register-link">立即注册</a>
            </p>
          </div>
        </form>
      </div>

      <!-- 右侧品牌卡片 -->
      <div class="card-brand">
        <BrandCard
          :subtitle="'智慧学习空间 · 高效资源管理'"
          :welcome-text="'欢迎回来，请登录您的账户'"
        />
      </div>
    </div>
  </div>
</template>

<script>
// 使用相对路径导入组件
import BackgroundElements from "@/components/auth/BackgroundElements.vue";
import BrandCard from "@/components/auth/BrandCard.vue";
import InputField from "@/components/auth/InputField.vue";
import request from "@/req";
import { Toast } from "vant";

export default {
  name: "Login",
  components: {
    BackgroundElements,
    BrandCard,
    InputField,
  },
  data() {
    return {
      number: "",
      password: "",
      rememberMe: true,
      showPassword: false,
      loading: false,
      numberError: "",
      passwordError: "",
    };
  },
  mounted() {
    const savedNumber = localStorage.getItem("rememberedNumber");
    if (savedNumber) {
      this.number = savedNumber;
    }

    if (process.env.NODE_ENV === "development") {
      this.number = "999";
      this.password = "123456789";
    }
  },
  methods: {
    signUp_asd() {
      this.$router.replace({ path: "/register" });
    },

    handleForgotPassword() {
      Toast("请前往图书馆服务台或联系管理员重置密码");
    },

    fillTestStudent() {
      this.number = "2023001";
      this.password = "student123";
      Toast("测试学生账户已填充");
    },

    fillTestTeacher() {
      this.number = "1001";
      this.password = "teacher123";
      Toast("测试教师账户已填充");
    },

    validateForm() {
      let isValid = true;

      this.numberError = "";
      this.passwordError = "";

      if (!this.number.trim()) {
        this.numberError = "请输入学工号";
        isValid = false;
      } else if (!/^\d+$/.test(this.number)) {
        this.numberError = "学工号应为数字";
        isValid = false;
      }

      if (!this.password) {
        this.passwordError = "请输入密码";
        isValid = false;
      } else if (this.password.length < 6) {
        this.passwordError = "密码长度至少为6位";
        isValid = false;
      }

      return isValid;
    },

    handleSubmit() {
      if (!this.validateForm()) {
        return;
      }

      this.loading = true;

      if (this.rememberMe) {
        localStorage.setItem("rememberedNumber", this.number);
      } else {
        localStorage.removeItem("rememberedNumber");
      }

      request
        .post("/public/login", {
          number: this.number,
          password: this.password,
        })
        .then((res) => {
          this.loading = false;

          if (res.code === 200) {
            Toast.success({
              message: "登录成功！",
              duration: 1500,
              onClose: () => {
                localStorage.setItem("user", JSON.stringify(res.user));
                if (res.user.type === 0) {
                  this.$router.replace("/student/forum");
                } else if (res.user.type === 1) {
                  this.$router.replace("/Teacher/seat");
                }
              },
            });
          } else {
            Toast.fail("账户或密码错误");
            this.passwordError = "账户或密码错误";
          }
        })
        .catch((err) => {
          this.loading = false;
          Toast.fail("网络错误，请稍后重试");
        });
    },
  },
};
</script>

<style scoped>
/* 保持原有的样式，但调整左右布局 */
.login-container {
  min-height: 100vh;
  width: 100%;
  background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 50%, #90caf9 100%);
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
  position: relative;
  overflow: hidden;
  font-family: "Segoe UI", "Microsoft YaHei", sans-serif;
}

/* 主登录卡片 - 表单在左，品牌在右 */
.login-card {
  width: 1000px;
  height: 650px;
  background: white;
  border-radius: 24px;
  box-shadow: 0 20px 60px rgba(66, 133, 244, 0.15);
  display: flex;
  overflow: hidden;
  z-index: 1;
  position: relative;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.3);
}

/* 左侧表单区域 */
.card-form {
  width: 55%;
  padding: 60px 70px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

/* 右侧品牌区域 */
.card-brand {
  width: 45%;
}

.form-header {
  margin-bottom: 40px;
}

.form-header h2 {
  font-size: 36px;
  color: #2d3748;
  margin-bottom: 10px;
  font-weight: 700;
}

.form-header p {
  color: #718096;
  font-size: 16px;
}

/* 记住我样式 */
.remember-me {
  display: flex;
  align-items: center;
  margin: 15px 0 25px;
}

.checkbox-label {
  display: flex;
  align-items: center;
  cursor: pointer;
  font-size: 15px;
  color: #4a5568;
}

.checkbox-input {
  display: none;
}

.checkbox-custom {
  width: 20px;
  height: 20px;
  border: 2px solid #cbd5e0;
  border-radius: 6px;
  margin-right: 10px;
  position: relative;
  transition: all 0.3s ease;
}

.checkbox-input:checked + .checkbox-custom {
  background-color: #4a90e2;
  border-color: #4a90e2;
}

.checkbox-input:checked + .checkbox-custom:after {
  content: "";
  position: absolute;
  top: 2px;
  left: 6px;
  width: 6px;
  height: 10px;
  border: solid white;
  border-width: 0 2px 2px 0;
  transform: rotate(45deg);
}

/* 登录按钮 */
.login-btn {
  width: 100%;
  padding: 18px;
  border-radius: 12px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
  transition: all 0.3s ease;
  font-family: inherit;
  border: none;
  background: linear-gradient(135deg, #4a90e2 0%, #2b6cb0 100%);
  color: white;
  box-shadow: 0 4px 15px rgba(74, 144, 226, 0.3);
  margin-top: 10px;
  position: relative;
}

.login-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(74, 144, 226, 0.4);
}

.login-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.login-btn.loading {
  cursor: wait;
}

.loading-spinner {
  width: 20px;
  height: 20px;
  border: 3px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  border-top-color: white;
  animation: spin 1s ease-in-out infinite;
}

/* 注册提示 */
.register-prompt {
  text-align: center;
  margin: 25px 0;
  color: #718096;
  font-size: 15px;
}

.register-link {
  color: #4a90e2;
  text-decoration: none;
  font-weight: 600;
  cursor: pointer;
  transition: color 0.3s ease;
}

.register-link:hover {
  color: #2b6cb0;
  text-decoration: underline;
}

/* 快速登录 */
.quick-login {
  margin-top: 30px;
}

.divider {
  display: flex;
  align-items: center;
  color: #a0aec0;
  font-size: 14px;
  margin-bottom: 20px;
}

.divider:before,
.divider:after {
  content: "";
  flex: 1;
  height: 1px;
  background: #e2e8f0;
}

.divider span {
  padding: 0 15px;
}

.quick-login-buttons {
  display: flex;
  gap: 15px;
}

.quick-btn {
  flex: 1;
  padding: 14px;
  border-radius: 10px;
  font-size: 14px;
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 8px;
  transition: all 0.3s ease;
  font-family: inherit;
  border: 2px solid #e2e8f0;
  background: white;
  color: #4a90e2;
  font-weight: 500;
}

.quick-btn:hover {
  border-color: #4a90e2;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(74, 144, 226, 0.15);
}

.quick-icon {
  font-size: 18px;
  line-height: 1;
}

/* 忘记密码链接 */
.forgot-password {
  position: absolute;
  right: 16px;
  color: #4a90e2;
  font-size: 14px;
  text-decoration: none;
  transition: color 0.3s ease;
  background: none;
  border: none;
  cursor: pointer;
  font-family: inherit;
  padding: 5px;
}

.forgot-password:hover {
  color: #2b6cb0;
  text-decoration: underline;
}

/* 底部信息 */
.form-footer {
  margin-top: 40px;
  text-align: center;
  color: #a0aec0;
  font-size: 13px;
  border-top: 1px solid #e2e8f0;
  padding-top: 20px;
}

/* 响应式设计 */
@media (max-width: 1100px) {
  .login-card {
    width: 95%;
    height: auto;
  }
}

@media (max-width: 900px) {
  .login-card {
    flex-direction: column-reverse; /* 在小屏幕上表单在上，品牌在下 */
    height: auto;
  }

  .card-brand,
  .card-form {
    width: 100%;
  }

  .card-form {
    padding: 40px 30px;
  }

  .quick-login-buttons {
    flex-direction: column;
  }
}

@media (max-width: 576px) {
  .card-form {
    padding: 30px 20px;
  }

  .form-header h2 {
    font-size: 28px;
  }
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

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
