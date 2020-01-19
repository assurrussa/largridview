<template>
  <div class="row w-100">
    <div v-if="paginationData.from && paginationData.to" class="col-sm-5">
      <div class="dataTables_info">{{ getTextPagination() }}</div>
    </div>
    <div :class="[{ 'col-sm-7' : (paginationData.from && paginationData.to) }, { 'col-sm-12' : (!paginationData.from || !paginationData.to) }]">
      <ul class="pagination b-pagination float-right pull-right">
        <li class="page-item"
            :class="[{ 'disabled' : (paginationData.currentPage <= 1) }, { 'cursor-pointer' : (paginationData.currentPage > 1) }]"
            :disabled="paginationData.currentPage <= 1"
            @click.prevent="onCLickPrev">
          <a v-if="paginationData.currentPage > 1" href="#" class="page-link">{{ textPrev }}</a>
          <span v-else class="page-link">{{ textPrev }}</span>
        </li>

        <li class="page-item"
            :class="[{ 'disabled' : !paginationData.hasMore }, { 'cursor-pointer' : paginationData.hasMore }]"
            :disabled="!paginationData.hasMore"
            @click.prevent="onCLickNext">
          <a v-if="!paginationData.hasMore" href="#" class="page-link">{{ textNext }}</a>
          <span v-else class="page-link">{{ textNext }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
  export default {
    props: {
      pagination: {
        type: [Array, Object],
        default: () => [],
        required: true,
      },
      id: {
        type: [String],
        default: 'idPagination',
        required: false,
      },
      isBusy: {
        type: [Boolean],
        default: false,
        required: false,
      },
      maxLinks: {
        type: [Number],
        default: 12,
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
    data() {
      return {
        links: [],
        paginationData: {
          currentPage: '',
          total: '',
          path: '',
          hasMore: '',
          firstPageUrl: '',
          nextPageUrl: '',
          prevPageUrl: '',
          from: '',
          to: '',
        },
      };
    },
    mounted() {
      this.fetchPagination(this.pagination);
    },
    watch: {
      pagination(data) {
        this.fetchPagination(data);
      }
    },
    methods: {
      getTextPagination() {
        let string = this.textFromTo
                .replace(':from', this.paginationData.from)
                .replace(':to', this.paginationData.to);

        if(this.paginationData.total) {
          string += ' ' + this.textTotal.replace(':total', this.paginationData.total)
        }

        return string;
      },
      onCLickPrev() {
        if(this.paginationData.currentPage > 1 && !this.isBusy) {
          this.$emit('prev', this.paginationData.prevPageUrl);
        }
      },
      onCLickNext() {
        if(this.paginationData.hasMore && !this.isBusy) {
          this.$emit('next', this.paginationData.nextPageUrl);
        }
      },
      fetchPagination(data) {
        if(data.has_more !== undefined) {
          this.paginationData.currentPage = data.current_page;
          this.paginationData.total = data.total;
          this.paginationData.path = data.path;
          this.paginationData.hasMore = data.has_more;
          this.paginationData.firstPageUrl = data.first_page_url;
          this.paginationData.nextPageUrl = data.next_page_url;
          this.paginationData.prevPageUrl = data.prev_page_url;
          this.paginationData.from = data.from;
          this.paginationData.to = data.to;
        }

        return this.paginationData;
      },
    }
  }
</script>
