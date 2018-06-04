
<template>
  <div @click="focusNewTag()" :class="{'read-only': readOnly}" class="input-text">
    <slot class="icon" name="left-icon"></slot>
    <div v-for="(tag, index) in innerTags" :key="index" class="input-tag-wrapper">
      <div class="tag-item">
        <div class="tag-label">{{ tag }}</div>
        <div v-if="!readOnly" @click.prevent.stop="remove(index)" class="tag-icon"></div>
      </div>
    </div>
    <div class="input-tag-wrapper new-tag-wrapper">
      <input v-if="!readOnly && !isLimit" ref="inputtag" :placeholder="placeholder" type="text" v-model="newTag" v-on:keydown.delete.stop="removeLastTag" v-on:keydown="addNew" v-on:blur="addNew" class="new-tag" />
    </div>
    <slot class="icon" name="right-icon"></slot>
  </div>
</template>

<style lang="scss">
.input-text {
  position: relative;
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  padding: 0rem 1rem;
  border: none;
  box-shadow: inset 0 0 0 0.0625rem #e0e0e0;
  border-radius: 0.25rem;
  font-size: 0.875rem;
  color: #616161;
  outline: none;
  line-height: 1.25rem;
  cursor: text;
  background-color: #fff;
  transition: box-shadow 0.1s linear;

  .input-tag-wrapper {
    display: inline-block;
    padding: 5px;
    max-width: 100%;
    .tag-item {
      max-width: 100%;
      background-color: #e0e0e0;
      display: inline-block;
      border-radius: 0.25rem;
      display: -ms-inline-flexbox;
      display: inline-flex;
      -ms-flex-align: center;
      align-items: center;
      cursor: pointer;
      color: #616161;
      transition: all 0.1s;

      .tag-label {
        font-size: 0.875rem;
        height: 1.5rem;
        padding: 0 0.625rem;
        line-height: 1.5rem;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
      }
      .tag-icon {
        margin-left: -0.625rem;
        cursor: pointer;
        min-width: 1.5rem;
        width: 1.5rem;
        height: 1.5rem;
        line-height: 1.5rem;
        text-align: center;
        font-size: 0.75rem;
        transform: scale(0.91667);
        transform-origin: 0.75rem 0.75rem;
        display: inline-block;
        &::before {
          content: 'x';
        }
      }
    }
    &.new-tag-wrapper {
      flex: 1;
      min-width: 20px;
      // in chrome, when click the input,then hide the placeholder
      input:focus::-webkit-input-placeholder {
        text-indent: -999em;
        z-index: -20;
      }
      .new-tag {
        border: none;
        background: transparent;
        outline: none;
        color: #616161;
        line-height: 1.5rem;
        min-width: 1.25rem;
        max-width: 100%;
      }
    }
  }
  .read-only {
    cursor: default;
  }
}
</style>
<script>
const validators = {
  email: new RegExp(
    /^(([^<>()[\]\\.,;:\s@\"]+(\.[^<>()[\]\\.,;:\s@\"]+)*)|(\".+\"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
  ),
  url: new RegExp(
    /^(https?|ftp|rmtp|mms):\/\/(([A-Z0-9][A-Z0-9_-]*)(\.[A-Z0-9][A-Z0-9_-]*)+)(:(\d+))?\/?/i
  ),
  text: new RegExp(/^[a-zA-Z]+$/),
  digits: new RegExp(/^[\d() \.\:\-\+#]+$/),
  isodate: new RegExp(/^\d{4}[\/\-](0?[1-9]|1[012])[\/\-](0?[1-9]|[12][0-9]|3[01])$/)
};

export default {
  name: 'InputTag',
  props: {
    tags: {
      type: Array,
      default: () => []
    },
    placeholder: {
      type: String,
      default: ''
    },
    readOnly: {
      type: Boolean,
      default: false
    },
    validate: {
      type: String | Object,
      default: ''
    },
    addTagOnKeys: {
      type: Array,
      default: function() {
        return [
          13, // Return
          32 // Space
          // 188, // Comma ','
          // 9, // Tab
        ];
      }
    },
    addTagOnBlur: {
      type: Boolean,
      default: false
    },
    limit: {
      default: -1
    }
  },
  data() {
    return {
      newTag: '',
      innerTags: [...this.tags]
    };
  },
  watch: {
    tags() {
      this.innerTags = [...this.tags];
    }
  },
  computed: {
    isLimit: function() {
      return this.limit > 0 && Number(this.limit) === this.innerTags.length;
    }
  },
  methods: {
    focusNewTag() {
      if (this.readOnly || !this.$el.querySelector('.new-tag')) {
        return;
      }
      this.$el.querySelector('.new-tag').focus();
    },
    addNew(e) {
      // Do nothing if the current key code is
      // not within those defined within the addTagOnKeys prop array.
      if (
        (e &&
          this.addTagOnKeys.indexOf(e.keyCode) === -1 &&
          (e.type !== 'blur' || !this.addTagOnBlur)) ||
        this.isLimit
      ) {
        return;
      }
      if (e) {
        e.stopPropagation();
        e.preventDefault();
      }
      this.generateTag();
    },
    generateTag() {
      if (
        this.newTag &&
        this.innerTags.indexOf(this.newTag) === -1 &&
        this.validateIfNeeded(this.newTag)
      ) {
        this.innerTags.push(this.newTag);
        this.newTag = '';
        this.tagChange();
      }
    },
    validateIfNeeded(tagValue) {
      if (this.validate === '' || this.validate === undefined) {
        return true;
      } else if (
        typeof this.validate === 'string' &&
        Object.keys(validators).indexOf(this.validate) > -1
      ) {
        return validators[this.validate].test(tagValue);
      } else if (typeof this.validate === 'object' && this.validate.test !== undefined) {
        return this.validate.test(tagValue);
      }
      return true;
    },
    remove(index) {
      this.innerTags.splice(index, 1);
      this.tagChange();
    },
    removeLastTag() {
      if (this.newTag) {
        return;
      }
      this.innerTags.pop();
      this.tagChange();
    },
    tagChange() {
      this.$emit('update:tags', this.innerTags);
    }
  }
};
</script>
