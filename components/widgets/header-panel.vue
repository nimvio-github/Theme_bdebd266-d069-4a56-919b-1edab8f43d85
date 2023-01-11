<template>
  <section class="h-8 bg-light-gray">
    <div class="container mx-auto h-full flex items-center">
      <nuxt-img
        :src="props.logo"
        :alt="props.logoAlt || 'panel-logo'"
        width="16"
        height="16"
      />
      <div class="pl-2 text-dark-gray" v-html="props.description"></div>
    </div>
  </section>
</template>

<script setup>
const props = defineProps({
  logo: {
    type: String,
    required: true,
  },
  logoAlt: {
    type: String,
    default: "",
  },
  description: {
    type: String,
    default: "",
  },
});
</script>
