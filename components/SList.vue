<template>
  <div class="flex flex-col gap-4 suim_list">
    <s-modal :display="false" ref="deleteModal" @submit="confirmDelete">
      You will delete data ! Are you sure ?<br/>
      Please be noted, this can not be undone !
    </s-modal>
    
    <div class="flex gap-2 justify-center header" v-if="!hideControl">
      <input
        type="text"
        class="grow bg-transparent outline-none border-b border-slate-300 text-slate-500 text-sm py-2 search_input"
        placeholder="enter search keyword"
        v-model="data.keyword"
        @keyup.enter="refreshData"
        v-if="!hideSearch"
      />
      <div class="grow" v-else>&nbsp;</div>
      <button @click="changeSortDirection" v-if="!hideSort"  class="sort_btn">
        <mdicon :name="sortIcon" size="18" />
      </button>
      <select
        v-model="data.sortField"
        class="bg-transparent border-b border-slate-300 text-[0.8em] pt-1 pb-1 sort_select "
        @change="refreshData"
        v-if="config.setting && !hideSort"
      >
        <option value="" class="text-black">No Sort</option>
        <option
          v-for="f in config.setting.sortable"
          :value="f"
          class="text-black"
        >
          {{ f }}
        </option>
      </select>
      <div class="flex gap-1 header_button">
        <slot name="header_buttons_1" :config="config"></slot>
        <slot name="header_buttons" :config="config">
          <s-button
            icon="refresh"
            class="bg-primary text-white refresh_btn"
            @click="refreshData"
            v-if="!hideRefreshButton"
          />
          <s-button
            icon="plus"
            class="bg-primary text-white new_btn"
            @click="newData"
            v-if="!hideNewButton"
          />
        </slot>
        <slot name="header_buttons_2" :config="config"></slot>
      </div>
    </div>

    <div v-if="!data.loading">
      <div
        v-if="data.items && data.items.length != 0"
        class="flex flex-col gap-2"
      >
        <ul class="grid grid-cols-2 gap-2" :class="[clsGridCol, clsGap]">
          <li v-if="showHeader">
            <slot name="header" v-bind:items="data.items">
              <div class="border-b border-slate-600 py-2 font-bold">
                Showing {{ items.length }} data
              </div>
            </slot>
          </li>
          <li
            v-if="data.items.length > 0"
            v-for="(record, idx) in data.items"
            class="gap-2 items-start cursor-pointers last:border-none bg-white"
            :class="{
              'border-b first:pt-0': col == 1 && !hideLineSeparator,
              'rounded-sm shadow-md': col > 1,
            }"
          >
            <slot
              name="box_item"
              :item="record"
              :index="idx"
              :delete-data="deleteData"
              :select-data="selectData"
            >
              <div class="p-2 flex">
                <div
                  class="grow cursor-pointer hover:text-primary"
                  :class="{ 'hover:none pointer-events-none ':hideDetail}"
                  @click="selectData(record, 'detail')"
                >
                  <slot name="item" :item="record">
                    {{ record }}
                  </slot>
                </div>
                <div class="flex gap-1 action">
                  <slot name="action_1" :item="record" :index="idx"></slot>
                  <slot name="actions" :item="record" :index="idx">
                    <s-button
                      class="delete_action"
                      type="icon"
                      icon="delete"
                      v-if="!hideDeleteButton"
                      @click="deleteData(record, idx)"
                    />
                  </slot>
                  <slot name="action_2" :item="record" :index="idx"></slot>
                </div>
              </div>
            </slot>
          </li>
        </ul>
        <div v-if="!props.hideFooter">
          <slot
            name="footer_1"
            v-bind="{
              items: data.items,
              recordCount: data.recordCount,
              currentPage: data.currentPage,
              pageCount: pageCount,
            }"
          >
          </slot>
          <slot
            name="paging"
            v-bind="{
              items: data.items,
              recordCount: data.recordCount,
              currentPage: data.currentPage,
              pageCount: pageCount,
            }"
          >
            <div v-if="pageCount > 1" class="flex gap-2 justify-center pagination">
              <mdicon
                name="arrow-left"
                class="cursor-pointer"
                :class="{
                  'opacity-25': data.currentPage == 1,
                }"
                @click="changePage(data.currentPage - 1)"
              />
              <div class="pagination_info">Page {{ data.currentPage }} of {{ pageCount }}</div>
              <mdicon
                name="arrow-right"
                class="cursor-pointer"
                :class="{ 'opacity-25': data.currentPage == pageCount }"
                @click="changePage(data.currentPage + 1)"
              />
            </div>
          </slot>
          <slot
            name="footer_2"
            v-bind="{
              items: data.items,
              recordCount: data.recordCount,
              currentPage: data.currentPage,
              pageCount: pageCount,
            }"
          >
          </slot>
        </div>
      </div>

      <div v-else>
        <slot name="nodata"> No data </slot>
      </div>
    </div>
    <div v-else>
      <slot name="loading"> loading data from server .... </slot>
    </div>
  </div>
</template>

<script setup>
import { computed, inject, onMounted, reactive, ref } from "vue";
import SButton from "./SButton.vue";
import SModal from './SModal.vue';
import util from "../scripts/util";

