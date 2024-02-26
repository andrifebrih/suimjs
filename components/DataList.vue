<template> 
  <s-card
    :title="title"
    class="w-full bg-white suim_datalist"
    hide-footer
    :no-gap="noGap"
    :hide-title="hideTitle"
  >
    <template v-if="data.loadingGridCfg || data.loadingSelectData" >
      <slot name="loader">
        <div class="loader"></div>
      </slot> 
    </template>


    <div
      v-if="data.listCfg.setting && gridMode == 'list'"
      v-show="data.controlMode == 'grid'"
    >
      <s-list
        ref="gridCtl"
        class="w-full"
        :read-url="data.gridReadUrl"
        :delete-url="data.gridDeleteUrl"
        :hide-search="gridHideSearch"
        :hide-control="gridHideControl"
        :hide-sort="gridHideSort"
        :hide-delete-button="gridHideDelete"
        :hide-refresh-button="gridHideRefresh"
        :hide-detail="gridHideDetail"
        :hide-new-button="gridHideNew"
        :sort-field="gridSortField"
        :sort-direction="gridSortDirection"
        v-if="data.listCfg.setting"
        :config="data.listCfg"
        @select-data="selectData"
        @new-data="newData"
        @resetCustomFilter="handleGridResetCustomFilter"
      >
        <template v-slot:item="item">
          <slot name="list_item" :item="item" :config="data.listCfg">
            {{ item }}
          </slot>
        </template>
        <template #grid_item_buttons_1="item">
          <slot name="list_item_buttons_1" :item="item"></slot>
        </template>
        <template #grid_item_buttons="item">
          <slot name="list_item_buttons" :item="item"></slot>
        </template>
        <template #grid_item_buttons_2="item">
          <slot name="list_item_buttons_2" :item="item"></slot>
        </template>
        <template #footer_1="prop">
          <slot name="grid_footer_1" :items="prop.items" :recordCount="prop.recordCount" :currentPage="prop.currentPage" :pageCount="prop.pageCount"></slot>
        </template>
        <template #paging="prop">
          <slot name="grid_paging" :items="prop.items" :recordCount="prop.recordCount" :currentPage="prop.currentPage" :pageCount="prop.pageCount"></slot>
        </template>
        <template #footer_2="prop">
          <slot name="grid_footer_2" :items="prop.items" :recordCount="prop.recordCount" :currentPage="prop.currentPage" :pageCount="prop.pageCount"></slot>
        </template>
      </s-list>
    </div>

    <div
      v-if="data.listCfg.setting && gridMode == 'grid'"
      v-show="data.controlMode == 'grid'"
    >
      <s-grid
        ref="gridCtl"
        class="w-full"
        :hide-select="gridHideSelect"
        :editor="gridEditor"
        :read-url="data.gridReadUrl"
        :update-url="data.gridUpdateUrl"
        :delete-url="data.gridDeleteUrl"
        :config="data.listCfg"
        :hide-search="gridHideSearch"
        :hide-control="gridHideControl"
        :hide-detail="gridHideDetail"
        :hide-sort="gridHideSort"
        :hide-edit-button="gridHideEdit"
        :hide-delete-button="gridHideDelete"
        :label-method="gridLabelMethod"
        :hide-refresh-button="gridHideRefresh"
        :hide-new-button="gridHideNew"
        :custom-filter="gridCustomFilter"
        :no-confirm-delete="gridNoConfirmDelete"
        :auto-commit-line="gridAutoCommitLine"
        :total-url="gridTotalUrl"
        :sort-field="gridSortField"
        :sort-direction="gridSortDirection"
        @select-data="selectData"
        @new-data="newData"
        @get-data="getData"
        @delete-data="handleGridRowDelete"
        @row-updated="gridRowUpdated"
        @row-field-changed="handleGridFieldChanged"
        @save-row-data="handleGridRowSave"
        @row-deleted="handleGridRowDeleted"
        @grid-refreshed="handleGridRefreshed"
        @checkUncheckAll="onCheckUncheckAll"
        @checkUncheck="onCheckUncheck"
        @resetCustomFilter="handleGridResetCustomFilter"
      >
        <template #header_search="{ config }">
          <slot name="grid_header_search" :config="config"></slot>
        </template>
        <template #header_buttons_1="{ config }">
          <slot name="grid_header_buttons_1" :config="config"></slot>
        </template>
        <template #header_buttons="{ config }">
          <slot name="grid_header_buttons" :config="config"></slot>
        </template>
        <template #header_buttons_2="{ config }">
          <slot name="grid_header_buttons_2" :config="config"></slot>
        </template>
        <template #item_buttons_1="prop">
          <slot
            name="grid_item_buttons_1"
            :item="prop.item"
            :config="prop.config"
          />
        </template>

        <template #item_buttons="prop">
          <slot
            name="grid_item_buttons"
            :item="prop.item"
            :config="prop.config"
          />
        </template>

        <template #item_buttons_2="prop">
          <slot
            name="grid_item_buttons_2"
            :item="prop.item"
            :config="prop.config"
          />
        </template>

        <template
          v-for="slotname in gridFieldInputSlotNames"
          v-slot:[slotname.itemSlotName]="slotData"
        >
          <slot
            :name="slotname.gridSlotName"
            :item="slotData.item"
            :config="slotData.config"
          ></slot>
        </template>

        
        <template #grid_total="prop">
          <slot name="grid_item_total" :item="prop.item"></slot>
        </template>

        <template #footer_1="prop">
            <slot name="grid_footer_1" :items="prop.items" :recordCount="prop.recordCount" :currentPage="prop.currentPage" :pageCount="prop.pageCount"></slot>
        </template>
        <template #paging="prop">
          <slot name="grid_paging" :items="prop.items" :recordCount="prop.recordCount" :currentPage="prop.currentPage" :pageCount="prop.pageCount"></slot>
        </template>
        <template #footer_2="prop">
          <slot name="grid_footer_2" :items="prop.items" :recordCount="prop.recordCount" :currentPage="prop.currentPage" :pageCount="prop.pageCount"></slot>
        </template>
          
      </s-grid>
    </div>

    <s-form
      v-if="data.controlMode == 'form' && data.formCfg && data.formCfg.setting"
      ref="formCtl"
      v-model="data.record"
      :keep-label="formKeepLabel"
      :config="data.formCfg"
      :mode="data.formMode"
      class="pt-2"
      :auto-focus="formAutoFocus"
      :focus="formFocus"
      @submitForm="save"
      :tabs="formTabTitles"
      :initialTab="formInitialTab"
      @cancelForm="cancelForm"
      :hide-buttons="formHideButtons"
      :hide-submit="formHideSubmit"
      :hide-cancel="formHideCancel"
      @fieldChange="handleFormFieldChange"
      @recordChange="handleFormRecordChange"
    >
      <template
        v-for="name in formFieldInputHeaderSlotNames"
        v-slot:[name]="slotData"
      >
        <slot
          :name="'form_' + name"
          :item="slotData.item"
          :config="slotData.config"
        ></slot>
      </template>

      <template
        v-for="name in formFieldInputSlotNames"
        v-slot:[name]="slotData"
      >
        <slot
          :name="'form_' + name"
          :item="slotData.item"
          :config="slotData.config"
        ></slot>
      </template>

      <template
        v-for="name in formFieldInputFooterSlotNames"
        v-slot:[name]="slotData"
      >
        <slot
          :name="'form_' + name"
          :item="slotData.item"
          :config="slotData.config"
        ></slot>
      </template>

      <template
        v-for="name in formFieldInputOptionSlotNames"
        v-slot:[name]="slotData"
      >
        <slot :name="'form_' + name" :option="slotData.option"></slot>
      </template>

      <template
        v-for="name in formFieldInputSelectedOptionSlotNames"
        v-slot:[name]="slotData"
      >
        <slot :name="'form_' + name" :option="slotData.option"></slot>
      </template>

      <template v-for="tabName in formTabNames" v-slot:[tabName]="{ item }">
        <slot
          :name="'form_' + tabName"
          :item="item"
          :config="{
            formMode: data.formMode,
            appMode: data.controlMode,
            formCfg: data.formCfg,
          }"
        >
          {{ tabName }}
        </slot>
      </template>

      <template #form_header="{ item, config }">
        <slot name="form_header" :item="item" :config="config"></slot>
      </template>

      <template #loader>
        <slot name="form_loader"></slot>
      </template>

      <template #buttons>
        <slot name="form_buttons"></slot>
      </template>

      <template #buttons_1="{ item, config,inSubmission,loading }">
        <slot name="form_buttons_1" :item="item" :config="config" :in-submission="inSubmission" :loading="loading"></slot>
      </template>

      <template #buttons_2="{ item, config }">
        <slot name="form_buttons_2" :item="item" :config="config" :in-submission="inSubmission" :loading="loading"></slot>
      </template>

      <template v-slot:footer_1="{ item, config }">
        <slot name="form_footer_1" :item="item" :config="config" :in-submission="inSubmission" :loading="loading"></slot>
      </template>

      <template v-slot:footer_2="{ item, config }">
        <slot name="form_footer_2" :item="item" :config="config" :in-submission="inSubmission" :loading="loading"></slot>
      </template>
    </s-form>
  </s-card>
