# 標註注入 JavaScript 參考

本文件包含在網頁截圖前注入 DOM 標註 overlay 的完整 JavaScript 函式。
所有函式透過 `mcp__playwright__browser_run_code` 執行。

## 注入標註函式

以下為完整的標註注入程式碼。使用時將 `ANNOTATIONS_ARRAY` 替換為實際的標註配置陣列。

```javascript
async (page) => {
  await page.evaluate((annotations) => {
    // 清除所有舊標註
    document.querySelectorAll('[data-doc-annotation]').forEach(el => el.remove());

    // 建立固定定位容器
    const container = document.createElement('div');
    container.setAttribute('data-doc-annotation', 'container');
    container.style.cssText = `
      position: fixed;
      top: 0; left: 0; right: 0; bottom: 0;
      z-index: 2147483647;
      pointer-events: none;
    `;
    document.body.appendChild(container);

    annotations.forEach((ann) => {
      const target = document.querySelector(ann.selector);
      if (!target) {
        console.warn(`[doc-annotation] selector not found: ${ann.selector}`);
        return;
      }
      const rect = target.getBoundingClientRect();

      // ─── highlight：橘色邊框 + 半透明背景 + 文字標籤 ───
      if (ann.type === 'highlight') {
        const overlay = document.createElement('div');
        overlay.setAttribute('data-doc-annotation', 'highlight');
        overlay.style.cssText = `
          position: fixed;
          top: ${rect.top - 4}px;
          left: ${rect.left - 4}px;
          width: ${rect.width + 8}px;
          height: ${rect.height + 8}px;
          border: 3px solid #FF6B35;
          border-radius: 6px;
          background: rgba(255, 107, 53, 0.08);
          pointer-events: none;
        `;
        container.appendChild(overlay);

        if (ann.label) {
          const labelEl = document.createElement('div');
          labelEl.setAttribute('data-doc-annotation', 'label');
          labelEl.textContent = ann.label;
          labelEl.style.cssText = `
            position: fixed;
            top: ${rect.bottom + 6}px;
            left: ${rect.left}px;
            background: #FF6B35;
            color: #ffffff;
            padding: 2px 8px;
            border-radius: 4px;
            font-size: 12px;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
            font-weight: 600;
            white-space: nowrap;
            pointer-events: none;
          `;
          container.appendChild(labelEl);
        }
      }

      // ─── marker：紅色編號圓圈 + 標籤文字 ───
      if (ann.type === 'marker') {
        const badge = document.createElement('div');
        badge.setAttribute('data-doc-annotation', 'marker');
        badge.textContent = String(ann.number);
        badge.style.cssText = `
          position: fixed;
          top: ${rect.top - 12}px;
          left: ${rect.right - 12}px;
          width: 28px;
          height: 28px;
          background: #E63946;
          color: #ffffff;
          border-radius: 50%;
          display: flex;
          align-items: center;
          justify-content: center;
          font-size: 14px;
          font-weight: 700;
          font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
          border: 2px solid #ffffff;
          box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
          pointer-events: none;
        `;
        container.appendChild(badge);

        if (ann.label) {
          const labelEl = document.createElement('div');
          labelEl.setAttribute('data-doc-annotation', 'marker-label');
          labelEl.textContent = `${ann.number}. ${ann.label}`;
          labelEl.style.cssText = `
            position: fixed;
            top: ${rect.top - 10}px;
            left: ${rect.right + 20}px;
            background: rgba(230, 57, 70, 0.95);
            color: #ffffff;
            padding: 3px 10px;
            border-radius: 4px;
            font-size: 12px;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
            font-weight: 600;
            white-space: nowrap;
            box-shadow: 0 1px 4px rgba(0, 0, 0, 0.2);
            pointer-events: none;
          `;
          container.appendChild(labelEl);
        }
      }

      // ─── arrow：紅色向下箭頭 ───
      if (ann.type === 'arrow') {
        const arrow = document.createElement('div');
        arrow.setAttribute('data-doc-annotation', 'arrow');
        arrow.style.cssText = `
          position: fixed;
          top: ${rect.top - 30}px;
          left: ${rect.left + rect.width / 2 - 10}px;
          width: 0;
          height: 0;
          border-left: 10px solid transparent;
          border-right: 10px solid transparent;
          border-top: 16px solid #E63946;
          pointer-events: none;
        `;
        container.appendChild(arrow);

        if (ann.label) {
          const labelEl = document.createElement('div');
          labelEl.setAttribute('data-doc-annotation', 'arrow-label');
          labelEl.textContent = ann.label;
          labelEl.style.cssText = `
            position: fixed;
            top: ${rect.top - 52}px;
            left: ${rect.left + rect.width / 2 - 40}px;
            background: rgba(230, 57, 70, 0.95);
            color: #ffffff;
            padding: 3px 10px;
            border-radius: 4px;
            font-size: 12px;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
            font-weight: 600;
            white-space: nowrap;
            box-shadow: 0 1px 4px rgba(0, 0, 0, 0.2);
            pointer-events: none;
          `;
          container.appendChild(labelEl);
        }
      }
    });

    return { success: true, count: annotations.length };
  }, ANNOTATIONS_ARRAY);
}
```

