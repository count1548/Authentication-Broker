<template>
  <div class="table-wrapper">
    <div class="table-toolbar">
      <div class="filter">
        <select v-model="selected">
          <option v-for="(item, idx) in t_category" :key="idx" :value="item.field">{{ item.label }}</option>
        </select>
        <search-bar v-model="search" placeholder="Search" @onsearch="onSearch(search)" :width="200" />
      </div>
      <hr />
    </div>
    <ve-table :columns="columns" :table-data="tableData" class="table-container" />
    <div class="table-pagination">
      <ve-pagination
        :total="searchedData.length"
        :page-index="pageIndex"
        :page-size="pageSize"
        :layout="['prev', 'pager', 'next', 'sizer', 'jumper']"
        :pagingCount="2"
        @on-page-number-change="pageNumberChange"
        @on-page-size-change="pageSizeChange"
      />
    </div>
  </div>
</template>

<style scoped>
.table-wrapper {
  min-width: 1080px;
}
.table-pagination {
  margin-top: 20px;
  text-align: left;
}
</style>

<script>
import SearchBar from './SearchBar.vue';

const filteringData = (object, filter) => {
  for (const value of Object.values(object)) {
    if (typeof filter === typeof value && value.includes(filter)) return true;
  }
  return false;
};

export default {
  components: { SearchBar },
  props: {
    category: Array,
    columns: Array,
    buttons: Array,
    data: {
      type: Array,
      immediate: true,
    },
  },
  data() {
    return {
      search: '',
      searchedData: [],
      selected: '',
      t_category: [],
      pageIndex: 1,
      pageSize: 10, // page index / page size
    };
  },
  computed: {
    // table data
    tableData() {
      const { pageIndex, pageSize } = this;
      return this.searchedData.slice((pageIndex - 1) * pageSize, pageIndex * pageSize);
    },
  },
  created() {
    this.searchedData = this.data;
    this.t_category = [{ label: '전체', field: 'all' }, ...this.category];
    this.selected = 'all';
  },
  methods: {
    pageNumberChange(pageIndex) {
      this.pageIndex = pageIndex;
    },
    pageSizeChange(pageSize) {
      this.pageIndex = 1;
      this.pageSize = pageSize;
    },
    onSearch(search) {
      const { selected, data } = this;
      this.pageIndex = 1;
      if (selected !== 'all') {
        const selectedCategory = this.category.find((item) => item.field === selected);
        if (selectedCategory.filter_func) {
          this.searchedData = data.filter((item) => selectedCategory.filter_func(item, search));
        } else this.searchedData = data.filter((item) => item[selected] && item[selected].includes(search));
      } else this.searchedData = data.filter((item) => filteringData(item, search));
    },
  },
  watch: {
    data(new_val) {
      this.searchedData = new_val;
    },
  },
};
</script>