</template>

<script setup>
import SCard from "./SCard.vue";
import SGrid from "./SGrid.vue";
import SList from "./SList.vue";
import SForm from "./SForm.vue";
import loadFormConfig from "../scripts/load_form_config.js";
import loadGridConfig from "../scripts/load_grid_config.js";

import {
  reactive,
  inject,
  onMounted,
  ref,
  nextTick,
  computed,
  watch,
} from "vue";
import util from "../scripts/util.js";

const props = defineProps({
  noGap: { type: Boolean },
  title: { type: String, default: "" },
  hideTitle: { type: Boolean },
  gridEditor: { type: Boolean },
  gridEditorNoForm: { type: Boolean },
  gridFields: { type: Array, default: () => [] },
  gridHideControl: { type: Boolean, default: false },
  gridHideSearch: { type: Boolean, default: false },
  gridHideSelect: { type: Boolean, default: false },
  gridHideDetail: { type: Boolean },
  gridHideSort: { type: Boolean },
  gridHideNew: { type: Boolean, default: false },
  gridHideRefresh: { type: Boolean, default: false },
  gridHideEdit: { type: Boolean, default: false },
  gridHideDelete: { type: Boolean, default: false },
  gridSortField: {type: String, default:""},
  gridSortDirection: {type: String, default:""},
  gridCustomFilter: { type: Object, default: () => {} },
  gridNoConfirmDelete: { type: Boolean, default: false },
  formFields: { type: Array, default: () => [] },
  formConfig: { type: [String, Object], default: () => {} },
  formConfigNew: { type: [String, Object], default: () => {} },
  formDefaultMode: { type: String, default: "edit" },
  formRead: { type: String, default: "" },
  formInsert: { type: String, default: "" },
  formUpdate: { type: String, default: "" },
  formHideButtons: { type: Boolean, default: false },
  formHideSubmit: { type: Boolean, default: false },
  formHideCancel: { type: Boolean, default: false },
  formFocus: { type: Boolean, default: false },
  formAutoFocus: { type: Boolean },
  formKeepLabel: { type: Boolean },
  gridMode: { type: String, default: "list" },
  gridLabelMethod: { type: String, default: "labelfield" },
  gridAutoCommitLine: { type: Boolean, default: false },
  gridConfig: { type: [String, Object], default: () => {} },
  gridRead: { type: String, default: "" },
  gridUpdate: { type: String, default: "" },
  gridDelete: { type: String, default: "" },
  gridTotalUrl: { type: String, default: "" },
  stayOnFormAfterSave: { type: Boolean, default: false },
  formTabsNew: { type: Array, default: () => [] },
  formTabsEdit: { type: Array, default: () => [] },
  formTabsView: { type: Array, default: () => [] },
  formInitialTab: { type: Number, default: 0 },
  initAppMode: { type: String, default: "grid" },
  newRecordType: { type: String, default: "form" },
  //initFormMode: { type: String, default: "edit" },
});

