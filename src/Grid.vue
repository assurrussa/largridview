<template>
  <div :id="idTable" :class="classBlock">

    <datatable-filter :selectCount="tableData.length"
                      :id="idTable"
                      :selectSearch="tableData.search"
                      :perPage="perPage"
                      :placeholder="placeholderFilterSearch"
                      @select="onFilterCount"
                      @search="onFilterSearch"
                      :isPerPageOn="isPerPageOn"
                      :isSearch="isSearch">
      <slot name="filter"></slot>
      <template v-slot:filter>
        <slot name="filterin"></slot>
      </template>
    </datatable-filter>

    <datatable :classTable="classTable" :fields="columns" :sortKey="tableData.column" :id="idTable" @sort="onSortBy" :isBusy="isBusy || isLoading">
      <template v-slot:filter>
        <slot name="filtertable"></slot>
      </template>
      <slot></slot>
    </datatable>

    <datatable-pagination v-if="isPaginateOn && ! isPaginateSimple"
                          :id="idTable"
                          :pagination="pagination"
                          :isBusy="isBusy || isLoading"
                          :textFromTo="textFromTo + (textTotal.length > 0 ? ' ' + textTotal : '')"
                          :addDeltaLeft="paginateDeltaLeft"
                          :addDeltaRight="paginateDeltaRight"
                          @select="onClickPageSelect"
                          @prev="onClickPagePrev"
                          @next="onClickPageNext">
    </datatable-pagination>

    <datatable-pagination-simple v-if="isPaginateOn && isPaginateSimple"
                                 :id="idTable"
                                 :pagination="pagination"
                                 :isBusy="isBusy || isLoading"
                                 :textFromTo="textFromTo"
                                 :textTotal="textTotal"
                                 :textNext="textNext"
                                 :textPrev="textPrev"
                                 @prev="onClickPagePrev"
                                 @next="onClickPageNext">
    </datatable-pagination-simple>

  </div>
</template>

