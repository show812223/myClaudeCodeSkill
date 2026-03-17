# 頁面設計模板

開發新頁面時，根據頁面類型選用對應模板。所有頁面都遵循統一的設計慣例。

---

## 1. DataTable 頁面 {#datatable}

最常用的頁面類型。以 `UIDataTable` 為根元件，支援搜尋、篩選、排序、分頁。

### 結構概覽

```
<UIDataTable>
  ├── #title              → 頁面標題 (Icon + 文字)
  ├── #appendTitleSpacer  → 標題列左側附加操作（排序切換、導航連結等）
  ├── #appendTitle        → 標題列右側主要操作（新增按鈕）
  ├── #item.{fieldName}   → 自訂欄位渲染
  └── #item.actions       → 行操作按鈕
<UIDialog type="Input">   → 新增/編輯彈窗
<UIDialog type="Delete">  → 刪除確認彈窗
```

### 完整範例

```vue
<i18n lang="json">
{
  "zh-TW": {
    "pageTitle": "區域設定",
    "add": "新增",
    "edit": "編輯",
    "name": "名稱",
    "enabled": "啟用",
    "description": "描述",
    "fieldName": "欄位名稱"
  },
  "en": {
    "pageTitle": "Zone Settings",
    "add": "Add",
    "edit": "Edit",
    "name": "Name",
    "enabled": "Enabled",
    "description": "Description",
    "fieldName": "Field Name"
  }
}
</i18n>

<template>
  <UIDataTable
    class="table"
    :paging-tool="service"
    :headers="headers"
    :items="service.items"
    :loading="isLoading"
    :server-items-length="service.totalItemNum"
    :filterable="false"
    :searchable="true"
    @update:options="loadData"
  >
    <!-- 標題：Icon + 文字 -->
    <template #title>
      <v-icon start>mdi-alpha-z-box-outline</v-icon>
      {{ t("pageTitle") }}
    </template>

    <!-- 次要操作（可選） -->
    <template #appendTitleSpacer>
      <v-tooltip :text="t('sort')" location="bottom">
        <template #activator="{ props }">
          <v-btn
            icon="mdi-sort"
            variant="text"
            density="comfortable"
            v-bind="props"
            class="mr-2"
          />
        </template>
      </v-tooltip>
    </template>

    <!-- 主要操作 -->
    <template #appendTitle>
      <v-btn
        variant="elevated"
        color="primary"
        prepend-icon="mdi-plus"
        @click="addDialog.isOpen = true"
      >
        {{ t("add") }}
      </v-btn>
    </template>

    <!-- 自訂欄位渲染：布林值 -->
    <template #item.isEnable="{ item }">
      <v-checkbox
        v-model="item.isEnable"
        disabled
        density="compact"
        hide-details
      />
    </template>

    <!-- 自訂欄位渲染：帶 Chip 標籤 -->
    <template #item.name="{ item }">
      {{ item.name }}
      <v-chip
        v-if="item.isMain"
        size="small"
        variant="tonal"
        class="ml-1"
      >
        {{ t("main") }}
      </v-chip>
    </template>

    <!-- 操作欄：圖示按鈕模式（hover 變色） -->
    <template #item.actions="{ item }">
      <div class="d-flex">
        <v-hover>
          <template v-slot:default="{ isHovering, props }">
            <v-btn
              v-bind="props"
              :color="isHovering ? 'primary' : 'secondary'"
              icon="mdi-pencil"
              size="36"
              variant="plain"
              @click="openEditDialog(item)"
            />
          </template>
        </v-hover>
        <v-hover>
          <template v-slot:default="{ isHovering, props }">
            <v-btn
              v-bind="props"
              :color="isHovering ? 'error' : 'secondary'"
              icon="mdi-delete"
              size="36"
              variant="plain"
              @click="openDeleteDialog(item)"
            />
          </template>
        </v-hover>
      </div>
    </template>
  </UIDataTable>

  <!-- 新增 Dialog -->
  <UIDialog
    v-model="addDialog.isOpen"
    :type="DialogType.Input"
    :is-loading="addDialog.isLoading"
    :title-text="t('add')"
    persistent
    @click:cancel="addDialog.isOpen = false"
    @confirm="addItem()"
  >
    <template #content>
      <v-text-field
        v-model="addDialog.name"
        :label="t('fieldName')"
        variant="outlined"
        density="compact"
        hide-details
        class="ma-2"
      />
    </template>
  </UIDialog>

  <!-- 編輯 Dialog -->
  <UIDialog
    v-model="editDialog.isOpen"
    :type="DialogType.Input"
    :is-loading="editDialog.isLoading"
    :title-text="t('edit')"
    persistent
    @click:cancel="editDialog.isOpen = false"
    @confirm="editItem()"
  >
    <template #content>
      <v-text-field
        v-model="editDialog.item.name"
        :label="t('name')"
        variant="outlined"
        density="compact"
        hide-details
        class="ma-2"
      />
    </template>
  </UIDialog>

  <!-- 刪除 Dialog -->
  <UIDialog
    v-model="deleteDialog.isOpen"
    :type="DialogType.Delete"
    :is-loading="deleteDialog.isLoading"
    :deleteContent="deleteDialog.item?.name"
    @click:cancel="deleteDialog.isOpen = false"
    @confirm="deleteItem()"
  />
</template>

<script setup lang="ts">
import { definePageMeta } from "#imports";
import { DialogType } from "~ui/domain-classes/universal/dialog";
import {
  type SBDataTableHeader,
  SBTableHeaderType,
} from "~ui/utils/dataTableHeader";
import { XxxService } from "~/services/XxxService";

definePageMeta({
  layout: "default-menu-item-permission-check",
});

// --- 工具初始化 ---
const { t } = useI18n();
const service = reactive(new XxxService());
const $toast = useNuxtApp().$toast;
const route = useRoute();
const isLoading = ref(false);

// --- Dialog 狀態 ---
const addDialog = reactive({
  isOpen: false,
  isLoading: false,
  name: "",
});

const editDialog = reactive({
  isOpen: false,
  isLoading: false,
  item: { id: "", name: "" },
});

const deleteDialog = reactive({
  isOpen: false,
  isLoading: false,
  item: null as { id: string; name: string } | null,
});

// --- 表頭定義 ---
const headers = computed<SBDataTableHeader[]>(() => [
  {
    title: t("name"),
    key: "name",
    type: SBTableHeaderType.text,
    searchable: true,
    filterable: true,
    sortable: false,
  },
  {
    title: t("enabled"),
    key: "isEnable",
    type: SBTableHeaderType.boolean,
    searchable: false,
    filterable: true,
    sortable: false,
  },
  {
    title: t("description"),
    key: "description",
    type: SBTableHeaderType.text,
    searchable: true,
    filterable: true,
    sortable: false,
  },
  {
    title: "",
    key: "actions",
    width: "100",
    searchable: false,
    sortable: false,
    filterable: false,
  },
]);

// --- Lifecycle ---
onMounted(() => {
  loadData();
});

// --- Methods ---
async function loadData() {
  try {
    isLoading.value = true;
    await service.getItems();
  } catch (error: unknown) {
    console.error(error);
  } finally {
    isLoading.value = false;
  }
}

function openEditDialog(item: any) {
  editDialog.isOpen = true;
  editDialog.item = JSON.parse(JSON.stringify(item));
}

function openDeleteDialog(item: any) {
  deleteDialog.isOpen = true;
  deleteDialog.item = item;
}

async function addItem() {
  try {
    addDialog.isLoading = true;
    await service.addItem(addDialog.name);
    $toast.success(t("dialog.add_successfully"));
  } catch (error: unknown) {
    $toast.error(t("dialog.add_failed"));
  } finally {
    addDialog.isLoading = false;
    addDialog.isOpen = false;
    addDialog.name = "";
  }
}

async function editItem() {
  try {
    editDialog.isLoading = true;
    await service.updateItem(editDialog.item);
    $toast.success(t("dialog.update_successfully"));
  } catch (error: unknown) {
    $toast.error(t("dialog.update_failed"));
  } finally {
    editDialog.isLoading = false;
    editDialog.isOpen = false;
  }
}

async function deleteItem() {
  try {
    deleteDialog.isLoading = true;
    await service.deleteItem(deleteDialog.item?.id ?? "");
    $toast.success(t("dialog.delete_successfully"));
  } catch (error: unknown) {
    $toast.error(t("dialog.delete_failed"));
  } finally {
    deleteDialog.isLoading = false;
    deleteDialog.isOpen = false;
  }
}
</script>
```

