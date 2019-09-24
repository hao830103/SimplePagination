<template>
  <div class="row">
    <div class="col-md-12 row">
      <h2>Default</h2>
      <p>With Emit function, we can get the current page number.</p>
      <p>Default example page number: {{ defaultPage }}</p>
      <pagination
        class="col-md-8 col-md-offset-2"
        :MaxPage="12"
        :ShowenPages="3"
        @ChangePage="DefaultPageChange"
      />
    </div>
    <div class="col-md-12 row">
      <h2>With First & Last Button</h2>
      <p>F&L style example page number: {{ secondPage }}</p>
      <pagination
        class="col-md-8 col-md-offset-2"
        :FirsLastButton="true"
        :MaxPage="24"
        :ShowenPages="7"
        @ChangePage="SecondPageChange"
      />
    </div>
  </div>
</template>

<script>
import Pagination from '@/components/Plugin/Pagination'
export default {
  components: {
    Pagination
  },
  data: function () {
    return {
      defaultPage: 0,
      secondPage: 0
    }
  },
  methods: {
    DefaultPageChange: function (page) {
      this.defaultPage = page
    },
    SecondPageChange: function (page) {
      this.secondPage = page
    }
  }
}
</script>