<script>
  import Datatable from "./Datatable.vue";
  import DatatableFilter from "./DatatableFilter.vue";
  import DatatablePagination from "./DatatablePagination.vue";
  import DatatablePaginationSimple from "./DatatablePaginationSimple.vue";
  import debounce from "lodash/debounce";

  export default {
    name: 'LarGridView',
    props: {
      url: {
        type: [String],
        default: '/',
        required: true,
      },
      fields: {
        type: [Array, Object],
        default: () => [],
        required: true,
      },
      isSend: {
        type: [Boolean],
        default: true,
        required: false,
      },
      isBusy: {
        type: [Boolean],
        default: false,
        required: false,
      },
      isLoadingOn: {
        type: [Boolean],
        default: true,
        required: false,
      },
      isSearch: {
        type: [Boolean],
        default: false,
        required: false,
      },
      isPerPageOn: {
        type: [Boolean],
        default: true,
        required: false,
      },
      isPaginateOn: {
        type: [Boolean],
        default: true,
        required: false,
      },
      isPaginateSimple: {
        type: [Boolean],
        default: false,
        required: false,
      },
      isPaginateAuto: {
        type: [Boolean],
        default: true,
        required: false,
      },
      sortKey: {
        type: [String],
        default: null,
        required: false,
      },
      data: {
        type: [Object],
        default: () => {},
        required: false,
      },
      perPage: {
        type: [Array],
        default: () => [20, 50, 100, 200],
        required: false,
      },
      perPageDefault: {
        type: [Number],
        default: 20,
        required: false,
      },
      idTable: {
        type: [String],
        default: 'tableID',
        required: false,
      },
      classTable: {
        type: [String],
        default: '',
        required: false,
      },
      classBlock: {
        type: [String],
        default: '',
        required: false,
      },
      placeholderFilterSearch: {
        type: [String],
        default: 'Search...',
        required: false,
      },
      dataPath: {
        type: [String],
        default: 'data.list',
        required: false,
      },
      textFromTo: {
        type: [String],
        default: 'Showing :from to :to',
        required: false,
      },
      textTotal: {
        type: [String],
        default: 'of :total',
        required: false,
      },
      textNext: {
        type: [String],
        default: 'Next ›',
        required: false,
      },
      textPrev: {
        type: [String],
        default: '‹ Prev',
        required: false,
      },
      paginateDeltaLeft: {
        type: [Number],
        default: 2,
        required: false,
      },
      paginateDeltaRight: {
        type: [Number],
        default: 2,
        required: false,
      },
    },
    components: {
      Datatable,
      DatatableFilter,
      DatatablePagination,
      DatatablePaginationSimple,
    },
    data() {
      return {
        isLoading: false,
        list: [],
        columns: [],
        tableData: {
          length: 20,
          search: '',
          column: null,
          by: 'desc',
        },
        pagination: {},
      };
    },
    mounted() {
      this.tableData.column = this.sortKey;
      this.setTableLength(this.perPageDefault);
      this.setTableQueryParams();
      this.columns = this.fields;
      this.getListData(this.url);
    },
    watch: {
      sortKey(value) {
        this.tableData.column = value;
      },
    },
    methods: {
      onFilterCount(event) {
        this.tableData.length = parseInt(event);
        this.$emit('filter:select', parseInt(event));
        this.getListData(this.url);
      },
      onFilterSearch(event) {
        this.tableData.search = event;
        this.$emit('filter:search', event);
        this.getListData(this.url);
      },
      onSortBy(event) {
        this.tableData.column = event.column;
        this.tableData.by = event.dir;
        this.$emit('filter:sort', event);
        this.getListData(this.url);
      },
      onClickPageSelect(event) {
        this.$emit('page:select', event);
        this.getListData(event);
      },
      onClickPageNext(event) {
        this.$emit('page:next', event);
        this.getListData(event);
      },
      onClickPagePrev(event) {
        this.$emit('page:prev', event);
        this.getListData(event);
      },
      getListData: debounce(async function(url) {
        if(! this.isSend) {
          return;
        }

        if(url === '' || url === null || url === undefined) {
          return;
        }

        this.setLoadingOn();
        let objectUrlQuery = this.getObjectFromUrl(url);
        if(objectUrlQuery.page && this.tableData.page) {
          delete this.tableData.page;
        }

        let tableDataList = Object.assign(this.tableData, this.data);
        const { data } = await this.$axios.get(url, {params: tableDataList});

        if(this.isPaginateOn) {
          let dataValue = this.getDataValue(data);
          if(dataValue && this.setAutoPaginate(dataValue['has_more'])) {
            this.pagination = dataValue;
          }
        }

        this.$emit('response', data);
        this.setLoadingOff();
      }, 300),
      getIndex(array, key, value) {
        return array.findIndex(i => i[key] === value)
      },
      getDataValue(arrayObject) {
        let dataPath = this.dataPath.split('.'),
                value = null;
        for(let key in dataPath) {
          value = value === null ? arrayObject[dataPath[key]] : value[dataPath[key]];
        }

        return value;
      },
      getObjectFromUrl(url) {
        url = (!url && typeof window !== 'undefined') ? window.location.href : url;
        let question = url.indexOf("?"),
                hash = url.indexOf("#");

        if(hash === -1 && question === -1) {
          return {};
        }

        if(hash === -1) {
          hash = url.length;
        }

        let query = (question === -1 || hash === (question + 1)) ? url.substring(hash) : url.substring(question + 1,hash),
                result = {},
                queryList = query.split("&");

        for(let key in queryList) {
          let itemQuery = queryList[key];
          if(itemQuery) {
            // replace every + with space, regexp-free version
            itemQuery = itemQuery.split("+").join(" ");
            let eq = itemQuery.indexOf("="),
                    key = (eq > -1) ? itemQuery.substr(0,eq) : itemQuery,
                    val = (eq > -1) ? decodeURIComponent(itemQuery.substr(eq + 1)) : "",
                    from = key.indexOf("[");

            if(from === -1) {
              result[decodeURIComponent(key)] = val;
            } else {
              let to = key.indexOf("]",from),
                      index = decodeURIComponent(key.substring(from+1,to));
              key = decodeURIComponent(key.substring(0,from));
              if(!result[key]) {
                result[key] = [];
              }
              if(!index) {
                result[key].push(val);
              } else {
                result[key][index] = val;
              }
            }
          }
        }

        return result;
      },
      setTableLength(value) {
        let minValue = Math.min(...this.perPage),
                maxValue = Math.max(...this.perPage);

        if(value < minValue) {
          value = minValue;
        } else if(value > maxValue) {
          value = maxValue;
        }

        this.tableData.length = value;
      },
      setTableQueryParams() {
        let queryUrl = this.getQueryUrl();
        for(let key in queryUrl) {
          if(this.tableData[key] !== undefined) {
            this.tableData[key] = queryUrl[key];
            if(key === 'length') {
              this.setTableLength(queryUrl[key]);
            } else if(key === 'search' && !this.isSearch) {
              this.tableData[key] = '';
            } else if(key === 'column') {
              this.tableData[key] = queryUrl[key];
              let indexField = this.getIndex(this.fields, 'name', queryUrl[key]);
              if(indexField > -1) {
                this.fields[indexField].sortValue = (queryUrl.by === 'asc') ? 1 : 0;
              }
            }
          }
        }
        if(queryUrl['page'] && parseInt(queryUrl['page']) > 1) {
          this.tableData['page'] = parseInt(queryUrl['page']);
        }
      },
      getQueryUrl() {
        if(this.$route && this.$route.query) {
          return this.$route.query;
        }

        let items = [];
        if(window && window.location) {
          let params = new URLSearchParams(window.location.search);
          params.forEach((value, key) => {
            items[key] = value;
          });
        }

        return items;
      },
      setAutoPaginate(value) {
        if(this.isPaginateOn && this.isPaginateAuto === true) {
          if(value !== undefined && typeof value === 'boolean') {
            this.isPaginateSimple = true;
          } else {
            this.isPaginateSimple = false;
          }
        }

        return true;
      },
      setLoadingOn() {
        if(! this.isLoadingOn) {
          return;
        }
        this.isLoading = true;
        this.$emit('loading', this.isLoading);
      },
      setLoadingOff() {
        if(! this.isLoadingOn) {
          return;
        }
        this.isLoading = false;
        this.$emit('loading', this.isLoading);
      },
    },
  }
</script>
