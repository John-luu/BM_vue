<template>
  <div class="register-container">
    <!-- 使用背景组件 -->
    <BackgroundElements
      :circle-color="'rgba(255, 255, 255, 0.1)'"
      :element-color="'rgba(255, 255, 255, 0.15)'"
    />
    <!-- 主注册卡片 -->
    <div class="register-card">
      <!-- 左侧表单区域 - 添加滚动容器 -->
      <div class="card-form-wrapper">
        <div class="card-form">
          <div class="form-header">
            <h2>注册账号</h2>
            <p>创建您的图书馆账户</p>
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

            <!-- 姓名输入 -->
            <InputField
              v-model="username"
              label="姓名"
              placeholder="请输入姓名"
              :error="usernameError"
              icon="user"
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
              hint="密码长度至少为6位"
              :show-password="showPassword"
              @toggle-password="showPassword = !showPassword"
            />

            <!-- 用户类型选择 -->
            <div class="user-type-selector">
              <p class="user-type-label">选择您的身份</p>
              <div class="type-options">
                <div
                  class="type-option"
                  :class="{ active: type === '学生' }"
                  @click="type = '学生'"
                >
                  <div class="option-icon">🎓</div>
                  <p>学生</p>
                </div>
                <div
                  class="type-option"
                  :class="{ active: type === '老师' }"
                  @click="type = '老师'"
                >
                  <div class="option-icon">👨‍🏫</div>
                  <p>老师</p>
                </div>
              </div>
            </div>

            <!-- 协议声明 -->
            <div class="agreement">
              <label class="checkbox-label">
                <input
                  type="checkbox"
                  v-model="agreement"
                  class="checkbox-input"
                />
                <span class="checkbox-custom"></span>
                <span class="checkbox-text"
                  >我已阅读并同意 <a href="#">服务条款</a> 和
                  <a href="#">隐私政策</a></span
                >
              </label>
              <div v-if="agreementError" class="error-message">
                {{ agreementError }}
              </div>
            </div>

            <!-- 注册按钮 -->
            <button
              type="submit"
              :disabled="loading"
              class="register-btn"
              :class="{ loading: loading }"
            >
              <span v-if="loading" class="loading-spinner"></span>
              <span v-else class="button-text">注册账号 →</span>
            </button>

            <!-- 登录链接 -->
            <div class="login-prompt">
              <p>
                已有账户？
                <a @click="$router.replace('/login')" class="login-link"
                  >立即登录</a
                >
              </p>
            </div>
          </form>

          <!-- 底部信息 -->
          <div class="form-footer">
            <p>注册即代表您同意遵守图书馆使用规定</p>
          </div>
        </div>
      </div>

      <!-- 右侧品牌卡片 -->
      <div class="card-brand">
        <BrandCard
          :subtitle="'欢迎加入我们的智慧图书馆'"
          :features="['智能预约', '座位管理', '学习统计']"
          :welcome-text="'开始您的智慧学习之旅'"
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
  name: "Register",
  components: {
    BackgroundElements,
    BrandCard,
    InputField,
  },
  data() {
    return {
      type: "学生",
      number: "",
      username: "",
      password: "",
      agreement: false,
      showPassword: false,
      loading: false,
      numberError: "",
      usernameError: "",
      passwordError: "",
      agreementError: "",
    };
  },
  methods: {
    validateForm() {
      let isValid = true;

      this.numberError = "";
      this.usernameError = "";
      this.passwordError = "";
      this.agreementError = "";

      // 验证学工号
      if (!this.number.trim()) {
        this.numberError = "请输入学工号";
        isValid = false;
      } else if (!/^\d+$/.test(this.number)) {
        this.numberError = "学工号应为数字";
        isValid = false;
      }

      // 验证姓名
      if (!this.username.trim()) {
        this.usernameError = "请输入姓名";
        isValid = false;
      } else if (this.username.length < 2) {
        this.usernameError = "姓名至少2个字符";
        isValid = false;
      }

      // 验证密码
      if (!this.password) {
        this.passwordError = "请输入密码";
        isValid = false;
      } else if (this.password.length < 6) {
        this.passwordError = "密码长度至少为6位";
        isValid = false;
      }

      // 验证协议
      if (!this.agreement) {
        this.agreementError = "请阅读并同意服务条款和隐私政策";
        isValid = false;
      }

      return isValid;
    },

    handleSubmit() {
      if (!this.validateForm()) {
        return;
      }

      this.loading = true;

      request
        .post("/public/register", {
          number: this.number,
          username: this.username,
          password: this.password,
          type: this.type === "学生" ? 0 : 1,
        })
        .then((res) => {
          this.loading = false;

          Toast.success({
            message: "注册成功！",
            duration: 1500,
            onClose: () => {
              this.$router.replace("/login");
            },
          });
        })
        .catch((err) => {
          this.loading = false;

          if (err.response && err.response.data) {
            const errorMsg = err.response.data.message || "注册失败";
            Toast.fail(errorMsg);

            // 根据错误类型设置对应的错误提示
            if (errorMsg.includes("学工号")) {
              this.numberError = errorMsg;
            } else if (errorMsg.includes("用户名")) {
              this.usernameError = errorMsg;
            }
          } else {
            Toast.fail("网络错误，请稍后重试");
          }
        });
    },
  },
};
</script>

