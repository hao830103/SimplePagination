<template>
  <div class="Container">
    <div>
      <h2>Default</h2>
      <pagination :MaxPage="12" :ShowenPages="3" />
    </div>
    <div>
      <h2>With First & Last Button</h2>
      <pagination :FirsLastButton="true" :MaxPage="24" :ShowenPages="7" />
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
.Container div {
  margin: 3%;
  height: 20vh;
}
</style>