### 標題變體

#### 基本模式

```vue
<template #title>
  <v-icon start>mdi-icon-name</v-icon>
  {{ t("pageTitle") }}
</template>
```

#### 返回按鈕模式

```vue
<template #title>
  <v-btn icon :to="backRoute" size="small">
    <v-icon size="24">mdi-arrow-left</v-icon>
  </v-btn>
  <v-icon size="24">mdi-icon-name</v-icon>
  {{ t("pageTitle") }}
</template>
```

#### 動態標題模式

```vue
<template #title>
  <v-icon size="22" start>mdi-account-supervisor-outline</v-icon>
  <p>{{ service.currentItem?.name }} {{ t("members") }}</p>
</template>
```

### 操作欄變體

#### 文字按鈕模式

適用於需要明確標示操作文字的場景：

```vue
<template #item.actions="{ item }">
  <div class="d-flex justify-end">
    <v-btn
      variant="elevated"
      color="primary"
      size="small"
      prepend-icon="mdi-cog"
      class="mr-2"
      @click="editDialogOpen(item)"
    >
      {{ t("editRoles") }}
    </v-btn>
    <v-btn
      variant="elevated"
      color="error"
      size="small"
      prepend-icon="mdi-delete"
      @click="removeDialogOpen(item)"
    >
      {{ t("remove") }}
    </v-btn>
  </div>
</template>
```

