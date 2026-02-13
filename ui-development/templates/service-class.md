# Service Class 模板

## 設計原則

| Vue 檔案職責 | Service Class 職責 |
| ------------ | ------------------ |
| UI 渲染 | 業務邏輯處理 |
| 事件綁定 | API 呼叫 |
| 響應式狀態展示 | 資料轉換與驗證 |
| 使用者互動回饋 | 狀態管理邏輯 |

## 檔案命名規則

```text
頁面: pages/user-management.vue
Service: services/UserManagementService.ts
測試: tests/services/UserManagementService.test.ts
```

## Service Class 模板

```typescript
// services/UserManagementService.ts
import type { User, UserCreateInput } from '~stores/types';
import { useUserStore } from '~stores/stores/user';
import { useCustomFetch } from '~api/composables/useCustomFetch';

export class UserManagementService {
  private userStore = useUserStore();

  // ============================================
  // 狀態
  // ============================================
  readonly loading = ref(false);
  readonly error = ref<string | null>(null);
  readonly users = ref<User[]>([]);

  // ============================================
  // Getters (計算屬性)
  // ============================================
  get activeUsers(): User[] {
    return this.users.value.filter(u => u.isActive);
  }

  get hasError(): boolean {
    return this.error.value !== null;
  }

  // ============================================
  // Actions (業務邏輯)
  // ============================================
  async fetchUsers(): Promise<void> {
    this.loading.value = true;
    this.error.value = null;

    try {
      const { data } = await useCustomFetch<User[]>('/api/users');
      this.users.value = data.value ?? [];
    } catch (e) {
      this.error.value = e instanceof Error ? e.message : '載入失敗';
    } finally {
      this.loading.value = false;
    }
  }

  async createUser(input: UserCreateInput): Promise<boolean> {
    this.loading.value = true;

    try {
      await useCustomFetch('/api/users', {
        method: 'POST',
        body: input,
      });
      await this.fetchUsers();
      return true;
    } catch (e) {
      this.error.value = e instanceof Error ? e.message : '建立失敗';
      return false;
    } finally {
      this.loading.value = false;
    }
  }

  async deleteUser(userId: string): Promise<boolean> {
    try {
      await useCustomFetch(`/api/users/${userId}`, {
        method: 'DELETE',
      });
      this.users.value = this.users.value.filter(u => u.id !== userId);
      return true;
    } catch (e) {
      this.error.value = e instanceof Error ? e.message : '刪除失敗';
      return false;
    }
  }

  // ============================================
  // 輔助方法 (資料轉換、驗證)
  // ============================================
  validateUserInput(input: UserCreateInput): string | null {
    if (!input.name?.trim()) return '名稱為必填';
    if (!input.email?.includes('@')) return 'Email 格式錯誤';
    return null;
  }

  resetError(): void {
    this.error.value = null;
  }
}
```

## 頁面使用 Service Class

```vue
<script setup lang="ts">
import { UserManagementService } from '~/services/UserManagementService';

// 建立 Service 實例
const service = new UserManagementService();

// 初始化載入
onMounted(() => {
  service.fetchUsers();
});

// 事件處理 - 僅呼叫 Service 方法
async function handleCreate(input: UserCreateInput) {
  const validationError = service.validateUserInput(input);
  if (validationError) {
    return;
  }
  await service.createUser(input);
}

async function handleDelete(userId: string) {
  await service.deleteUser(userId);
}
</script>

<template>
  <v-container>
    <v-progress-linear v-if="service.loading.value" indeterminate />

    <v-alert v-if="service.hasError" type="error" closable @click:close="service.resetError()">
      {{ service.error.value }}
    </v-alert>

    <v-data-table :items="service.users.value" :loading="service.loading.value">
      <template #item.actions="{ item }">
        <v-btn icon="mdi-delete" @click="handleDelete(item.id)" />
      </template>
    </v-data-table>
  </v-container>
</template>
```

## 規範

1. **所有業務邏輯必須在 Service Class 中**
2. **Vue 檔案只負責 UI 渲染與事件綁定**
3. **Service Class 必須達到 100% 測試覆蓋率**
4. **一個頁面對應一個 Service Class**
5. **共用邏輯抽取到獨立的 Service 或 Composable**
