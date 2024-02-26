<template>
  <div style="w-full" class="flex flex-col gap-1 suim_grid">
    <s-modal :display="false" ref="deleteModal" @submit="confirmDelete">
      You will delete data ! Are you sure ?<br />
      Please be noted, this can not be undone !
    </s-modal>

    <div
      class="flex gap-2 justify-center items-center header"
      v-if="!hideControl"
    >
      <slot name="header_search" :config="config">
        <input
          type="text"
          class="grow input_field border_b_[1px] search_input"
          placeholder="enter search keyword"
          v-model="data.keyword"
          @keyup.enter="refreshData"
          v-if="!hideSearch"
        />
        <div v-else class="grow">&nbsp;</div>
      </slot>
      <button @click="changeSortDirection" v-if="!hideSort" class="sort_btn">
        <mdicon :name="sortIcon" size="18" />
      </button>
      <select
        v-model="data.sortField"
        class="sort_select border-b"
        @change="refreshData"
        v-if="config.setting && !hideSort"
      >
        <option value="">No Sort</option>
        <option v-for="f in config.setting.sortable" :value="f">{{ f }}</option>
      </select>
      <div class="flex gap-[1px] header_button">
        <slot name="header_buttons_1" :config="config"></slot>
        <slot name="header_buttons" :config="config">
          <s-button
            icon="refresh"
            class="btn_primary refresh_btn"
            tooltip="refresh"
            @click="refreshData"
            v-if="!hideRefreshButton"
          />
          <s-button
            icon="plus"
            class="btn_primary new_btn"
            tooltip="add new"
            @click="newData"
            v-if="!hideNewButton"
            :disabled="data.recordChanged && !hideSaveButton"
          />
        </slot>
        <slot name="header_buttons_2" :config="config"></slot>
      </div>
    </div>

    <div v-if="!data.loading">
      <div v-if="data.items.length > 0">
        <table class="w-full table-auto suim_table">
          <!-- header -->
          <thead name="grid_header">
            <tr class="border-b-[1px] border-slate-500">
              <th class="row_select" v-if="!hideSelect">
                <input type="checkbox" @change="checkUncheckAll" />
              </th>
              <th
                v-for="(hdr, hdrIndex) in config.fields.filter(
                  (el) => el.readType == 'show'
                )"
                :key="'grid_col_' + hdrIndex"
                class=""
                :class="{
                  'text-right': hdr.align == 'right' || hdr.kind == 'number',
                  'pr-4': hdr.align == 'right' || hdr.kind == 'number',
                  'text-left': !(hdr.align == 'right' || hdr.kind == 'number'),
                }"
                :style="hdr.width != '' ? `width:${hdr.width}` : ''"
              >
                {{ hdr.label }}
              </th>
              <th
                class="w-[50px] max-w-[180px] ml-[10px] border-l-[1px] border-white text-left text-sm font-semibold"
                v-if="!hideAction"
              >
                Action
              </th>
            </tr>
          </thead>

          <!-- records -->
          <tbody name="grid_body" :class="{ 'text-[0.9em] editor': editor }">
            <tr
              v-for="(r, rIdx) in data.items"
              :key="'grid_item_' + rIdx"
              class="cursor-pointer border-b-[1px] border-slate-200 last:border-non hover:bg-slate-200"
              :class="{ 'even:bg-slate-100': !editor, 'hover:none pointer-events-none ':hideDetail}"
              @dblclick="selectData(r, 'detail', true)"
            >
              <td class="w-[30px] text-center" v-if="!hideSelect">
                <!-- <input type="checkbox" v-model="r.isSelected" /> -->
                <slot name="checkbox" :item="r"
                  ><input
                    type="checkbox"
                    v-model="r.isSelected"
                    @change="checkUncheck(r)"
                /></slot>
              </td>

              <td
                class=""
                v-for="(hdr, hdrIndex) in config.fields.filter(
                  (el) => el.readType == 'show'
                )"
                :key="'grid_col_' + hdrIndex"
              >
                <slot :name="'item_' + hdr.field" :item="r" :header="hdr">
                  <div v-if="editor && !hdr.input.readOnly">
                    <s-input
                      hide-label
                      :ctl-ref="{ rowIndex: rIdx }"
                      :field="hdr.input.field"
                      :kind="hdr.input.kind"
                      :label="
                        hdr.input.kind == 'checkbox' || hdr.input.kind == 'bool'
                          ? ''
                          : hdr.input.label
                      "
                      :disabled="hdr.input.readOnly"
                      :caption="hdr.input.caption"
                      :hint="hdr.input.hint"
                      :multi-row="hdr.input.multiRow"
                      :use-list="hdr.input.useList"
                      :items="hdr.input.items"
                      :rules="hdr.input.rules"
                      :required="hdr.input.required"
                      :read-only="hdr.input.readOnly"
                      :lookup-url="hdr.input.lookupUrl"
                      :lookup-key="hdr.input.lookupKey"
                      :allow-add="hdr.input.allowAdd"
                      :lookup-format1="hdr.input.lookupFormat1"
                      :lookup-format2="hdr.input.lookupFormat2"
                      :decimal="hdr.input.decimal"
                      :date-format="hdr.input.dateFormat"
                      :multiple="hdr.input.multiple"
                      :lookup-labels="hdr.input.lookupLabels"
                      :lookup-searchs="
                        hdr.input.lookupSearchs &&
                        hdr.input.lookupSearchs.length == 0
                          ? hdr.input.lookupLabels
                          : hdr.input.lookupSearchs
                      "
                      @focus="rowFieldFocus"
                      @change="rowFieldChanged"
                      v-model="r[hdr.input.field]"
                      ref="inputs"
                    />
                  </div>
                  <div v-else>
                    <s-grid-column
                      :record="r"
                      :column-config="hdr"
                      :label-method="labelMethod"
                    />
                  </div>
                </slot>
              </td>

              <td class="row_action" v-if="!hideAction">
                <slot name="item_buttons_1" :item="r" :config="config"></slot>
                <slot name="item_buttons" :item="r" :config="config">
                  <a
                    href="#"
                    v-if="
                      editor &&
                      r.suimRecordChange === true &&
                      !hideSaveButton &&
                      !autoCommitLine
                    "
                    @click="saveRowData(r, rIdx)"
                    class="save_action"
                  >
                    <mdicon
                      name="content-save"
                      width="16"
                      alt="edit"
                      class="cursor-pointer hover:text-primary"
                    />
                  </a>
                  <a
                    href="#"
                    v-if="!hideDetail && !hideEditButton"
                    @click="selectData(r, rIdx)"
                    class="edit_action"
                  >
                    <mdicon
                      name="pencil"
                      width="16"
                      alt="edit"
                      class="cursor-pointer hover:text-primary"
                    />
                  </a>
                  <a
                    href="#"
                    v-if="!hideDeleteButton"
                    @click="deleteData(r, rIdx)"
                    class="delete_action"
                  >
                    <mdicon
                      name="delete"
                      width="16"
                      alt="delete"
                      class="cursor-pointer hover:text-primary"
                    />
                  </a>
                </slot>
                <slot name="item_buttons_2" :item="r" :config="config"></slot>
              </td>
            </tr>
          </tbody>

          <!-- total -->
          <tbody
            name="grid_body_total"
            :class="{ 'text-[0.9em] editor': editor }"
            v-if="totalUrl !== ''"
          >
            <slot name="grid_total" :item="data.total"></slot>
          </tbody>
        </table>

        <div v-if="!props.hideFooter" class="mt-5 text-sm">
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
            <div
              v-if="pageCount > 1"
              class="flex gap-2 justify-center pagination"
            >
              <mdicon
                name="arrow-left"
                class="cursor-pointer"
                :class="{
                  'opacity-25': data.currentPage == 1,
                }"
                @click="changePage(data.currentPage - 1)"
              />
              <div class="pagination_info">
                Page {{ data.currentPage }} of {{ pageCount }}
              </div>
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
        <slot name="nodata">
          <div class="nodata">No data</div>
        </slot>
      </div>
    </div>

    <div v-else>
      <slot name="loading">
        <div class="loading">loading data from server ...</div>
      </slot>
    </div>
  </div>
