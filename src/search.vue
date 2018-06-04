<template>
  <div class="search-content">
    <input-tag ref="inputTag" :tags.sync="tagsArray" :placeholder="'输入关键词过滤显示内容'" :limit="3">
      <i class="el-icon-search icon search-icon" slot="left-icon" @click="onSearch"></i>
      <i class="el-icon-close icon close-icon" slot="right-icon" @click="onClear"></i>
    </input-tag>
  </div>
</template>
<style lang="scss">
.search-content {
  .icon {
    cursor: pointer;
  }
  .search-icon {
    position: absolute;
    top: 10px;
    left: 5px;
  }
}
</style>
<script>
import inputTag from './input-tag.vue';

export default {
  data(){
    return {
    tagsArray : [],
   value :''
    }
  },
    components: {
    inputTag
  },
  watch:{
    tagsArray(v,ov) {
        if (v.length) {
          this.$emit('searchmode', true);
        } else {
          this.$emit('searchmode', false);
        }
        if (!v || !Array.isArray(v)) {
          return;
        }
        this.onSearch();
      
    }
  },
  methods: {
    onSearch() {
      this.$refs.inputTag.generateTag();
      this.$emit('search', this.tagsArray);
    },
    onClear() {
      this.tagsArray = [];
      this.onSearch();
    }
  }

}
</script>
