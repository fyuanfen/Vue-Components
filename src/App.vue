<template>
  <div id="app">
<search ref="search" @search="onSearch" @searchmode="isSearchMode = $event"></search>
  </div>
</template>

<script>
import Search from './search.vue'
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';

export default {
  name: 'app',
  data () {
    return {
       filterReg: null,
       isSearchMode:  false
    }
  },
  components:{
    Search
  },
  methods:{
    onSearch(words) {
    
    if (words instanceof Array) {
      const s = words.reduce((prev, next) => `${prev}(?=.*${next})`, '^');
      this.filterReg = RegExp(`${s}.*`, 'i');
    }
  
    }
  }
}
</script>