#### 圖示按鈕模式（UITooltipBtn）

適用於操作較多、需要節省空間的場景：

```vue
<template #item.actions="{ item }">
  <div class="d-flex">
    <UITooltipBtn
      color="primary"
      icon="mdi-eye"
      :tooltip="t('view')"
      @click="viewItem(item)"
    />
    <UITooltipBtn
      color="primary"
      icon="mdi-pencil"
      :tooltip="t('edit')"
      @click="editItem(item)"
    />
    <UITooltipBtn
      color="error"
      icon="mdi-delete"
      :tooltip="t('delete')"
      @click="deleteItem(item)"
    />
  </div>
</template>
```

### 主要操作按鈕變體

#### 分割按鈕（下拉選單）

```vue
<template #appendTitle>
  <div>
    <v-btn
      prepend-icon="mdi-plus"
      color="primary"
      style="border-radius: 4px 0 0 4px; margin-right: 1px"
      @click="primaryAction()"
    >
      {{ t("addMember") }}
    </v-btn>
    <v-menu>
      <template #activator="{ props }">
        <v-btn
          v-bind="props"
          variant="elevated"
          color="primary"
          class="px-1"
          style="min-width: 32px !important; border-radius: 0 4px 4px 0"
        >
          <v-icon>mdi-menu-down</v-icon>
        </v-btn>
      </template>
      <v-list slim>
        <v-list-item
          :title="t('alternativeAction')"
          prepend-icon="mdi-export"
          @click="alternativeAction()"
        />
      </v-list>
    </v-menu>
  </div>
</template>
```

---

## 2. 設定表單頁 {#settings-form}

多個設定區塊組成的表單頁面，每個區塊獨立儲存。

### 結構概覽

```
<v-skeleton-loader v-if="isLoading">       → 載入骨架屏
<div class="mx-auto d-flex flex-column ga-2" style="max-width: 800px">
  ├── <v-card-title class="pa-0">          → 頁面主標題（Icon + 文字）
  ├── <v-card variant="flat" rounded="lg">  → 設定區塊
  │   ├── <v-card-title class="border-b">  → 區塊標題（Icon + 文字）
  │   ├── <v-card-text>                    → 表單欄位
  │   └── <v-card-actions>                 → 儲存按鈕（靠右）
  └── <v-card variant="flat" rounded="lg"> → 其他設定區塊...
```

### 完整範例

