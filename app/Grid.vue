<template>
  <div :id="idTable" :class="classBlock">

    <datatable-filter :selectCount="tableData.length"
                      :id="idTable"
                      :selectSearch="tableData.search"
                      :perPage="perPage"
                      :placeholder="placeholderFilterSearch"
                      @select="onFilterCount"
                      @search="onFilterSearch"
                      :isSearch="isSearch">
      <slot name="filter"></slot>
      <template v-slot:filter>
        <slot name="filterin"></slot>
      </template>
    </datatable-filter>

    <datatable :classTable="classTable" :fields="columns" :sortKey="tableData.column" :id="idTable" @sort="onSortBy" :isBusy="isBusy">
      <template v-slot:filter>
        <slot name="filtertable"></slot>
      </template>
      <slot></slot>
    </datatable>

    <datatable-pagination v-if="isPaginateOn && ! isPaginateSimple"
                          :id="idTable"
                          :pagination="pagination"
                          :isBusy="isBusy"
                          :textFromTo="textFromTo + (textTotal.length > 0 ? ' ' + textTotal : '')"
                          @select="getListData($event)"
                          @prev="getListData($event)"
                          @next="getListData($event)">
    </datatable-pagination>

    <datatable-pagination-simple v-if="isPaginateOn && isPaginateSimple"
                                 :id="idTable"
                                 :pagination="pagination"
                                 :isBusy="isBusy"
                                 :textFromTo="textFromTo"
                                 :textTotal="textTotal"
                                 :textNext="textNext"
                                 :textPrev="textPrev"
                                 @select="getListData($event)"
                                 @prev="getListData($event)"
                                 @next="getListData($event)">
    </datatable-pagination-simple>

  </div>
</template>

<script>
  import Datatable from "./Datatable";
  import DatatableFilter from "./DatatableFilter";
  import DatatablePagination from "./DatatablePagination";
  import DatatablePaginationSimple from "./DatatablePaginationSimple";
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
      isBusy: {
        type: [Boolean],
        default: false,
        required: false,
      },
      isSearch: {
        type: [Boolean],
        default: false,
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
        default: 'jambo_table bulk_action table-striped table-bordered',
        required: false,
      },
      classBlock: {
        type: [String],
        default: 'dataTables_wrapper form-inline dt-bootstrap',
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
    },
    components: {
      Datatable,
      DatatableFilter,
      DatatablePagination,
      DatatablePaginationSimple,
    },
    data() {
      return {
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
      }
    },
    methods: {
      onFilterCount(event) {
        this.tableData.length = parseInt(event);
        this.getListData(this.url);
      },
      onFilterSearch(event) {
        this.tableData.search = event;
        this.getListData(this.url);
      },
      onSortBy(event) {
        this.tableData.column = event.column;
        this.tableData.by = event.dir;
        this.getListData(this.url);
      },
      getListData: debounce(async function(url) {
        if(url === '' || url === null || url === undefined) {
          return;
        }
        let objectUrlQuery = this.getObjectFromUrl(url);

        if(objectUrlQuery.page && this.tableData.page) {
          delete this.tableData.page;
        }

        const { data } = await this.$axios.get(url, {params: this.tableData});

        if(this.isPaginateOn) {
          let dataValue = this.getDataValue(data);
          if(dataValue && this.setAutoPaginate(dataValue['has_more'])) {
            this.pagination = dataValue;
          }
        }

        this.$emit('response', data);
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
        for(let key in this.$route.query) {
          if(this.tableData[key] !== undefined) {
            this.tableData[key] = this.$route.query[key];
            if(key === 'length') {
              this.setTableLength(this.$route.query[key]);
            } else if(key === 'search' && !this.isSearch) {
              this.tableData[key] = '';
            } else if(key === 'column') {
              this.tableData[key] = this.$route.query[key];
              let indexField = this.getIndex(this.fields, 'name', this.$route.query[key]);
              if(indexField > -1) {
                this.fields[indexField].sortValue = (this.$route.query.by === 'asc') ? 1 : 0;
              }
            }
          }
        }
        if(this.$route.query['page'] && parseInt(this.$route.query['page']) > 1) {
          this.tableData['page'] = parseInt(this.$route.query['page']);
        }
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
    },
  }
</script>
