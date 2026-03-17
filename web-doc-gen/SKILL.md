---
name: web-doc-gen
description: 自動生成網頁操作手冊，含標註截圖。使用此技能當被要求生成手冊、建立操作文件、截圖標註、web-doc-gen 時。
---

# Web Doc Gen - 網頁操作手冊生成器

自動導航網頁、標註截圖、生成 Nuxt Content 相容的操作手冊。

## 使用方式

```
/web-doc-gen [需求描述]
```

範例：
```
/web-doc-gen 為 https://example.com/dashboard 生成操作手冊，涵蓋：登入、檢視報表、匯出資料
/web-doc-gen 為 http://localhost:3000/tasks 建立操作文件，包含新增、編輯、刪除任務流程
```

## 執行流程

### Phase 1：需求分析與探索

1. 解析 `$ARGUMENTS` 中的需求：
   - 目標 URL
   - 要涵蓋的功能 / 操作流程
   - 手冊標題（可從 URL 或功能推斷）
2. 詢問使用者確認輸出路徑（預設：`content/docs/manual/{slug}/` 和 `public/docs/manual/{slug}/screenshots/`）
3. 使用 `mcp__playwright__browser_navigate` 導航到目標頁面
4. 使用 `mcp__playwright__browser_resize` 設定視窗為 **1280 x 800**
5. 使用 `mcp__playwright__browser_snapshot` 取得頁面結構，分析可操作的 UI 元素
6. 規劃操作步驟序列（每個步驟包含：動作、目標元素、要標註的元素清單）

### Phase 2：逐步截圖與標註

對每個步驟重複以下流程：

#### Step 2a：執行操作

根據步驟類型使用對應的 Playwright MCP 工具：

| 操作類型 | MCP 工具 |
|---------|----------|
| 導航頁面 | `mcp__playwright__browser_navigate` |
| 點擊元素 | `mcp__playwright__browser_click` |
| 輸入文字 | `mcp__playwright__browser_type` |
| 選擇選項 | `mcp__playwright__browser_select_option` |
| 填寫表單 | `mcp__playwright__browser_fill_form` |

#### Step 2b：等待頁面穩定

```
mcp__playwright__browser_wait_for
- text: "預期出現的文字"   （等待特定文字出現）
- time: 1                  （或等待 1 秒）
```

#### Step 2c：確保目標元素在視窗內

```
mcp__playwright__browser_evaluate
function: "(element) => { element.scrollIntoView({ behavior: 'instant', block: 'center' }); }"
ref: "目標元素的 ref"
element: "目標元素描述"
```

#### Step 2d：注入標註

使用 `mcp__playwright__browser_run_code` 注入標註 DOM overlay。

**重要**：注入函式的完整程式碼在 `references/annotation-injection.md` 中。

三種標註類型：

| 類型 | 視覺效果 | 使用時機 |
|------|----------|----------|
| `highlight` | 橘色邊框 + 半透明背景 + 文字標籤 | 框選區域（表格、側邊欄、對話框） |
| `marker` | 紅色編號圓圈 ① ② ③ + 標籤文字 | 標記點擊位置或輸入位置 |
| `arrow` | 紅色向下箭頭 | 指示特定小元素 |

呼叫方式：