```vue
<i18n lang="json">
{
  "zh-TW": {
    "basicSetting": "基本設定",
    "projectSetting": "專案名稱設定",
    "projectName": "專案名稱",
    "documents": "文件",
    "isVectorMode": "向量模式開啟PDF",
    "save": "儲存",
    "saveSuccess": "儲存成功"
  },
  "en": {
    "basicSetting": "Basic Settings",
    "projectSetting": "Project Name Setting",
    "projectName": "Project Name",
    "documents": "Document",
    "isVectorMode": "PDF Opened by Vector Mode",
    "save": "Save",
    "saveSuccess": "Saved Successfully"
  }
}
</i18n>

<template>
  <!-- 載入骨架屏 -->
  <v-skeleton-loader v-if="isLoading" type="card,card,card" />

  <!-- 主內容 -->
  <div
    v-else
    class="mx-auto d-flex flex-column ga-2"
    style="max-width: 800px"
  >
    <!-- 頁面主標題 -->
    <v-card-title class="pa-0">
      <v-icon start>mdi-cog-outline</v-icon>
      {{ t("basicSetting") }}
    </v-card-title>

    <!-- 設定區塊 1：專案名稱 -->
    <v-card variant="flat" rounded="lg">
      <v-card-title class="bg-secondary text-subtitle-1">
        <v-icon start>mdi-file-document-outline</v-icon>
        {{ t("projectSetting") }}
      </v-card-title>
      <v-card-text class="pb-0">
        <v-text-field
          v-model="service.formData.projectName"
          :label="t('projectName')"
          variant="outlined"
          density="compact"
          hide-details
          class="mb-1 mt-2"
        />
      </v-card-text>
      <v-card-actions>
        <v-spacer />
        <v-btn
          variant="elevated"
          color="primary"
          :loading="isLoadingSection1"
          :disabled="!isSection1Valid"
          @click="saveSection1"
        >
          {{ t("save") }}
        </v-btn>
      </v-card-actions>
    </v-card>

    <!-- 設定區塊 2：文件設定（條件顯示） -->
    <v-card v-if="hasDocModule" variant="flat" rounded="lg">
      <v-card-title class="bg-secondary text-subtitle-1">
        <v-icon start>mdi-file-outline</v-icon>
        {{ t("documents") }}
      </v-card-title>
      <v-card-text class="py-0">
        <v-checkbox
          v-model="service.formData.isVectorMode"
          :label="t('isVectorMode')"
          hide-details
        />
      </v-card-text>
      <v-card-actions>
        <v-spacer />
        <v-btn
          variant="elevated"
          color="primary"
          :loading="isLoadingSection2"
          @click="saveSection2"
        >
          {{ t("save") }}
        </v-btn>
      </v-card-actions>
    </v-card>
  </div>
</template>

<script setup lang="ts">
import { definePageMeta } from "#imports";
import { BasicSettingsService } from "~/services/BasicSettingsService";

definePageMeta({
  layout: "default-menu-item-permission-check",
});

const { t } = useI18n();
const service = reactive(new BasicSettingsService());
const $toast = useNuxtApp().$toast;

const isLoading = ref(false);
const isLoadingSection1 = ref(false);
const isLoadingSection2 = ref(false);

const hasDocModule = computed(() => {
  // 根據模組權限條件顯示
  return true;
});

const isSection1Valid = computed(() => {
  return service.formData.projectName.trim().length > 0;
});

async function saveSection1() {
  try {
    isLoadingSection1.value = true;
    await service.updateProjectName(service.formData.projectName);
    $toast.success(t("saveSuccess"));
  } catch (error: unknown) {
    console.error(error);
  } finally {
    isLoadingSection1.value = false;
  }
}

async function saveSection2() {
  try {
    isLoadingSection2.value = true;
    await service.updateDocSetting(service.formData.isVectorMode);
    $toast.success(t("saveSuccess"));
  } catch (error: unknown) {
    console.error(error);
  } finally {
    isLoadingSection2.value = false;
  }
}

onMounted(async () => {
  try {
    isLoading.value = true;
    await service.init();
  } catch (error: unknown) {
    console.error(error);
  } finally {
    isLoading.value = false;
  }
});
</script>
```

### 設定表單頁慣例

| 慣例 | 說明 |
|------|------|
| 最大寬度 | `max-width: 800px` + `mx-auto` |
| 區塊間距 | 父容器 `d-flex flex-column ga-2` |
| Card 屬性 | `variant="flat" rounded="lg"` |
| 區塊標題 | `<v-card-title class="bg-secondary text-subtitle-1">` + `<v-icon start>` + 文字 |
| 儲存按鈕 | `<v-card-actions>` 內靠右，`variant="elevated" color="primary"` |
| 載入態 | 初始載入用 `<v-skeleton-loader type="card,card,card">`，各區塊獨立 loading |
| 條件區塊 | 用 `v-if` 根據模組權限控制顯示 |

