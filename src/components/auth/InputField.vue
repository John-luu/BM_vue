<template>
  <div class="form-group">
    <label v-if="label" class="input-label">{{ label }}</label>
    <div class="input-with-icon" :class="{ 'has-error': error }">
      <div v-if="icon" class="input-icon">
        <span class="icon-text">{{ iconText }}</span>
      </div>
      <input
        :type="inputType"
        :value="modelValue"
        @input="$emit('update:modelValue', $event.target.value)"
        :placeholder="placeholder"
        :required="required"
        :disabled="disabled"
        class="form-input"
        :class="{ error: error }"
      />
      <div class="input-suffix">
        <slot name="suffix">
          <button
            v-if="showPasswordToggle"
            type="button"
            class="password-toggle"
            @click="$emit('toggle-password')"
          >
            <span class="toggle-icon">{{ passwordToggleIcon }}</span>
          </button>
        </slot>
      </div>
    </div>
    <div v-if="error" class="error-message">{{ error }}</div>
    <div v-if="hint" class="hint-text">{{ hint }}</div>
  </div>
</template>

<script>
export default {
  name: "InputField",
  props: {
    modelValue: {
      type: [String, Number],
      default: "",
    },
    label: {
      type: String,
      default: "",
    },
    type: {
      type: String,
      default: "text",
    },
    placeholder: {
      type: String,
      default: "",
    },
    required: {
      type: Boolean,
      default: false,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    error: {
      type: String,
      default: "",
    },
    hint: {
      type: String,
      default: "",
    },
    icon: {
      type: String,
      default: "",
    },
    showPassword: {
      type: Boolean,
      default: false,
    },
  },
  emits: ["update:modelValue", "toggle-password"],
  computed: {
    inputType() {
      if (this.type === "password" && this.showPassword) {
        return "text";
      }
      return this.type;
    },
    iconText() {
      const icons = {
        user: "👤",
        password: "🔒",
        id: "🆔",
        email: "📧",
        phone: "📱",
      };
      return icons[this.icon] || "📝";
    },
    showPasswordToggle() {
      return this.type === "password";
    },
    passwordToggleIcon() {
      return this.showPassword ? "👁️‍🗨️" : "👁️";
    },
  },
};
</script>

<style scoped>
.form-group {
  margin-bottom: 25px;
}

.input-label {
  display: block;
  color: #4a5568;
  margin-bottom: 8px;
  font-size: 15px;
  font-weight: 500;
}

.input-with-icon {
  position: relative;
  display: flex;
  align-items: center;
}

.input-icon {
  position: absolute;
  left: 16px;
  z-index: 1;
  font-size: 18px;
}

.icon-text {
  display: block;
  line-height: 1;
}

.form-input {
  width: 100%;
  padding: 16px 20px 16px 50px;
  font-size: 16px;
  border: 2px solid #e2e8f0;
  border-radius: 12px;
  background: white;
  transition: all 0.3s ease;
  color: #2d3748;
  font-family: inherit;
}

.form-input:focus {
  outline: none;
  border-color: #4a90e2;
  box-shadow: 0 0 0 3px rgba(74, 144, 226, 0.1);
}

.form-input.error {
  border-color: #f56565;
}

.form-input.error:focus {
  box-shadow: 0 0 0 3px rgba(245, 101, 101, 0.1);
}

.form-input:disabled {
  background-color: #f7fafc;
  cursor: not-allowed;
}

.input-suffix {
  position: absolute;
  right: 16px;
  top: 50%;
  transform: translateY(-50%);
}

.password-toggle {
  background: none;
  border: none;
  cursor: pointer;
  color: #a0aec0;
  padding: 5px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: color 0.3s ease;
  font-size: 18px;
  line-height: 1;
}

.password-toggle:hover {
  color: #4a90e2;
}

.toggle-icon {
  display: block;
  line-height: 1;
}

.error-message {
  color: #f56565;
  font-size: 13px;
  margin-top: 5px;
}

.hint-text {
  color: #718096;
  font-size: 12px;
  margin-top: 5px;
}
</style>