</template>

<script setup>
import SButton from "./SButton.vue";
import SInput from "./SInput.vue";
import SGridColumn from "./SGridColumn.vue";
import SModal from "./SModal.vue";
import { computed, inject, onMounted, reactive, ref, watch } from "vue";
import util from "../scripts/util";
import { mdiEmoticon, mdiWindowShutter } from "@mdi/js";
import { useRoute } from "vue-router";

const route = useRoute();

const props = defineProps({
  modelValue: { type: Array, default: () => [] },
  config: { type: Object, default: () => {} },
  customFilter: { type: Object, default: () => {} },
  readUrl: { type: String, default: "" },
  updateUrl: { type: String, default: "" },
  deleteUrl: { type: String, default: "" },
  pageSize: { type: Number, default: 25 },
  hideHeader: { type: Boolean, default: false },
  editor: { type: Boolean, default: false },
  sortField: { type: String, default: "" },
  sortDirection: { type: String, default: "" },
  autoCommitLine: { type: Boolean, default: false },
  labelMethod: { type: String, default: "database" },
  noConfirmDelete: { type: Boolean },
  hideNew: { type: Boolean },
  hideControl: { type: Boolean },
  hideSearch: { type: Boolean },
  hideSort: { type: Boolean },
  hideButtons: { type: Boolean, default: false },
  hideRefreshButton: { type: Boolean, default: false },
  hideNewButton: { type: Boolean, default: false },
  hideSaveButton: { type: Boolean },
  hideDeleteButton: { type: Boolean, default: false },
  hideEditButton: { type: Boolean, default: false },
  hideFooter: { type: Boolean, default: false },
  hideSelect: { type: Boolean, default: false },
  hideDetail: { type: Boolean, default: false },
  hideAction: { type: Boolean, default: false },
  totalUrl: { type: String, default: "" },
});

