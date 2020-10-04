<script>
import Vue from 'vue';
import VuetifyXlsxCsvParser from '@/vuetify-xlsx-csv-parser.vue';

export default Vue.extend({
  name: 'ServeDev',
  components: {
    VuetifyXlsxCsvParser
  }
});
</script>

<template>
  <div id="app">
    <vuetify-xlsx-csv-parser />
  </div>
</template>