const axios = inject("axios");
const emit = defineEmits({
  preSave: null,
  postSave: null,
  formFieldChange: null,
  formRecordChange: null,
  formNewData: null,
  formEditData: null,
  formLoaded: null,
  alterGridConfig: null,
  alterFormConfig: null,
  gridRefreshed: null,
  gridRowAdd: null,
  gridRowUpdated: null,
  gridRowDeleted: null,
  gridRowDelete: null,
  gridRowSave: null,
  gridRowFieldChanged: null,
  controlModeChanged: null,
  formModeChanged: null,
  gridCheckUncheckAll: null,
  gridCheckUncheck: null,
  gridResetCustomFilter:null,
});

const data = reactive({
  controlMode: props.initAppMode,
  formMode: props.formDefaultMode,
  formCfg: {},
  listCfg: {},
  gridReadUrl: props.gridRead,
  gridDeleteUrl: props.gridDelete,
  gridUpdateUrl: props.gridUpdate,
  record: {
    Enable: true,
  },
  loadingGridCfg: false,
  loadingSelectData: false,
});

const gridCtl = ref(null);
const formCtl = ref(null);

watch(
  () => data.controlMode,
  (nv) => {
    emit("controlModeChanged", nv);
  }
);

watch(
  () => data.formMode,
  (nv) => {
    emit("formModeChanged", nv);
  }
);
function handleGridResetCustomFilter(){ 
  emit("gridResetCustomFilter")
}
function handleFormFieldChange(name, v1, v2, old) {
  emit("formFieldChange", name, v1, v2, old, data.record);
}