---

## 3. 個人資料/詳情頁 {#profile-detail}

展示資訊的 key-value 配對頁面，支援可選的編輯功能。

### 結構概覽

```
<v-container fluid class="pa-0">
  <v-row no-gutters>
    <v-col cols="12" md="10" lg="9" xl="6" class="mx-auto">
      ├── <v-card variant="flat" rounded="lg">  → 資訊卡片
      │   ├── <v-card-title class="border-b">  → 區塊標題（Icon + 文字）
      │   └── <v-card-text>
      │       └── <v-row>/<v-col>              → key-value 配對
      ├── <v-card variant="flat" rounded="lg" class="mt-4"> → 其他卡片...
```

### 完整範例

```vue
<i18n lang="json">
{
  "zh-TW": {
    "userInfo": "基本資料",
    "name": "名稱",
    "email": "電子郵件",
    "company": "所屬公司",
    "enabled": "啟用狀態",
    "preference": "偏好設定",
    "timeFormat": "時間格式",
    "fromNow": "相對時間",
    "general": "一般",
    "editName": "編輯名稱",
    "edit": "編輯"
  },
  "en": {
    "userInfo": "Profile",
    "name": "Name",
    "email": "Email",
    "company": "Company",
    "enabled": "Enable",
    "preference": "Preference",
    "timeFormat": "Time Format",
    "fromNow": "Relative",
    "general": "General",
    "editName": "Edit Name",
    "edit": "Edit"
  }
}
</i18n>

<template>
  <v-container fluid class="pa-0">
    <!-- 基本資料卡片 -->
    <v-row no-gutters>
      <v-col cols="12" md="10" lg="9" xl="6" class="mx-auto">
        <v-card variant="flat" rounded="lg">
          <v-card-title class="border-b">
            <v-icon start>mdi-account-outline</v-icon>
            {{ t("userInfo") }}
          </v-card-title>
          <v-card-text>
            <v-skeleton-loader
              v-if="!userInfo"
              min-width="100%"
              height="200"
              type="paragraph@3"
            />
            <v-container v-else>
              <v-row class="align-center text-subtitle-1 mb-3">
                <!-- 名稱（可編輯） -->
                <v-col :cols="4">{{ t("name") }}</v-col>
                <v-col
                  :cols="8"
                  class="d-flex align-center justify-space-between"
                >
                  <span>{{ userInfo.name }}</span>
                  <UITooltipBtn
                    color="secondary"
                    icon="mdi-pencil"
                    :tooltip="t('edit')"
                    @click="editDialog.isOpen = true"
                  />
                </v-col>

                <!-- Email（唯讀） -->
                <v-col :cols="4">{{ t("email") }}</v-col>
                <v-col :cols="8">
                  <span>{{ userInfo.email }}</span>
                </v-col>

                <!-- 公司 -->
                <v-col :cols="4">{{ t("company") }}</v-col>
                <v-col :cols="8">
                  <span>{{ userInfo.companyName }}</span>
                </v-col>

                <!-- 啟用狀態（Icon 展示） -->
                <v-col :cols="4">{{ t("enabled") }}</v-col>
                <v-col :cols="8">
                  <v-icon v-if="userInfo.isEnable" color="success">
                    mdi-check-circle
                  </v-icon>
                  <v-icon v-else color="error">mdi-close-circle</v-icon>
                </v-col>
              </v-row>
            </v-container>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <!-- 偏好設定卡片 -->
    <v-row no-gutters class="mt-4">
      <v-col cols="12" md="10" lg="9" xl="6" class="mx-auto">
        <v-card variant="flat" rounded="lg">
          <v-card-title class="border-b">
            <v-icon start>mdi-cog-outline</v-icon>
            {{ t("preference") }}
          </v-card-title>
          <v-card-text>
            <v-container>
              <v-row class="align-center">
                <v-col :cols="3">
                  <span class="text-subtitle-1">{{ t("timeFormat") }}</span>
                </v-col>
                <v-col :cols="9">
                  <v-radio-group
                    :model-value="userInfo?.setting?.isTimeFromNow"
                    mandatory
                    hide-details
                    inline
                    color="primary"
                    @update:modelValue="changeTimeFormat"
                  >
                    <v-radio :label="t('fromNow')" :value="true" />
                    <v-radio :label="t('general')" :value="false" />
                  </v-radio-group>
                </v-col>
              </v-row>
            </v-container>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>

  <!-- 編輯 Dialog -->
  <UIDialog
    v-model="editDialog.isOpen"
    :type="DialogType.Input"
    :is-loading="editDialog.isLoading"
    :title-text="t('editName')"
    persistent
    @click:cancel="editDialog.isOpen = false"
    @confirm="onEditDone"
  >
    <template #content>
      <v-text-field
        v-model="editDialog.name"
        class="mt-2"
        hide-details
      />
    </template>
  </UIDialog>
</template>

<script setup lang="ts">
import { DialogType } from "~ui/domain-classes/universal/dialog";
import { useUserStore } from "~stores/stores/user";

const { t } = useI18n();
const { userInfo, updateUserInfoName } = useUserStore();

const editDialog = reactive({
  isOpen: false,
  isLoading: false,
  name: "",
});

watch(
  () => editDialog.isOpen,
  (val) => {
    if (val) editDialog.name = userInfo?.name ?? "";
  },
);

async function onEditDone() {
  try {
    editDialog.isLoading = true;
    await updateUserInfoName(userInfo.id, editDialog.name);
  } catch (error: unknown) {
    console.error(error);
  } finally {
    editDialog.isLoading = false;
    editDialog.isOpen = false;
  }
}

async function changeTimeFormat(value: boolean) {
  // 更新偏好設定
}
</script>
```

