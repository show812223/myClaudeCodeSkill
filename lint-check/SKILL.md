---
name: lint-check
description: 執行程式碼風格檢查與自動修正。使用此技能當被要求 lint、檢查程式碼、修正風格時。
---

# Lint Check Skill

此技能用於執行 ESLint 程式碼風格檢查與自動修正。

## 執行流程

1. 執行 `pnpm lint` 檢查錯誤
2. 分析錯誤訊息
3. 如有錯誤，執行 `pnpm lint --fix`
4. 再次驗證修正結果
5. 報告修正摘要

## 指令

```bash
# 檢查整個專案
pnpm lint

# 檢查特定 package
pnpm --filter @syncobox/{package-name} lint

# 自動修正
pnpm --filter @syncobox/{package-name} lint --fix
```

## 主要檢查項目

### Vue 規則
- `vue/multi-word-component-names`
- `vue/no-mutating-props`
- `vue/require-default-prop`
- `vue/valid-v-slot`

### TypeScript 規則
- 型別安全
- 未使用變數
- 禁止 `any` 型別（除非特殊情況）

## 常見錯誤修正

### 未使用變數
```typescript
// ❌ 錯誤
const unused = 'value';

// ✅ 移除或使用該變數
```

### 缺少型別定義
```typescript
// ❌ 錯誤
const data: any = {};

// ✅ 正確
interface UserData {
  id: string;
  name: string;
}
const data: UserData = { id: '1', name: 'Test' };
```

### Props 未定義預設值
```typescript
// ❌ 警告
const props = defineProps<{
  title?: string;
}>();

// ✅ 使用 withDefaults
const props = withDefaults(defineProps<{
  title?: string;
}>(), {
  title: ''
});
```

## 完成檢查清單

- [ ] 執行 `pnpm lint` 無錯誤
- [ ] 無未使用的變數
- [ ] 無 `any` 型別
- [ ] 無 `console.log`（除了錯誤處理）