<style scoped>
.register-container {
  min-height: 100vh;
  width: 100%;
  background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 50%, #90caf9 100%);
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
  position: relative;
  overflow: auto;
  font-family: "Segoe UI", "Microsoft YaHei", sans-serif;
}

/* 主卡片 - 表单在左，品牌在右 */
.register-card {
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

/* 左侧表单包装器 - 添加滚动功能 */
.card-form-wrapper {
  width: 55%;
  height: 100%;
  overflow-y: auto;
  overflow-x: hidden;
  position: relative;
}

/* 自定义滚动条样式 */
.card-form-wrapper::-webkit-scrollbar {
  width: 6px;
}

.card-form-wrapper::-webkit-scrollbar-track {
  background: rgba(0, 0, 0, 0.05);
  border-radius: 3px;
}

.card-form-wrapper::-webkit-scrollbar-thumb {
  background: rgba(74, 144, 226, 0.5);
  border-radius: 3px;
}

.card-form-wrapper::-webkit-scrollbar-thumb:hover {
  background: rgba(74, 144, 226, 0.7);
}

/* 左侧表单区域 */
.card-form {
  width: 100%;
  min-height: 100%;
  padding: 20px 70px;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  box-sizing: border-box;
}

/* 右侧品牌区域 */
.card-brand {
  width: 45%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.form-header {
  margin-bottom: 30px;
  flex-shrink: 0;
}

.form-header h2 {
  font-size: 36px;
  color: #2d3748;
  margin-bottom: 5px;
  font-weight: 700;
}

.form-header p {
  color: #718096;
  font-size: 16px;
}

/* 用户类型选择 */
.user-type-selector {
  margin-bottom: 30px;
  flex-shrink: 0;
}

.user-type-label {
  color: #4a5568;
  margin-bottom: 12px;
  font-size: 16px;
}

.type-options {
  display: flex;
  gap: 20px;
  flex-shrink: 0;
}

.type-option {
  flex: 1;
  border: 2px solid #e2e8f0;
  border-radius: 12px;
  padding: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  cursor: pointer;
  transition: all 0.3s ease;
  background: white;
  flex-shrink: 0;
}

.type-option:hover {
  border-color: #4a90e2;
  transform: translateY(-2px);
}

.type-option.active {
  border-color: #4a90e2;
  background: rgba(74, 144, 226, 0.05);
}

.option-icon {
  margin-bottom: 12px;
  font-size: 32px;
  line-height: 1;
}

.type-option.active .option-icon {
  color: #4a90e2;
}

.type-option p {
  color: #4a5568;
  font-weight: 500;
  font-size: 16px;
}

.type-option.active p {
  color: #4a90e2;
}

/* 协议声明 */
.agreement {
  margin: 20px 0 30px;
  flex-shrink: 0;
}

.agreement .checkbox-label {
  display: flex;
  align-items: flex-start;
  cursor: pointer;
  font-size: 14px;
  color: #4a5568;
  line-height: 1.5;
}

.agreement .checkbox-custom {
  min-width: 20px;
  height: 20px;
  margin-top: 2px;
}

.agreement .checkbox-text a {
  color: #4a90e2;
  text-decoration: none;
}

.agreement .checkbox-text a:hover {
  text-decoration: underline;
}

.agreement .error-message {
  color: #f56565;
  font-size: 13px;
  margin-top: 5px;
}

/* 注册按钮 */
.register-btn {
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
  flex-shrink: 0;
}

.register-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(74, 144, 226, 0.4);
}

.register-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.register-btn.loading {
  cursor: wait;
}

/* 登录提示 */
.login-prompt {
  text-align: center;
  margin: 25px 0;
  color: #718096;
  font-size: 15px;
  flex-shrink: 0;
}

.login-link {
  color: #4a90e2;
  text-decoration: none;
  font-weight: 600;
  cursor: pointer;
  transition: color 0.3s ease;
}

.login-link:hover {
  color: #2b6cb0;
  text-decoration: underline;
}

/* 底部信息 */
.form-footer {
  text-align: center;
  color: #a0aec0;
  font-size: 13px;
  border-top: 1px solid #e2e8f0;
  padding-top: 20px;
  margin-top: auto; /* 使用auto让底部信息推到最下方 */
  margin-bottom: 0;
  flex-shrink: 0;
  width: 100%;
}

/* 响应式设计 */
@media (max-width: 1100px) {
  .register-card {
    width: 95%;
    height: auto;
    max-height: 90vh;
  }

  .card-form-wrapper {
    height: auto;
    max-height: calc(90vh - 40px);
  }
}

@media (max-width: 900px) {
  .register-card {
    flex-direction: column-reverse; /* 在小屏幕上表单在上，品牌在下 */
    height: auto;
    max-height: 90vh;
  }

  .card-form-wrapper,
  .card-brand {
    width: 100%;
    height: auto;
  }

  .card-form-wrapper {
    max-height: 60vh;
  }

  .card-brand {
    max-height: 30vh;
  }

  .card-form {
    padding: 40px 30px;
  }

  .type-options {
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

  .register-card {
    max-height: 95vh;
  }

  .card-form-wrapper {
    max-height: 65vh;
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
</style>
