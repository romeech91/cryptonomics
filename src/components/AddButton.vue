<template>
  <button
    class="header-btn"
    :disabled="disabled"
    :class="{
      'opacity-50': disabled
    }"
  >
    <svg fill="none" height="24" stroke="#000" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
         viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg">
      <circle cx="12" cy="12" r="10"/>
      <line fill="#fff" x1="12" x2="12" y1="8" y2="16"/>
      <line fill="#fff" x1="8" x2="16" y1="12" y2="12"/>
    </svg>
    Добавить
  </button>
</template>

<script>
export default {
  props: {
    disabled: {
      type: Boolean,
      required: false,
      default: false
    }
  },
}
</script>