const axios = inject("axios");

const emit = defineEmits({
  newData: null,
  selectData: null,
  deleteData: null,
  rowUpdated: null,
  rowDeleted: null,
  rowFieldChanged: null,
  gridRefreshed: null,
  "update:modelValue": null,
  refreshTotal: null,
  checkUncheckAll: null,
  checkUncheck: null,
  resetCustomFilter:null,
});

const data = reactive({
  keyword: "",
  items: props.modelValue == undefined ? [] : props.modelValue,
  recordCount: props.modelValue == undefined ? 0 : props.modelValue.length,
  currentPage: 1,
  sortField:
    props.sortField == undefined || props.sortField == ""
      ? "_id"
      : props.sortField,
  sortDirection: ["asc", "desc"].includes(props.sortField)
    ? props.sortDirection
    : "desc",
  pageSize: props.pageSize,
  deleteFn: undefined,
  loading: false,
  currentIndex: -1,
  recordChanged: false,
  showDeleteModal: false,
  total: [],
});

const deleteModal = ref(null);

function resetCustomFilter(){
  emit("resetCustomFilter")
}

function rowFieldFocus(name, v1, v2, ctlRef) {
  const prevIndex = data.currentIndex;
  const currentRowIndex = ctlRef.rowIndex;

  if (prevIndex == currentRowIndex) {
    return;
  }

  data.currentIndex = currentRowIndex;
  data.recordChanged = false;
}

function saveRowData(r, rowIndex) {
  if (
    props.updateUrl == undefined ||
    props.updateUrl == "" ||
    props.updateUrl == null
  ) {
    emit("saveRowData", r, rowIndex);
    return;
  }

  axios.post(props.updateUrl, r).then(
    (r) => {
      r = r.data;
      r.suimRecordChange = false;
      data.items[rowIndex] = r;
      emit("rowUpdated", r);
      updateRecordChanged();
    },
    (e) => util.showError(e)
  );
}

function updateRecordChanged() {
  for (const itIndex in data.items) {
    const it = data.items[itIndex];
    if (it.suimRecordChange === true) {
      data.recordChanged = true;
      return;
    }
  }

  data.recordChanged = false;
}

function rowFieldChanged(name, v1, v2) {
  const currentIndex = data.currentIndex;
  const current = data.items[currentIndex];
  if (props.autoCommitLine) {
    emit("rowFieldChanged", name, v1, v2, current, current);
    emit("update:modelValue", data.items);
  } else {
    current.suimRecordChange = true;
    data.recordChanged = true;
    emit("rowFieldChanged", name, v1, v2, current, current);
  }
}

