<template>
    <div :class="{ 'input-container': true, error: !props.valid }">
      <label :for="props.id" class="input-label">{{ props.label }}</label>
      <input
        :type="props.type"
        :id="props.id"
        :name="props.id"
        :aria-label="props.id"
        :value="modelValue"
        :placeholder="placeholder"
        :class="{ input: true, error: !props.valid }"
        @input="updateInput"
        @keyup="updateChangeInput"
        :maxlength="props.maxLength"
        ref="inputFieldRef"
      />
      <div v-if="!props.valid" :class="{ 'error-message': true, 'error-message--show': !hideErrorMessage }">
        {{ props.errorMessage }}
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, watch, computed } from "vue";
  
  const emit = defineEmits(["update:modelValue", "validateChange"]);
  
  const props = defineProps({
    id: {
      type: String,
      required: true,
    },
    type: {
      type: String,
      required: true,
    },
    name: {
      type: String,
      required: true,
    },
    label: {
      type: String,
      required: true,
    },
    placeholder: {
      type: String,
      required: true,
    },
    modelValue: {
      type: [String, Number],
      default: "",
    },
    valid: {
      type: Boolean,
      default: false,
    },
    errorMessage: {
      type: String,
      default: "This field is required",
    },
    showError: {
      type: Boolean,
      default: false,
    },
    maxLength: {
      type: Number,
    },
    shouldFocus: {
      type: Boolean,
      default: false,
    },
  });
  
  const inputFieldRef = ref(null);
  
  const updateChangeInput = () => {
    emit("validateChange");
  };
  
  const updateInput = (event) => {
    emit("update:modelValue", event.target.value);
  };
  
  const shouldFocusComputed = computed(() => props.shouldFocus);
  
  watch(shouldFocusComputed, (newVal) => {
    if (newVal) {
      inputFieldRef.value.focus();
    }
  });
  
  const hideErrorMessage = ref(false);
  
  watch(
    () => props.errorMessage,
    (oldVal, newVal) => {
      if (oldVal !== newVal) {
        hideErrorMessage.value = true;
        setTimeout(() => {
          hideErrorMessage.value = false;
        }, 100);
      }
    }
  );
  </script>
  
  <style scoped>
  /* Add your custom styles here */
  .input-container {
    display: flex;
    flex-direction: column;
    margin-bottom: 1rem; /* Adjust spacing as needed */
  }
  
  .input-label {
    margin-bottom: 0.5rem; /* Adjust label spacing as needed */
  }
  
  .input,
  .error-message {
    padding: 0.5rem 1rem; /* Adjust padding as needed */
    border: 1px solid #ddd;
    border-radius: 4px;
  }
  
  .input {
    background-color: #fff;
  }
  
  .error {
    border-color: #ff0000;
  }
  
  .error-message {
    color: #ff0000;
    font-size: 0.8rem; /* Adjust error message font size as needed */
    display: none;
  }
  
  .error-message--show {
    display: block;
  }
  </style>
  