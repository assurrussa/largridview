<template>
  <table class="table dataTable" :class="classTable">
    <thead :style="blockGetStyle()">
    <tr class="headings">
      <th class="column-title"
          v-for="(field, index) in sortOrdersFields"
          @click.prevent="sortBy(field, index)"
          :class="thGetClass(field, index)"
          :style="thGetStyle(field, index)"
          :key="'thtable' + id + field.name + index"
      >
        {{ field.label }}
      </th>
    </tr>
    </thead>
    <tbody>
    <slot name="filter" :style="blockGetStyle()"></slot>
    <tr v-if="isBusy">
      <td :colspan="fields.length" class="loader-block">
        <div class="loader-ellipsis"><div></div><div></div><div></div><div></div></div>
      </td>
    </tr>
    <slot v-if="! isBusy"></slot>
    </tbody>
  </table>
</template>

<script>
  export default {
    props: {
      id: {
        type: [String],
        default: 'tableID',
        required: false,
      },
      sortKey: {
        type: [String],
        default: null,
        required: false,
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
      classTable: {
        type: [String],
        default: 'jambo_table bulk_action table-striped table-bordered',
        required: false,
      },
    },
    data() {
      return {
        sortOrdersFields: [],
      };
    },
    mounted() {
      this.fetchSortOrders(this.fields);
    },
    watch: {
      fields(data) {
        this.fetchSortOrders(data);
      }
    },
    methods: {
      blockGetStyle() {
        let string = '';

        if(this.isBusy) {
          string += 'opacity: 0.5;';
        }

        return string;
      },
      fetchSortOrders(data) {
        data = JSON.parse(JSON.stringify(data));

        data.forEach((column) => {
          column.sortValue = column.sortValue !== undefined ? column.sortValue : -1;
          this.sortOrdersFields.push(column);
        });
      },
      thGetStyle(item, index) {
        let string = 'cursor: pointer;';

        if(item.width) {
          string += 'width: ' + item.width + ';';
        }

        return string;
      },
      thGetClass(item, index) {
        if(item.isAction === true) {
          return '';
        }

        if(item.isSort === false) {
          return '';
        }

        let string = 'sorting';

        if(this.sortKey === item.name) {
          string = (item.sortValue > 0) ? 'sorting_asc' : 'sorting_desc'
        }

        return string;
      },
      sortBy(item, index) {
        let data = {};

        if(item.isAction === true) {
          return;
        }
        if(item.isSort === false) {
          return;
        }

        this.sortKey = item.name;
        item.sortValue = item.sortValue * -1;

        data.index = index;
        data.column = this.sortKey;
        data.dir = item.sortValue === 1 ? 'asc' : 'desc';
        this.$emit('sort', data);
      },
    },
  }
</script>

<style scoped>
  .loader-block {
    position: relative;
    width: 100%;
    height: 200px;
    text-align: center;
  }
  .loader-ellipsis {
    display: inline-block;
    position: absolute;
    width: 80px;
    height: 80px;
    top: 50%;
    left: 50%;
    -webkit-transform: translate(-50%, -50%);
    transform: translate(-50%, -50%);
  }
  .loader-ellipsis div {
    position: absolute;
    top: 33px;
    width: 13px;
    height: 13px;
    border-radius: 50%;
    background: #2a3f54;
    animation-timing-function: cubic-bezier(0, 1, 1, 0);
  }
  .loader-ellipsis div:nth-child(1) {
    left: 8px;
    animation: loader-ellipsis1 0.6s infinite;
  }
  .loader-ellipsis div:nth-child(2) {
    left: 8px;
    animation: loader-ellipsis2 0.6s infinite;
  }
  .loader-ellipsis div:nth-child(3) {
    left: 32px;
    animation: loader-ellipsis2 0.6s infinite;
  }
  .loader-ellipsis div:nth-child(4) {
    left: 56px;
    animation: loader-ellipsis3 0.6s infinite;
  }
  @keyframes loader-ellipsis1 {
    0% {
      transform: scale(0);
    }
    100% {
      transform: scale(1);
    }
  }
  @keyframes loader-ellipsis3 {
    0% {
      transform: scale(1);
    }
    100% {
      transform: scale(0);
    }
  }
  @keyframes loader-ellipsis2 {
    0% {
      transform: translate(0, 0);
    }
    100% {
      transform: translate(24px, 0);
    }
  }
</style>