const selecteds = computed(() => {
  return data.items.filter((el) => el.isSelected === true);
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

function setLoading(loading) {
  data.loading = loading;
}

function checkUncheckAll(ev) {
  const checked = ev.target.checked;
  data.items.forEach((i) => (i.isSelected = checked));
  emit("checkUncheckAll", checked);
}

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
function resetFilter(){
  data.keyword ="";
  resetCustomFilter()

}
function refreshData(callBackFn) {
  if (props.readUrl == undefined || props.readUrl == "") {
    emit("getData", data.keyword);
    if (callBackFn && typeof callBackFn == "function") callBackFn();
    emit("gridRefreshed");
    return;
  }
  setLoading(true);
  axios.post(props.readUrl, queryParam()).then(
    (r) => {
      data.items = r.data.data.map((d) => {
        d.isSelected = false;
        return d;
      });
      data.recordCount = r.data.count;
      setLoading(false);
      emit("gridRefreshed", data.items);
      if (props.totalUrl != "") {
        setTotal();
      }
      if (callBackFn && typeof callBackFn == "function") callBackFn();
    },
    (e) => {
      //util.showError(e);
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
  emit("rowUpdated", dt);
  updateRecordChanged();
}

function deleteData(record, dataIndex) {
  data.deleteFn = () => {
    if (props.deleteUrl == "") {
      emit("deleteData", data, dataIndex);
      refreshData(() => {
        emit("rowDeleted", record);
        updateRecordChanged();
      });
      return;
    } else {
      axios.post(props.deleteUrl, record).then(
        (r) => {
          refreshData(() => {
            emit("rowDeleted", record);
            updateRecordChanged();
          });
        },
        (e) => {
          util.showError(e);
        }
      );
    }
  };

  if (props.noConfirmDelete) {
    data.deleteFn();
    return;
  }

  deleteModal.value.show();
}

function confirmDelete() {
  deleteModal.value.hide();
  data.deleteFn();
}

function selectData(data, index, dblclick) {
  if(props.hideDetail) return
  if (dblclick && props.editor) return;
  data.currentIndex = index;
  emit("selectData", data, index);
}

function changePage(page) {
  data.currentPage = page;
  refreshData();
}

function getRecords() {
  return data.items;
}

function getRecord(recordIndex) {
  return data.items[recordIndex];
}

function setRecord(recordIndex, record) {
  data.items[recordIndex] = record;
}

function setRecordByID(record) {
  data.items.forEach((dt, index) => {
    if (dt._id == record._id) {
      data.items[index] = record;
    }
  });
}

function setTotal() {
  const url = props.totalUrl;
  const params = data.items;
  axios.post(url, params).then(
    (r) => {
      data.total = r.data;
      emit("RefreshTotal", r.data);
    },
    (e) => {
      util.showError(e);
    }
  );
}

function getActiveIndex() {
  return data.currentIndex;
}

function setRecords(newDataSet) {
  data.items = newDataSet;
}

function getCurrentIndex() {
  return data.currentIndex;
}

function getSortField() {
  return data.sortField;
}

function setSortField(s) {
  data.sortField = s;
}

function getSortDirection() {
  return data.sortDirection;
}

function setSortDirection(d) {
  data.sortDirection = d;
}

function getSelected() {
  return selecteds;
}

function checkUncheck(item) {
  emit("checkUncheck", item);
}
defineExpose({
  getCurrentIndex,
  getRecords,
  getRecord,
  getActiveIndex,
  setRecord,
  setRecordByID,
  getSelected,
  refreshData,
  addData,
  newData,
  getSortField,
  setSortField,
  getSortDirection,
  setSortDirection,
  setLoading,
  setRecords,
  setTotal,
  resetFilter,
});

onMounted(() => {
  refreshData();
});

watch(
  () => props.modelValue,
  (nv) => {
    if (nv == undefined) data.items = [];
    data.items = nv;
  }
);

watch(
  () => route.query,
  () => {
    data.keyword = "";
  }
);
</script>
