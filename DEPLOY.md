# 年度行銷企劃系統 — 部署說明

## 部署資訊

| 項目 | 內容 |
|------|------|
| 部署平台 | GitHub Pages |
| GitHub Repo | https://github.com/jacky5453-beep/derlife-marketing-planner |
| 線上網址 | https://jacky5453-beep.github.io/derlife-marketing-planner/ |
| Firebase 專案 | derlife-audit |
| Firestore Collection | `marketing-planner-data/main`（資料）、`marketing-planner-whitelist`（白名單） |
| 最後部署日期 | 2026-04-17 |

---

## 部署指令

```bash
cd "/Users/jacky/Desktop/claude/claude code/年度行銷企劃系統"

# 確保 index.html 與 marketing-planner-v2.html 同步
cp marketing-planner-v2.html index.html

git add index.html logo.png
git commit -m "更新描述"
git push origin main
# GitHub Pages 自動部署（約 1~2 分鐘生效）
```

---

## Firebase 初始設定（首次部署必做）

### 1. 啟用 Google 登入
1. 前往 [Firebase Console](https://console.firebase.google.com/) → `derlife-audit`
2. 左側選單 → **Authentication** → **Sign-in method**
3. 啟用 **Google** provider

### 2. 新增授權網域
1. Authentication → **Settings** → **Authorized domains**
2. 新增：`jacky5453-beep.github.io`

### 3. Firestore 安全規則
在 Firestore → Rules 加入：

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // 白名單 collection：登入使用者可讀，寫入需驗證
    match /marketing-planner-whitelist/{email} {
      allow read: if request.auth != null;
      allow write: if request.auth != null;
    }
    // 資料 collection：登入使用者可讀寫
    match /marketing-planner-data/{doc} {
      allow read, write: if request.auth != null;
    }
  }
}
```

---

## 帳號管理說明

- **首位登入者**自動成為管理員
- 管理員可至「帳號管理」頁新增／移除授權帳號
- 白名單儲存於 Firestore `marketing-planner-whitelist` collection

---

## 環境變數

無（Firebase config 已內嵌在 HTML 中，使用公開 API Key）

---

## 系統架構

- **類型**：單一 HTML 檔案（Type A）
- **入口**：`index.html`（從 `marketing-planner-v2.html` 複製）
- **資料庫**：Firebase Firestore（`derlife-audit` 專案）
- **認證**：Firebase Authentication + Google provider
- **資料同步**：In-memory cache + 非同步 Firestore 寫入
