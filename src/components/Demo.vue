<template>
  <div>
    <div class="DemoBlock">
      <h2>Default</h2>
      <pagination :MaxPage="12" />
    </div>
    <div class="DemoBlock">
      <h2>With First & Last Button</h2>
      <pagination :FirsLastButton="true" :MaxPage="12" />
    </div>
  </div>
</template>

<script>
import Pagination from '@/components/Plugin/Pagination'
export default {
  components: {
    Pagination
  }
}
</script>

<style scoped>
.DemoBlock {
  margin: 3%;
}
</style>
