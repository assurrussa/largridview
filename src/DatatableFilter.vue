<template>
  <div class="row w-100">
    <div class="col-sm-12 m-0 p-0">
      <slot></slot>
    </div>
    <div class="col-sm-12 m-0 p-0">
      <div class="pull-left float-left">
        <label v-if="isPerPageOn">
          <select v-model="model" @change="$emit('select', parseInt(model))" class="form-control input-sm">
            <option v-for="(value, index) in perPage" :key="id + index + value" :value="value">{{ value }}</option>
          </select>
        </label>
        <slot name="filter"></slot>
      </div>
      <div v-if="isSearch" class="pull-right float-right">
        <label><input class="form-control input-sm"
                      type="text"
                      v-model="search"
                      :placeholder="placeholder"
                      @input="$emit('search', search)"></label>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    props: {
      id: {
        type: [String],
        default: 'tableID',
        required: false,
      },
      selectCount: {
        type: [String, Number],
        default: 0,
        required: true,
      },
      selectSearch: {
        type: [String],
        default: null,
        required: false,
      },
      isPerPageOn: {
        type: [ Boolean],
        default: true,
        required: false,
      },
      isSearch: {
        type: [ Boolean],
        default: false,
        required: false,
      },
      perPage: {
        type: [Array],
        default: () => [20, 50, 100],
        required: false,
      },
      placeholder: {
        type: [String],
        default: 'Поиск по таблице...',
        required: false,
      },
    },
    data() {
      return {
        model: null,
        search: null,
      };
    },
    mounted() {
      this.fetchSearch(this.selectSearch);
      this.fetchModel(this.selectCount);
    },
    watch: {
      selectCount(data) {
        this.fetchModel(data);
      },
      selectSearch(data) {
        this.fetchSearch(data);
      }
    },
    methods: {
      fetchModel(data) {
        if(parseInt(data) > 0) {
          this.model = parseInt(data);
        }
      },
      fetchSearch(data) {
        if(this.isSearch) {
          this.search = data;
        }
      },
    },
  }
</script>
