# Nuxt Content 手冊輸出範本

生成手冊時，使用以下範本結構。將 `{placeholder}` 替換為實際內容。

## 檔案範本

```markdown
---
title: "{title}"
description: "{description}"
date: "{YYYY-MM-DD}"
category: "manual"
navigation:
  title: "{short_title}"
---

# {title}

{overview_paragraph}

## 目錄

::list{type="info"}
- [Step 1: {step_1_name}](#step-1-{step_1_slug})
- [Step 2: {step_2_name}](#step-2-{step_2_slug})
- [Step 3: {step_3_name}](#step-3-{step_3_slug})
::

---

## Step 1: {step_name}

{step_description}

![Step 1: {step_name}](/{slug}/screenshots/01-{step_slug}.png)

**操作說明：**

1. {annotation_1_label}
2. {annotation_2_label}
3. {annotation_3_label}

---

## Step 2: {step_name}

{step_description}

![Step 2: {step_name}](/{slug}/screenshots/02-{step_slug}.png)

**操作說明：**

1. {annotation_1_label}
2. {annotation_2_label}

---

（重複以上結構直到所有步驟完成）
```

## 檔案路徑規則

### Markdown 文件位置
```
content/docs/manual/{slug}/index.md
```

### 截圖檔案位置
```
public/docs/manual/{slug}/screenshots/01-{step-slug}.png
public/docs/manual/{slug}/screenshots/02-{step-slug}.png
```

### 圖片引用路徑（在 Markdown 中）
```markdown
![描述](/docs/manual/{slug}/screenshots/01-{step-slug}.png)
```

> Nuxt Content 的圖片路徑對應 `public/` 目錄下的檔案，
> 所以 Markdown 中寫 `/docs/manual/...` 會對應到 `public/docs/manual/...`。

## Slug 命名規則

- 手冊 slug：從標題或 URL 取英文關鍵字，小寫 kebab-case
  - 例：「任務管理操作手冊」→ `task-management`
  - 例：`https://example.com/user-settings` → `user-settings`
- 步驟 slug：從步驟名稱取英文動作 + 名詞，小寫 kebab-case
  - 例：「檢視任務列表」→ `view-task-list`
  - 例：「開啟新增對話框」→ `open-create-dialog`

## 截圖檔名格式

```
{序號:02d}-{step-slug}.png
```

範例：
- `01-view-task-list.png`
- `02-open-create-dialog.png`
- `03-fill-task-form.png`
- `04-confirm-success.png`

## 注意事項

- 每個步驟的截圖必須包含可見的標註（紅色圓圈編號、橘色框選）
- 操作說明的編號應與截圖中的 marker 編號對應
- 圖片 alt text 應包含步驟編號和名稱，方便 SEO 和無障礙使用
- 如果使用者的 Nuxt Content 專案有不同的目錄結構，在 Phase 1 詢問確認路徑