## 標註配置格式

```typescript
interface Annotation {
  selector: string;    // CSS selector，精確指向目標元素
  type: 'highlight' | 'marker' | 'arrow';
  label?: string;      // 標籤文字（可選）
  number?: number;     // 編號，僅 marker 類型需要
}
```

### 範例配置

```json
[
  {
    "selector": ".v-data-table",
    "type": "highlight",
    "label": "任務列表"
  },
  {
    "selector": "[data-testid='add-task-btn']",
    "type": "marker",
    "number": 1,
    "label": "點擊新增任務"
  },
  {
    "selector": "#settings-icon",
    "type": "marker",
    "number": 2,
    "label": "開啟設定"
  },
  {
    "selector": ".notification-bell",
    "type": "arrow",
    "label": "通知"
  }
]
```

## 清除標註函式

截圖後必須清除所有標註，避免影響後續操作：

```javascript
async (page) => {
  await page.evaluate(() => {
    document.querySelectorAll('[data-doc-annotation]').forEach(el => el.remove());
  });
}
```

## Scroll 確保元素可見

在注入標註前，確保目標元素在視窗內：

```javascript
async (page) => {
  await page.evaluate((selector) => {
    const el = document.querySelector(selector);
    if (el) {
      el.scrollIntoView({ behavior: 'instant', block: 'center' });
    }
  }, TARGET_SELECTOR);
}
```

## 全頁截圖模式

當需要 `fullPage: true` 截圖時，標註必須改用 `position: absolute`：

```javascript
// 在注入前，將容器改為 absolute 定位
container.style.cssText = `
  position: absolute;
  top: 0; left: 0;
  width: ${document.documentElement.scrollWidth}px;
  height: ${document.documentElement.scrollHeight}px;
  z-index: 2147483647;
  pointer-events: none;
`;

// 元素定位也改用 absolute，加上 scrollY/scrollX 偏移
const scrollX = window.scrollX;
const scrollY = window.scrollY;
// top: ${rect.top + scrollY - 4}px
// left: ${rect.left + scrollX - 4}px
```

## 取得元素 Selector 的輔助方式

如果從 `browser_snapshot` 只能取得 ref，可以用以下方式取得可用的 CSS selector：

```javascript
async (page) => {
  // 方法 1：透過元素的 id
  const id = await page.evaluate((ref) => {
    // 假設 ref 對應到某個元素
    return document.querySelector(ref)?.id;
  }, refValue);

  // 方法 2：透過 data 屬性
  const selector = await page.evaluate(() => {
    // 列出所有有 data-testid 的元素
    return [...document.querySelectorAll('[data-testid]')]
      .map(el => ({
        testid: el.getAttribute('data-testid'),
        tag: el.tagName,
        text: el.textContent?.substring(0, 30)
      }));
  });
}
```

## 標註顏色參考

| 用途 | 顏色 | Hex |
|------|------|-----|
| Highlight 邊框 | 橘色 | `#FF6B35` |
| Highlight 背景 | 淺橘半透明 | `rgba(255, 107, 53, 0.08)` |
| Marker 圓圈 | 紅色 | `#E63946` |
| Marker 邊框 | 白色 | `#FFFFFF` |
| Arrow 箭頭 | 紅色 | `#E63946` |
| 所有標籤文字 | 白色 | `#FFFFFF` |
