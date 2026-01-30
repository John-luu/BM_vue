<template>
  <div class="user-type-selector">
    <p class="user-type-label">{{ label }}</p>
    <div class="type-options">
      <div
        v-for="option in options"
        :key="option.value"
        class="type-option"
        :class="{ active: modelValue === option.value }"
        @click="$emit('update:modelValue', option.value)"
      >
        <div class="option-icon">
          <span class="icon-text">{{ option.icon }}</span>
        </div>
        <p>{{ option.label }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "UserTypeSelector",
  props: {
    modelValue: {
      type: String,
      default: "",
    },
    label: {
      type: String,
      default: "选择您的身份",
    },
    options: {
      type: Array,
      default: () => [
        { label: "学生", value: "学生", icon: "🎓" },
        { label: "老师", value: "老师", icon: "👨‍🏫" },
      ],
    },
  },
  emits: ["update:modelValue"],
};
</script>

<style scoped>
.user-type-selector {
  margin-bottom: 30px;
}

.user-type-label {
  color: #4a5568;
  margin-bottom: 12px;
  font-size: 16px;
}

.type-options {
  display: flex;
  gap: 20px;
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
  color: #a0aec0;
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

@media (max-width: 576px) {
  .type-options {
    flex-direction: column;
  }
}
</style>