const props = defineProps({
  col: { type: Number, default: 1 },
  gap: { type: Number, default: 0 },
  modelValue: { type: Array, default: () => [] },
  customFilter: { type: Object, default: () => {} },
  config: { type: Object, default: () => {} },
  readUrl: { type: String, default: "" },
  deleteUrl: { type: String, default: "" },
  pageSize: { type: Number, default: 10 },
  sortField:{ type: String, default:""},
  sortDirection: { type: String, default:""},
  showHeader: { type: Boolean },
  hideControl: { type: Boolean },
  hideSearch: { type: Boolean },
  hideSort: { type: Boolean },
  hideButtons: { type: Boolean, default: false },
  hideRefreshButton: { type: Boolean, default: false },
  hideDetail: { type: Boolean, default: false },
  hideNewButton: { type: Boolean, default: false },
  hideDeleteButton: { type: Boolean, default: false },
  hideFooter: { type: Boolean, default: false },
  hideLineSeparator: { type: Boolean, default: false },
});

const axios = inject("axios");
const deleteModal = ref(null);

const emit = defineEmits({
  newData: null,
  selectData: null,
  deleteData: null,
  "modelValue:update": null,
  resetCustomFilter:null,
});

const data = reactive({
  keyword: "",
  items: props.modelValue == undefined ? [] : props.modelValue,
  recordCount: props.modelValue == undefined ? 0 : props.modelValue.length,
  currentPage: 1,
  sortField: props.sortField == undefined || props.sortField == '' ? "_id" : props.sortField,
  sortDirection: ["asc","desc"].includes(props.sortField) ?  props.sortDirection : "desc",
  pageSize: props.pageSize,
  deleteFn: undefined,
  loading: false,
});

const clsGridCol = computed({
  get() {
    return "lg:grid-cols-" + String(props.col);
  },
});

const clsGap = computed({
  get() {
    return `gap-x-${String(props.gap)}`;
  },
});

const sortIcon = computed({
  get() {
    switch (data.sortDirection) {
      case "asc":
        return "arrow-up";

      case "desc":
        return "arrow-down";

      default:
        return "radiobox-blank";
    }
  },
});

const pageCount = computed({
  get() {
    return Math.ceil(data.recordCount / data.pageSize);
  },
});

const items = computed({
  get() {
    if (data.items == undefined) data.items = [];
    return data.items;
  },

  set(v) {
    data.items = v;
    emit("modelValue:update", v);
  },
});

function changeSortDirection() {
  const sorts = ["asc", "desc"];
  const sortCount = sorts.length;

  let found = false;
  for (let i = 0; i < sortCount; i++) {
    if (found) {
      data.sortDirection = sorts[i];
      if (data.sortDirection != "") refreshData();
      return;
    }
    if (sorts[i] == data.sortDirection) found = true;
  }

  data.sortDirection = "asc";
  if (data.sortDirection != "") refreshData();
}

function setLoading(loading) {
  data.loading = loading;
}

function queryParam() {
  const keywordFields = props.config.setting
    ? props.config.setting.keywordFields
    : [];
  const param = {
    Skip: (data.currentPage - 1) * data.pageSize,
    Take: data.pageSize,
  };

  const filters = [];
  if (keywordFields.length > 0 && data.keyword && data.keyword != "") {
    filters.push({
      Op: "$or",
      Items: keywordFields.map((k) => {
        return {
          Field: k,
          Op: "$contains",
          Value: [data.keyword],
        };
      }),
    });
  }

  if (props.customFilter && props.customFilter.Op != "")
    filters.push(props.customFilter);
  if (filters.length > 0)
    param.Where =
      filters.length === 1 ? filters[0] : { Op: "$and", Items: filters };


  if (data.sortField != "") {
    if (data.sortDirection == "asc") {
      param.Sort = [data.sortField];
    } else {
      param.Sort = ["-" + data.sortField];
    }
  }
  return param;
}

function refreshData() {
  if (props.readUrl == "") {
    emit("getData", data.keyword);
    return;
  }

  setLoading(true);
  axios.post(props.readUrl, queryParam()).then(
    (r) => {
      data.items = r.data.data;
      data.recordCount = r.data.count;
      setLoading(false);
    },
    (e) => {
      util.showError(e);
      setLoading(false);
    }
  );
}

function newData() {
  emit("newData", null);
}

function addData(dt) {
  data.items.push(dt);
  emit("modelValue:update", data.items);
}

function deleteData(record, dataIndex) {
  deleteModal.value.show();
  data.deleteFn = () => {
    if (props.deleteUrl == "") {
      emit("deleteData", data);
      refreshData();
      return;
    } else {
      axios.post(props.deleteUrl, record).then(
        (r) => {
          refreshData();
        },
        (e) => {
          util.showError(e);
        }
      );
    }
  };
}

function confirmDelete() {
  deleteModal.value.hide();
  data.deleteFn();
}

function selectData(data, op) {
  if(props.hideDetail) return
  emit("selectData", data, op);
}

function changePage(page) {
  data.currentPage = page;
  refreshData();
}

defineExpose({
  refreshData,
  addData,
});

onMounted(() => {
  refreshData();
});
</script>
