# Vue 元件模板

## 元件結構

```vue
<script setup lang="ts">
// 1. 型別導入 (從 shared packages 優先)
import type { User } from '~stores/types';

// 2. Props 定義 (使用 TypeScript 泛型)
const props = defineProps<{
  userId: string;
  editable?: boolean;
}>();

// 3. Emits 定義
const emit = defineEmits<{
  (e: 'save', data: FormData): void;
  (e: 'cancel'): void;
}>();

// 4. Model 定義 (雙向綁定)
const modelValue = defineModel<string>();

// 5. 響應式狀態 (優先使用 ref)
const loading = ref(false);
const errorMessage = ref('');

// 6. API 呼叫 (使用 useFetch)
const { data, pending, error } = await useFetch<User>('/api/user', {
  query: { id: props.userId }
});
</script>

<template>
  <v-card variant="outlined" class="ma-4">
    <v-card-title class="d-flex align-center justify-space-between">
      <span class="text-h6">標題</span>
    </v-card-title>

    <v-card-text>
      <div v-if="pending" class="d-flex justify-center pa-4">
        <v-progress-circular indeterminate color="primary" />
      </div>

      <template v-else>
        <!-- 元件內容 -->
      </template>
    </v-card-text>

    <v-card-actions class="d-flex justify-end">
      <v-btn variant="text" @click="emit('cancel')">取消</v-btn>
      <v-btn color="primary" :loading="loading">儲存</v-btn>
    </v-card-actions>
  </v-card>
</template>
```

## Vuetify 對應規則 (禁止使用原生 HTML)

| 禁止 | 使用 |
| ---- | ---- |
| `<button>` | `<v-btn>` |
| `<input>` | `<v-text-field>` |
| `<select>` | `<v-select>` |
| `<div>` (容器) | `<v-sheet>` / `<v-card>` |
| `<table>` | `<v-data-table>` |
| `<dialog>` | `<v-dialog>` |
| `<img>` | `<v-img>` |

## 必須遵守規則

1. 使用 `<script setup lang="ts">`
2. Props/Emits 使用 TypeScript 泛型定義
3. 禁止原生 HTML，使用 Vuetify 元件
4. 使用 Path Alias，不用相對路徑
5. 禁止使用 `any` 型別
