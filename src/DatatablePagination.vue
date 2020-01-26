<template>
  <div class="row w-100">
    <div class="col-sm-5">
      <div class="dataTables_info">{{ getTextPagination() }}</div>
    </div>
    <div class="col-sm-7">
      <ul class="pagination b-pagination float-right pull-right">
        <li class="page-item"
            :class="{ 'disabled' : (paginationData.currentPage <= 1) }"
            :disabled="paginationData.currentPage <= 1"
            @click.prevent="onCLickPrev">
          <a v-if="paginationData.currentPage > 1" href="#" class="page-link">‹</a>
          <span v-else class="page-link">‹</span>
        </li>

        <li v-for="(item, index) in links" :key="'pagination' + id + item.label + index"
            class="page-item"
            :class="{'active' : item.isActive }"
            @click.prevent="onCLickSelect(item)">
          <a v-if="item.link" href="#" class="page-link">{{ item.label }}</a>
          <span v-else class="page-link">{{ item.label }}</span>
        </li>

        <li class="page-item"
            :class="{ 'disabled' : (paginationData.currentPage === paginationData.lastPage) }"
            :disabled="paginationData.currentPage === paginationData.lastPage"
            @click.prevent="onCLickNext">
          <a v-if="paginationData.currentPage < paginationData.lastPage" href="#" class="page-link">›</a>
          <span v-else class="page-link">›</span>
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
        default: 'Showing :from to :to of :total',
        required: false,
      },
      addDeltaLeft: {
        type: [Number],
        default: 2,
        required: false,
      },
      addDeltaRight: {
        type: [Number],
        default: 2,
        required: false,
      },
    },
    data() {
      return {
        links: [],
        paginationData: {
          lastPage: '',
          currentPage: '',
          total: '',
          path: '',
          lastPageUrl: '',
          nextPageUrl: '',
          prevPageUrl: '',
          from: '',
          to: ''
        },
      };
    },
    mounted() {
      this.fetchLinks(this.pagination);
    },
    watch: {
      pagination(data) {
        this.fetchLinks(data);
      }
    },
    methods: {
      getTextPagination() {
        return this.textFromTo
                .replace(':from', this.paginationData.from)
                .replace(':to', this.paginationData.to)
                .replace(':total', this.paginationData.total);
      },
      onCLickPrev() {
        if(this.paginationData.currentPage > 1 && !this.isBusy) {
          this.$emit('prev', this.paginationData.prevPageUrl);
        }
      },
      onCLickSelect(item) {
        if(! item.isActive && !this.isBusy) {
          this.$emit('select', item.link);
        }
      },
      onCLickNext() {
        if((this.paginationData.currentPage < this.paginationData.lastPage) && !this.isBusy) {
          this.$emit('next', this.paginationData.nextPageUrl);
        }
      },
      fetchListLinksPaginate(currentPage, lastPage) {
        let left = currentPage - (this.addDeltaLeft < 1 ? 1 : this.addDeltaLeft),
            right = currentPage + (this.addDeltaRight < 1 ? 1 : this.addDeltaRight),
            range = [],
            rangeWithDots = [],
            l;

        for (let i = 1; i <= lastPage; i++) {
          if ((i === 1) || (i === lastPage) || (i >= left && i <= right)) {
            range.push(i);
          }
        }

        for (let i of range) {
          if (l) {
            if (i - l === 2) {
              rangeWithDots.push(l + 1);
            } else if (i - l !== 1) {
              rangeWithDots.push('...');
            }
          }
          rangeWithDots.push(i);
          l = i;
        }

        return rangeWithDots;
      },
      fetchLinks(data) {
        data = this.fetchPagination(data);

        let list = [],
                firstPage = 1,
                currentPage = data.currentPage,
                lastPage = data.lastPage,
                basePath = data.path;

        if(lastPage > this.maxLinks) {
          let listDataLinks = this.fetchListLinksPaginate((currentPage < 3 ? currentPage : currentPage), lastPage);

          for(let key in listDataLinks) {
            let pageNumber = listDataLinks[key];
            if(typeof pageNumber === 'number') {
              list.push({
                link: basePath + ((pageNumber === firstPage) ? '' : '?page=' + pageNumber),
                isActive: (pageNumber === currentPage),
                label: pageNumber,
              });
            } else if(typeof pageNumber === 'string') {
              list.push({link: null, isActive: false, label: '...'});
            }
          }
        } else {
          for(let iPage = firstPage; iPage <= lastPage; iPage++) {
            list.push({
              link: basePath + ((iPage === firstPage) ? '' : '?page=' + iPage),
              isActive: (iPage === currentPage),
              label: iPage,
            });
          }
        }

        this.links = list;
      },
      fetchPagination(data) {
        if(data.last_page !== undefined) {
          this.paginationData.lastPage = data.last_page;
          this.paginationData.currentPage = data.current_page;
          this.paginationData.total = data.total;
          this.paginationData.path = data.path;
          this.paginationData.lastPageUrl = data.last_page_url;
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