function handleFormRecordChange(nv) {
  emit("formRecordChange", nv);
}

function handleGridRefreshed() {
  emit("gridRefreshed");
}

function handleGridFieldChanged(name, v1, v2, old, record) {
  emit("gridRowFieldChanged", name, v1, v2, old, record);
}

function handleGridRowDeleted(record) {
  emit("gridRowDeleted", record);
}

function handleGridRowDelete(record, index) {
  emit("gridRowDelete", record, index);
}

function handleGridRowSave(record, index) {
  emit("gridRowSave", record, index);
}

const formTabTitles = computed({
  get() {
    let tabNames =
      data.formMode == "new"
        ? props.formTabsNew
        : data.formMode == "edit"
        ? props.formTabsEdit
        : props.formTabsView;

    return tabNames;
  },
});

const formTabNames = computed({
  get() {
    let tabNames =
      data.formMode == "new"
        ? props.formTabsNew
        : data.formMode == "edit"
        ? props.formTabsEdit
        : props.formTabsView;

    if (tabNames.length <= 1) return [];

    return tabNames.slice(1, tabNames.length).map((el) => {
      return "tab_" + el.replace(" ", "_");
    });
  },
});

const formFieldInputSlotNames = computed({
  get() {
    return props.formFields.map((el) => {
      return "input_" + el;
    });
  },
});

const formFieldInputHeaderSlotNames = computed({
  get() {
    return props.formFields.map((el) => {
      return "input_" + el + "_header";
    });
  },
});

const formFieldInputFooterSlotNames = computed({
  get() {
    return props.formFields.map((el) => {
      return "input_" + el + "_footer";
    });
  },
});

const formFieldInputOptionSlotNames = computed({
  get() {
    return props.formFields.map((el) => {
      return "input_" + el + "_option";
    });
  },
});

const formFieldInputSelectedOptionSlotNames = computed({
  get() {
    return props.formFields.map((el) => {
      return "input_" + el + "_selected-option";
    });
  },
});

const gridFieldInputSlotNames = computed({
  get() {
    return props.gridFields.map((el) => {
      return {
        itemSlotName: "item_" + el,
        gridSlotName: "grid_" + el,
      };
    });
  },
});

