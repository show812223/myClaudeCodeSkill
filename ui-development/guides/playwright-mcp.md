# Playwright MCP 頁面測試指南

使用 Playwright MCP 工具進行視覺化測試，確保頁面在瀏覽器中正確運作。

## 何時使用

- 頁面開發完成後進行視覺化驗證
- 測試使用者互動流程
- 檢查響應式佈局
- 驗證表單提交與 API 整合
- Debug UI 問題

## 測試流程

### Step 1: 導航至目標頁面

```text
mcp__playwright__browser_navigate
URL: http://localhost:3000/{page-path}
```

### Step 2: 取得頁面快照

```text
mcp__playwright__browser_snapshot
- 比 screenshot 更適合用於後續操作
- 返回可存取性快照，包含元素 ref
```

### Step 3: 執行互動測試

```text
點擊按鈕:
  mcp__playwright__browser_click
  - element: "提交按鈕"
  - ref: "button[ref='submit-btn']"

填寫表單:
  mcp__playwright__browser_type
  - element: "使用者名稱輸入框"
  - ref: "input[ref='username']"
  - text: "test@example.com"

選擇下拉選項:
  mcp__playwright__browser_select_option
  - element: "角色選擇"
  - ref: "select[ref='role']"
  - values: ["admin"]
```

### Step 4: 驗證結果

```text
等待特定文字出現:
  mcp__playwright__browser_wait_for
  - text: "儲存成功"

截圖保存:
  mcp__playwright__browser_take_screenshot
  - filename: "result.png"
```

## 常用 MCP 工具

| 操作 | 工具 | 用途 |
| ---- | ---- | ---- |
| 導航 | `browser_navigate` | 前往指定 URL |
| 快照 | `browser_snapshot` | 取得頁面結構 (推薦) |
| 截圖 | `browser_take_screenshot` | 保存視覺畫面 |
| 點擊 | `browser_click` | 點擊元素 |
| 輸入 | `browser_type` | 輸入文字 |
| 選擇 | `browser_select_option` | 選擇下拉選項 |
| 填表 | `browser_fill_form` | 批次填寫表單 |
| 等待 | `browser_wait_for` | 等待文字/消失/時間 |
| 按鍵 | `browser_press_key` | 按下鍵盤按鍵 |
| 懸停 | `browser_hover` | 滑鼠懸停 |
| Console | `browser_console_messages` | 查看 console 訊息 |
| Network | `browser_network_requests` | 查看網路請求 |
| 調整大小 | `browser_resize` | 調整視窗大小 |
| 關閉 | `browser_close` | 關閉瀏覽器 |

## 測試範例

```text
1. browser_navigate: http://localhost:3000/user-management
2. browser_snapshot
3. browser_click: element="新增使用者按鈕", ref="[data-testid='add-user-btn']"
4. browser_fill_form: name="測試使用者", email="test@example.com"
5. browser_click: element="提交按鈕", ref="[data-testid='submit-btn']"
6. browser_wait_for: text="新增成功"
7. browser_take_screenshot: filename="add-user-success.png"
8. browser_network_requests: 確認 POST /api/users 成功
```

## 檢查清單

- [ ] 頁面正確載入，無 console 錯誤
- [ ] 所有互動元素可點擊
- [ ] 表單驗證正確運作
- [ ] API 請求正確發送
- [ ] 成功/錯誤訊息正確顯示
- [ ] 響應式佈局正確 (用 `browser_resize` 測試)
