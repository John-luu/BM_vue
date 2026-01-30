<template>
  <button
    :type="type"
    :class="[
      'auth-button',
      variant,
      size,
      { 'full-width': fullWidth, loading: loading },
    ]"
    :disabled="disabled || loading"
    @click="$emit('click', $event)"
  >
    <span v-if="loading" class="loading-spinner"></span>
    <slot v-else name="prefix-icon"></slot>
    <span class="button-text">
      <slot>{{ text }}</slot>
    </span>
    <slot v-if="!loading" name="suffix-icon"></slot>
  </button>
</template>

<script>
export default {
  name: "AuthButton",
  props: {
    type: {
      type: String,
      default: "button",
    },
    variant: {
      type: String,
      default: "primary", // primary, secondary, outline, text
      validator: (value) =>
        ["primary", "secondary", "outline", "text"].includes(value),
    },
    size: {
      type: String,
      default: "medium", // small, medium, large
      validator: (value) => ["small", "medium", "large"].includes(value),
    },
    text: {
      type: String,
      default: "",
    },
    fullWidth: {
      type: Boolean,
      default: true,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    loading: {
      type: Boolean,
      default: false,
    },
  },
  emits: ["click"],
};
</script>

<style scoped>
.auth-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  border-radius: 12px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  font-family: inherit;
  border: none;
  position: relative;
}

.auth-button.full-width {
  width: 100%;
}

/* 尺寸 */
.auth-button.small {
  padding: 10px 20px;
  font-size: 14px;
}

.auth-button.medium {
  padding: 16px 24px;
  font-size: 16px;
}

.auth-button.large {
  padding: 20px 32px;
  font-size: 18px;
}

/* 变体 */
.auth-button.primary {
  background: linear-gradient(135deg, #4a90e2 0%, #2b6cb0 100%);
  color: white;
  box-shadow: 0 4px 15px rgba(74, 144, 226, 0.3);
}

.auth-button.primary:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(74, 144, 226, 0.4);
}

.auth-button.secondary {
  background: white;
  color: #4a90e2;
  border: 2px solid #e2e8f0;
}

.auth-button.secondary:hover:not(:disabled) {
  border-color: #4a90e2;
  transform: translateY(-2px);
}

.auth-button.outline {
  background: transparent;
  color: #4a90e2;
  border: 2px solid #4a90e2;
}

.auth-button.outline:hover:not(:disabled) {
  background: rgba(74, 144, 226, 0.05);
  transform: translateY(-2px);
}

.auth-button.text {
  background: transparent;
  color: #4a90e2;
  padding: 8px 16px;
}

.auth-button.text:hover:not(:disabled) {
  color: #2b6cb0;
  text-decoration: underline;
}

/* 禁用状态 */
.auth-button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none !important;
  box-shadow: none !important;
}

/* 加载状态 */
.auth-button.loading {
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

.auth-button.secondary .loading-spinner,
.auth-button.outline .loading-spinner,
.auth-button.text .loading-spinner {
  border: 3px solid rgba(74, 144, 226, 0.3);
  border-top-color: #4a90e2;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