function selectData(dt, index) {
  if (props.formRead == "") {
    data.record = dt;
    emit("formEditData", data.record);
    data.controlMode = "form";
    data.formMode = props.formDefaultMode;
    nextTick(() => {
      emit("formLoaded", data.record);
    });
    return;
  }
  
  data.loadingSelectData =true
  axios.post(props.formRead, [dt._id]).then(
    (r) => {
      
     data.loadingSelectData = false
      emit("formEditData", r.data);
      data.controlMode = "form";
      data.formMode = props.formDefaultMode;
      data.record = r.data;
      nextTick(() => {
        emit("formLoaded", data.record);
      });
    },
    (e) => {
      data.loadingSelectData = false
      util.showError(e);
    }
  );
}

function newData(dt) {
  if (dt == undefined) dt = {};

  switch (props.newRecordType) {
    case "form":
      emit("formNewData", dt);
      data.record = dt == undefined ? {} : dt;
      data.controlMode = "form";
      data.formMode = "new";
      nextTick(() => {
        emit("gridRowUpdated", data.record);
        emit("formLoaded", data.record);
      });

    case "grid":
      emit("gridRowAdd", dt);
  }
}

function getData(keyword) {
  emit("gridGetData", keyword);
}

function cancelForm() {
  data.controlMode = "grid";
  nextTick(() => {
    gridCtl.value.refreshData();
  });
}

function save(saveData, cbOK, cbFalse) {
  emit("preSave", saveData);
  const saveEndPoint =
    data.formMode == "new" ? props.formInsert : props.formUpdate;

  if (saveEndPoint == "") {
    emit("postSave", saveData, data.currentIndex);
    if (!props.stayOnFormAfterSave) {
      data.controlMode = "grid";
    } else {
      util.showInfo("data has been saved");
    }
    cbOK();
    return;
  }

  axios.post(saveEndPoint, saveData).then(
    (r) => {
      let record = r.data;
      data.record = record;
      emit("postSave", record);
      emit("gridRowUpdated", record);
      if (!props.stayOnFormAfterSave) {
        data.controlMode = "grid";
        nextTick(() => { 
          gridCtl.value.resetFilter()
          nextTick(() => {
            gridCtl.value.refreshData();
          })
        });
      } else {
        util.showInfo("data has been saved");
        selectData(data.record, "detail");
      }
      cbOK();
    },
    (e) => {
      util.showError(e);
      cbFalse();
    }
  );
}

function gridRowUpdated(dt) {
  emit("gridRowUpdated", dt);
}

function refreshList() {
  if (gridCtl.value) {
    gridCtl.value.refreshData();
    return;
  }
  data.loadingGridCfg = true
  loadGridConfig(axios, props.gridConfig).then(
    (r) => {
      emit("alterGridConfig", r);
      data.listCfg = r;
      data.loadingGridCfg = false
    },
    (e) => data.loadingGridCfg = false
  );
  if (typeof props.gridRead == "string") data.gridReadUrl = props.gridRead;
}

function refreshForm() {
  if (props.formConfig == undefined || props.formConfig == "") return;
  loadFormConfig(axios, props.formConfig).then(
    (r) => {
      emit("alterFormConfig", r);
      data.formCfg = r;
    },
    //(e) => util.showError(e)
  );
}

function getFormSection(name) {
  return formCtl.value.getSection(name);
}

function getFormField(name) {
  return formCtl.value.getField(name);
}

function setFormSectionAttr(name, attr, value) {
  if (formCtl.value == undefined) return;
  formCtl.value.setSectionAttr(name, attr, value);
}

function setFormFieldAttr(name, attr, value) {
  if (formCtl.value == undefined) {
    console.warn("calling setFormFieldAttr when form is not yet initiated");
    return;
  }
  formCtl.value.setFieldAttr(name, attr, value);
}

function removeFormField(name) {
  formCtl.value.removeField(name);
}

function getGridConfig() {
  return data.listCfg;
}

function setGridConfig(cfg) {
  data.listCfg = cfg;
}

function setGridAttr(name, attr, value) {
  const fields = data.listCfg.fields.map((el) => {
    if (el.field != name) return el;

    el[attr] = value;
    return el;
  });
  data.listCfg.fields = fields;
}

