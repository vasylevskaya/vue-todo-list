<script>
export default {
  data() {
    return {
      text: '',
    }
  },
  methods: {
    hide() {
      this.text = '';
    },
    show(text) {
      this.text = text;
    }
  }
}
</script>

<template>
  <article
    class="message"
    :class="{ 'message--hidden': !text }"
  >
    <div class="message-header">
      <slot name="header">Error</slot>
      <button class="delete" @click="hide"></button>
    </div>

    <div class="message-body">
      <!-- x is data that can be accessed by parent component -->
      <slot :text="text">Something is wrong</slot>
    </div>
  </article>
</template>