### 個人資料頁慣例

| 慣例 | 說明 |
|------|------|
| 響應式寬度 | `cols="12" md="10" lg="9" xl="6"` + `mx-auto` |
| Card 屬性 | `variant="flat" rounded="lg"` |
| 區塊標題 | `<v-card-title class="border-b">` + `<v-icon start>` + 文字 |
| key-value 比例 | label `cols="4"` + value `cols="8"`（或 `3:9`） |
| 卡片間距 | 後續卡片加 `class="mt-4"` |
| 載入態 | `<v-skeleton-loader type="paragraph@3">` |
| 編輯功能 | `UITooltipBtn` 在 value 列右側，搭配 `UIDialog` |
| 狀態展示 | 用 `v-icon` (check/close) 或 `v-chip` (color) |

---

## 4. 內容列表頁 {#content-list}

卡片式內容列表，支援篩選、無限捲動、空狀態處理。

### 結構概覽

```
<div style="overflow: auto; position: absolute; inset: 0" class="px-4 pb-4">
  <div style="max-width: 900px" class="mx-auto w-100">
    ├── <v-card variant="tonal" color="primary">  → Header 區
    │   ├── 標題 + 副標題
    │   ├── 操作按鈕（重新整理 + 篩選）
    │   └── <v-chip-group>                         → 快速篩選
    ├── <v-infinite-scroll>                        → 內容列表
    │   └── <ItemCard v-for>
    └── 空狀態卡片
```

### 完整範例