```javascript
mcp__playwright__browser_run_code
code: `async (page) => {
  await page.evaluate((annotations) => {
    // 清除舊標註
    document.querySelectorAll('[data-doc-annotation]').forEach(el => el.remove());

    const container = document.createElement('div');
    container.setAttribute('data-doc-annotation', 'container');
    container.style.cssText = 'position:fixed; inset:0; z-index:2147483647; pointer-events:none;';
    document.body.appendChild(container);

    annotations.forEach(ann => {
      const target = document.querySelector(ann.selector);
      if (!target) return;
      const rect = target.getBoundingClientRect();

      if (ann.type === 'highlight') {
        const overlay = document.createElement('div');
        overlay.setAttribute('data-doc-annotation', 'highlight');
        overlay.style.cssText = \`
          position:fixed;
          top:\${rect.top - 4}px; left:\${rect.left - 4}px;
          width:\${rect.width + 8}px; height:\${rect.height + 8}px;
          border:3px solid #FF6B35; border-radius:6px;
          background:rgba(255,107,53,0.08);
        \`;
        container.appendChild(overlay);
        if (ann.label) {
          const lbl = document.createElement('div');
          lbl.setAttribute('data-doc-annotation', 'label');
          lbl.textContent = ann.label;
          lbl.style.cssText = \`
            position:fixed; top:\${rect.bottom + 6}px; left:\${rect.left}px;
            background:#FF6B35; color:#fff; padding:2px 8px;
            border-radius:4px; font:600 12px -apple-system,sans-serif; white-space:nowrap;
          \`;
          container.appendChild(lbl);
        }
      }

      if (ann.type === 'marker') {
        const badge = document.createElement('div');
        badge.setAttribute('data-doc-annotation', 'marker');
        badge.textContent = String(ann.number);
        badge.style.cssText = \`
          position:fixed; top:\${rect.top - 12}px; left:\${rect.right - 12}px;
          width:28px; height:28px; background:#E63946; color:#fff;
          border-radius:50%; display:flex; align-items:center; justify-content:center;
          font:700 14px -apple-system,sans-serif;
          border:2px solid #fff; box-shadow:0 2px 8px rgba(0,0,0,0.3);
        \`;
        container.appendChild(badge);
        if (ann.label) {
          const lbl = document.createElement('div');
          lbl.setAttribute('data-doc-annotation', 'marker-label');
          lbl.textContent = ann.number + '. ' + ann.label;
          lbl.style.cssText = \`
            position:fixed; top:\${rect.top - 10}px; left:\${rect.right + 20}px;
            background:rgba(230,57,70,0.95); color:#fff; padding:3px 10px;
            border-radius:4px; font:600 12px -apple-system,sans-serif;
            white-space:nowrap; box-shadow:0 1px 4px rgba(0,0,0,0.2);
          \`;
          container.appendChild(lbl);
        }
      }

      if (ann.type === 'arrow') {
        const arrow = document.createElement('div');
        arrow.setAttribute('data-doc-annotation', 'arrow');
        arrow.style.cssText = \`
          position:fixed; top:\${rect.top - 30}px;
          left:\${rect.left + rect.width/2 - 10}px;
          width:0; height:0;
          border-left:10px solid transparent;
          border-right:10px solid transparent;
          border-top:16px solid #E63946;
        \`;
        container.appendChild(arrow);
      }
    });
  }, ${JSON.stringify(annotations)});
}`
```

其中 `annotations` 是標註配置陣列，例如：

```json
[
  { "selector": ".data-table", "type": "highlight", "label": "資料列表" },
  { "selector": "#add-btn", "type": "marker", "number": 1, "label": "點擊新增" },
  { "selector": ".settings-icon", "type": "arrow" }
]
```

**如何決定標註的 selector**：
1. 先用 `browser_snapshot` 取得頁面的 accessibility tree
2. 從 snapshot 中找到要標註的元素的 ref
3. 用 `browser_evaluate` 取得該元素的實際 CSS selector 或 data 屬性
4. 或直接使用頁面上可識別的 CSS class / id / data-testid

#### Step 2e：截圖

```
mcp__playwright__browser_take_screenshot
filename: "{step_number:02d}-{step-slug}.png"
type: "png"
```

截圖自動保存到工作目錄。之後需要移動到正確的輸出路徑。

#### Step 2f：清除標註

```javascript
mcp__playwright__browser_run_code
code: `async (page) => {
  await page.evaluate(() => {
    document.querySelectorAll('[data-doc-annotation]').forEach(el => el.remove());
  });
}`
```

### Phase 3：生成 Nuxt Content Markdown

根據 `templates/doc-template.md` 範本，生成手冊文件：

1. 建立輸出目錄結構：
   ```
   {content_dir}/{slug}/index.md        # Nuxt Content Markdown
   {public_dir}/{slug}/screenshots/     # 截圖檔案
   ```

2. 將截圖移動到 `public` 目錄下

3. 撰寫 Markdown 文件，包含：
   - YAML frontmatter（title, description, date, category）
   - 每個步驟的標題、說明、截圖引用、操作說明清單
   - 圖片路徑使用 `/{slug}/screenshots/xx-name.png` 格式

### Phase 4：清理暫存截圖

截圖流程中，Playwright 會將截圖存到工作目錄（預設為專案根目錄或 Playwright 的暫存路徑）。
將截圖移動到 `public/` 目標路徑後，必須清理工作目錄中的暫存檔案：

```bash
# 刪除工作目錄中的暫存截圖（已移動到 public/ 的原始檔案）
rm -f 01-*.png 02-*.png 03-*.png ...
```

**清理規則**：
- 只刪除本次生成的截圖檔案（根據檔名 pattern `{序號:02d}-*.png` 匹配）
- 在移動到目標路徑後才執行刪除
- 確認目標路徑的檔案存在後，才刪除暫存檔
- 如果 Playwright 截圖產生了額外的暫存檔（如 `page-*.png`），一併清理

```bash
# 完整清理指令範例
rm -f ./01-*.png ./02-*.png ./03-*.png ./page-*.png
```

### Phase 5：驗證

1. 確認所有截圖檔案存在於 `public/` 目標路徑
2. 確認 Markdown 中的圖片路徑都對應到實際檔案
3. 確認工作目錄沒有殘留的暫存截圖
4. 輸出生成摘要：幾個步驟、幾張截圖、文件路徑

## 邊界情況處理

| 情境 | 處理方式 |
|------|----------|
| 元素在視窗外 | 先 `scrollIntoView({ block: 'center' })` 再標註 |
| Modal / Dialog | 先 click 開啟 → `wait_for` 確認出現 → 再注入標註 |
| 動態內容未載入 | `browser_wait_for` 等待目標文字出現 |
| 需要登入 | 在第一步處理登入流程，使用 `browser_fill_form` |
| 全頁長截圖 | 使用 `fullPage: true`，標註改用 `position: absolute` |
| 輸出路徑不確定 | 開始前詢問使用者確認 content 和 public 目錄位置 |

## 重要注意事項

- **每個步驟截圖前必須注入標註，截圖後必須清除標註**
- 截圖檔名格式：`{序號:02d}-{步驟英文縮寫}.png`（如 `01-view-task-list.png`）
- 標註的 `selector` 要盡量精確，避免選到多個元素
- 如果 `querySelector` 找不到元素，標註會被跳過，不會報錯
- 視窗大小固定為 1280x800，確保截圖一致性
