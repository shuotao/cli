# GWS CLI 專業部署與認證指南 (GWS0315)

本專案 (`gws`) 是基於 [googleworkspace/cli](https://github.com/googleworkspace/cli) 的個人封裝版。身為 SKILLS 開發者，應遵循以下正式部署與同步規範。

## 🛡️ 安全性規範 (Privacy & Security)
- **認證檔案：** 所有金鑰與 Token 均存儲於 `~/.config/gws/` 目錄中。
- **Git 隔離：** `.gitignore` 已配置過濾 `.env`、`*.enc`、`client_secret*.json`、`credentials*.json` 與 `token_cache.json`。
- **禁止提交：** 絕對禁止將 `~/.config/gws/` 內的任何檔案複製到專案目錄中並提交。

## 🚀 部署流程 (Deployment Workflow)

### 1. 初次安裝
```bash
git clone https://github.com/shuotao/cli.git
cd cli_project
git remote add upstream https://github.com/googleworkspace/cli.git
cargo build --release
```

### 2. 同步官方更新 (Sync Upstream)
```bash
git fetch upstream
git merge upstream/main
cargo build --release
```

## 🔑 解決 OAuth "Localhost 拒絕連線" 問題
當 CLI 在遠端或容器內執行時，瀏覽器無法連回 `localhost`。請使用以下「開發者手動換碼法」：

1. **取得手動授權連結：** 
   在登入指令中將 `redirect_uri` 指定為 `urn:ietf:wg:oauth:2.0:oob`（手動輸入碼模式）。
   *註：若當前 `gws` 版本不支援，請使用「網址列提取法」。*

2. **網址列提取法 (100% 成功)：**
   - 點擊一般的 `gws auth login --full` 產生的連結。
   - 完成授權後，當瀏覽器顯示「無法連上 localhost」時，**複製整個網址列**。
   - 網址中包含 `code=4/0Afr...`。
   - 使用以下指令手動換取 Token：
     ```bash
     curl -s -X POST https://oauth2.googleapis.com/token \
     -d code=[你的CODE] \
     -d client_id=[你的CLIENT_ID] \
     -d client_secret=[你的CLIENT_SECRET] \
     -d redirect_uri=http://localhost:[你的PORT] \
     -d grant_type=authorization_code
     ```
   - 將回傳的 `refresh_token` 手動寫入 `~/.config/gws/credentials.json`。

## 📋 常用驗證指令
- **檢查狀態：** `gws auth status`
- **讀取表單：** `gws drive files list --params '{"q": "mimeType = \"application/vnd.google-apps.form\""}'`
- **讀取郵件：** `gws gmail users messages list --params '{"userId": "me", "maxResults": 5}'`
\n## 📝 專案沿革與命名 (Project History)\n- **專案代號：** GWS0315\n- **GCP 後台專案 ID：** `dev2025-479802`\n- **說明：** 為避免多重用戶端造成的憑證混淆，開發團隊已於 2026/03/15 收斂配置，統一廢棄了原名為 GWS0315 的 GCP 專案與用戶端，全面沿用架構最完整的 `dev2025` 作為正式生產環境。維護人員在登入 Google Cloud Console 時，請務必切換至 `dev2025` 專案查看 API 與流量數據。