```vue
<i18n lang="json">
{
  "zh-TW": {
    "latestAnnouncements": "最新公告",
    "stayUpdated": "掌握最新消息",
    "all": "全部",
    "unread": "未讀",
    "hasAttachment": "有附件",
    "filters": "篩選",
    "reload": "重新載入",
    "noMore": "沒有更多了",
    "noData": "目前沒有資料",
    "noDataDescription": "目前沒有資料，請稍後再查看！",
    "noMatch": "沒有符合條件的結果",
    "noMatchDescription": "請嘗試調整篩選條件",
    "resetFilters": "重置篩選"
  },
  "en": {
    "latestAnnouncements": "Latest Announcements",
    "stayUpdated": "Stay updated with the latest news",
    "all": "All",
    "unread": "Unread",
    "hasAttachment": "Has Attachment",
    "filters": "Filters",
    "reload": "Reload",
    "noMore": "No more items",
    "noData": "No data available",
    "noDataDescription": "There is no data at the moment. Check back later!",
    "noMatch": "No matching results",
    "noMatchDescription": "Please try adjusting your filters",
    "resetFilters": "Reset Filters"
  }
}
</i18n>

<template>
  <div
    class="position-absolute px-4 pb-4"
    style="overflow: auto; top: 0; left: 0; right: 0; bottom: 0"
  >
    <div style="max-width: 900px" class="mx-auto w-100">
      <!-- Header 區：彩色背景卡片 -->
      <v-card
        color="primary"
        variant="tonal"
        rounded="lg"
        class="pa-6 mt-4"
      >
        <!-- 標題 + 操作按鈕 -->
        <div class="d-flex align-center justify-space-between mb-4">
          <div>
            <h1 class="text-h5 font-weight-bold text-secondary ma-0">
              {{ t("latestAnnouncements") }}
            </h1>
            <p class="text-body-2 text-grey ma-0">
              {{ t("stayUpdated") }}
            </p>
          </div>
          <div class="d-flex ga-2">
            <UITooltipBtn
              icon="mdi-refresh"
              color="secondary"
              variant="tonal"
              :loading="isFetchingData"
              :tooltip="t('reload')"
              @click="applyFilters()"
            />
            <v-badge
              :model-value="Boolean(appliedFiltersCount)"
              :content="appliedFiltersCount"
              color="primary"
              offset-x="8"
              offset-y="8"
            >
              <v-btn
                prepend-icon="mdi-filter-variant"
                variant="tonal"
                color="secondary"
                class="px-4"
                @click="filterDialog.isOpen = true"
              >
                {{ t("filters") }}
              </v-btn>
            </v-badge>
          </div>
        </div>

        <!-- 快速篩選 Chip -->
        <v-chip-group v-model="activeChip" mandatory>
          <v-chip
            prepend-icon="mdi-view-list"
            color="primary"
            variant="tonal"
            filter
            class="px-4"
          >
            {{ t("all") }}
          </v-chip>
          <v-chip
            prepend-icon="mdi-eye-off-outline"
            color="primary"
            variant="tonal"
            filter
            class="px-4"
          >
            {{ t("unread") }}
          </v-chip>
          <v-chip
            prepend-icon="mdi-paperclip"
            color="primary"
            variant="tonal"
            filter
            class="px-4"
          >
            {{ t("hasAttachment") }}
          </v-chip>
        </v-chip-group>
      </v-card>

      <!-- 內容列表：無限捲動 -->
      <v-infinite-scroll
        v-if="service.items.length > 0"
        class="d-flex flex-column ga-4 pa-1"
        :empty-text="t('noMore')"
        @load="loadMore"
      >
        <template v-for="item in service.items" :key="item.id">
          <ItemCard :item="item" @click="openDetail(item)" />
        </template>
      </v-infinite-scroll>

      <!-- 空狀態 -->
      <v-card
        v-else-if="!isFetchingData"
        class="text-center py-12 px-8 mx-auto"
        variant="flat"
        color="transparent"
        rounded="lg"
        max-width="500"
      >
        <v-card-text>
          <v-icon
            :icon="hasActiveFilters ? 'mdi-filter-remove-outline' : 'mdi-bullhorn-outline'"
            size="120"
            color="primary"
            class="opacity-40 mb-6"
          />
          <h3 class="text-h6 font-weight-medium text-secondary mb-2">
            {{ hasActiveFilters ? t("noMatch") : t("noData") }}
          </h3>
          <p class="text-body-2 text-grey mb-6">
            {{ hasActiveFilters ? t("noMatchDescription") : t("noDataDescription") }}
          </p>
          <v-btn
            v-if="hasActiveFilters"
            variant="text"
            prepend-icon="mdi-filter-off"
            @click="resetFilters"
          >
            {{ t("resetFilters") }}
          </v-btn>
        </v-card-text>
      </v-card>
    </div>
  </div>

  <!-- 篩選 Dialog -->
  <!-- <FilterDialog v-model="filterDialog.isOpen" ... /> -->
</template>

<script setup lang="ts">
import { XxxService } from "~/services/XxxService";

definePageMeta({
  layout: "no-padding-menu-item-permission-check",
});

const { t } = useI18n();
const $toast = useNuxtApp().$toast;
const service = reactive(new XxxService());
const isFetchingData = ref(false);
const activeChip = ref(0);

const filterDialog = reactive({ isOpen: false });

const appliedFiltersCount = computed(() => {
  // 計算啟用的篩選條件數量
  return 0;
});

const hasActiveFilters = computed(() => appliedFiltersCount.value > 0);

const loadMore = async ({ done }: { done: Function }) => {
  if (service.items.length >= service.totalItemNum) {
    done("empty");
    return;
  }
  try {
    isFetchingData.value = true;
    service.tableOptions.page++;
    await service.loadItems();
    done(service.items.length < service.totalItemNum ? "ok" : "empty");
  } catch (error: unknown) {
    console.error(error);
    done("error");
  } finally {
    isFetchingData.value = false;
  }
};

async function applyFilters() {
  try {
    isFetchingData.value = true;
    service.tableOptions.page = 1;
    await service.loadItems();
  } catch (error: unknown) {
    $toast.error(String(error));
  } finally {
    isFetchingData.value = false;
  }
}

async function resetFilters() {
  service.resetFilters();
  await applyFilters();
}

function openDetail(item: any) {
  // 打開詳情
}

onMounted(async () => {
  await service.loadItems();
});
</script>
```