function setGridField(name, field) {
  const fields = data.listCfg.fields.map((el) => {
    if (el.field != name) return el;

    return field;
  });
  data.listCfg.fields = fields;
}

function getGridField(name) {
  const fields = data.listCfg.fields.filter((el) => el.field == name);
  if (fields.length == 0) {
    return {};
  }
  return fields[0];
}

function removeGridField(name) {
  const newFields = data.listCfg.fields.filter((el) => {
    return el.field != name;
  });
  data.listCfg.fields = newFields;
}

function getFormRecord() {
  return data.record;
}

function setFormRecord(record) {
  data.record = record;
}

function getGridRecords() {
  return gridCtl.value.getRecords();
}

function getGridRecord(idx) {
  return gridCtl.value.getRecord(idx);
}

function setGridRecord(idx, dt) {
  return gridCtl.value.setRecord(idx, dt);
}

function setGridRecordByID(dt) {
  return gridCtl.value.setRecordByID(dt);
}

function setFormMode(mode) {
  data.formMode = mode;
}

function getFormMode() {
  return data.formMode;
}

function newGridData() {
  if (gridCtl.value) {
    gridCtl.value.newData();
  }
}

function refreshGrid() {
  refreshList();
}

function setControlMode(mode) {
  data.controlMode = mode;
}

function getControlMode() {
  return data.controlMode;
}

function setGridRecords(items) {
  gridCtl.value.setRecords(items);
}

watch(
  () => props.gridRead,
  (nv) => {
    data.gridReadUrl = nv;
    util.nextTickN(1, () => gridCtl.value.refreshData());
  }
);

function getGridCurrentIndex() {
  return gridCtl.value.getCurrentIndex();
}

function getGridSortField() {
  return gridCtl.value.getSortField();
}

function setGridSortField(s) {
  gridCtl.value.setSortField(s);
}

function getGridSortDirection() {
  return gridCtl.value.getSortDirection();
}

function setGridSortDirection(d) {
  gridCtl.value.setSortDirection(d);
}

function getGridSelected() {
  return gridCtl.value.getSelected;
}

function setFormLoading(loading) {
  formCtl.value.setLoading(loading);
}
function getFormLoading() {
  return formCtl.value.getLoading();
}
function formValidate() {
  return formCtl.value.validate();
}

function setFormCurrentTab(formCurrentTab) {
  if (formCtl.value == undefined) {
    return;
  }
  formCtl.value.setCurrentTab(formCurrentTab);
}
function getFormCurrentTab() {
  if (formCtl.value == undefined) {
    return;
  }
  return formCtl.value.getCurrentTab();
}
function getFormAllField() {
  if (formCtl.value == undefined) {
    return [];
  }
  return formCtl.value.getAllField();
}
function gridResetFilter(){
  gridCtl.value.resetFilter();
}
function onCheckUncheckAll(checked) {
  emit("gridCheckUncheckAll", checked);
}

function onCheckUncheck(val) {
  emit("gridCheckUncheck", val);
}

defineExpose({
  getGridRecords,
  getGridRecord,
  refreshGrid,
  setGridRecords,
  getGridSelected,
  setGridRecord,
  setGridRecordByID,
  getGridCurrentIndex,
  getFormRecord,
  setFormRecord,
  getFormField,
  removeFormField,
  setFormFieldAttr,
  getFormSection,
  setFormSectionAttr,
  getGridConfig,
  setGridConfig,
  getGridField,
  removeGridField,
  setGridField,
  setGridAttr,
  refreshList,
  refreshForm,
  setFormMode,
  getFormMode,
  newGridData,
  submitForm: save,
  cancelForm,
  setControlMode,
  getControlMode,
  getGridSortField,
  setGridSortField,
  getGridSortDirection,
  setGridSortDirection,
  setFormLoading,
  getFormLoading,
  formValidate,
  setFormCurrentTab,
  getFormCurrentTab,
  getFormAllField,
  gridResetFilter
});

onMounted(() => {
  refreshList();
  refreshForm();
});
</script>
