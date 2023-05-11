<template>
  <div>Page {{curr_page}} out of {{max_page}}</div>
</template>

<script>
export default {
  /* eslint-disable */
  props: ['max_page','curr_page']
}
</script>
<style scoped>
div {
    font-size: 20px
}
</style>