### 內容列表頁慣例

| 慣例 | 說明 |
|------|------|
| 容器定位 | `position-absolute` + `overflow: auto` + `inset: 0` |
| 最大寬度 | `max-width: 900px` + `mx-auto` |
| Header 卡片 | `v-card variant="tonal" color="primary" rounded="lg"` 彩色背景 |
| 快速篩選 | `v-chip-group` + `mandatory` + `filter` |
| 無限捲動 | `v-infinite-scroll` + `@load` callback 搭配 `done()` |
| 空狀態 | 區分「無資料」與「篩選無結果」，後者顯示重置按鈕 |
| Layout | 通常使用 `no-padding-menu-item-permission-check` |

---

## 通用慣例

### Script Setup 順序

```typescript
// 1. definePageMeta
definePageMeta({ layout: "default-menu-item-permission-check" });

// 2. Imports
import { DialogType } from "~ui/domain-classes/universal/dialog";
import { type SBDataTableHeader, SBTableHeaderType } from "~ui/utils/dataTableHeader";
import { XxxService } from "~/services/XxxService";

// 3. Service 初始化
const service = reactive(new XxxService());

// 4. 工具函式
const { t } = useI18n();
const $toast = useNuxtApp().$toast;
const route = useRoute();
const router = useRouter();

// 5. Loading 狀態
const isLoading = ref(false);

// 6. Dialog 狀態
const addDialog = reactive({ isOpen: false, isLoading: false });

// 7. Computed (headers, 衍生狀態)
const headers = computed<SBDataTableHeader[]>(() => [...]);

// 8. Lifecycle
onMounted(() => { loadData(); });

// 9. Methods
async function loadData() { ... }
```

### i18n 規範

- 每個頁面必須包含 `<i18n lang="json">` 區塊
- 必須同時支援 `zh-TW` 和 `en`（或 `en-US`）
- 所有使用者可見文字都必須使用 `t("key")` 引用
- Dialog 的通用訊息使用全域 key：`t("dialog.add_successfully")` 等

### Dialog 狀態管理

```typescript
// 標準 dialog reactive 結構
const dialog = reactive<{
  isOpen: boolean;
  isLoading: boolean;
  item: ItemType | null;
}>({
  isOpen: false,
  isLoading: false,
  item: null,
});
```

### 頁面標題必須包含 Icon

| 頁面類型 | 標題位置 | Icon 用法 |
|---------|---------|----------|
| DataTable | `#title` slot | `<v-icon start>` |
| 設定表單 | `<v-card-title class="pa-0">` | `<v-icon start>` |
| 個人資料 | 各卡片 `<v-card-title class="border-b">` | `<v-icon start>` |
| 內容列表 | Header 卡片內 `<h1>` | 不使用 Icon，用文字標題 |

### Card Actions 佈局

Dialog 或卡片底部的操作列，靠右對齊，取消在左、確認在右：

```vue
<v-card-actions>
  <v-spacer />
  <v-btn
    variant="tonal"
    color="secondary"
    @click="dialog.isOpen = false"
  >
    {{ t("cancel") }}
  </v-btn>
  <v-btn
    variant="elevated"
    color="primary"
    @click="confirm"
  >
    {{ t("confirm") }}
  </v-btn>
</v-card-actions>
```

| 按鈕類型 | variant | color | size |
|---------|---------|-------|------|
| 主要動作（新增、儲存、確認） | `elevated` | `primary` | `small` |
| 次要動作（取消、篩選） | `tonal` | `secondary` | `small` |
| 低強調（重置） | `text` | — | `small` |
| 危險動作（刪除、移除） | `elevated` | `error` | `small` |
