# 為什麼要用這套組合拳
> [!WARNING] 注意
>本文實作略閒複雜與冗長，如果覺得太麻煩不想搞的話，可以直接使用 Obsidian 或其他筆記的付費服務就不用太折騰😋

網路上已經很多筆記系統了，且對於 Markdown 支援都不錯，為什麼還要搞這套組合拳呢？我也用過幾個筆記系統，但說真的不太習慣。有的較為簡單能紀錄文字，但我想紀錄一些技術筆記會需要用到圖片，以下是筆者使用過的筆記方法：
- Google Keeps
  可以快速紀錄片段文字，複雜有階層的資訊較難整理。
- Google Docs
  用於報告很優秀，圖片要能重複使用時較為麻煩，且不支援 Markdown。
- Evernote
  最開始免費版本支援裝置與筆記數量較多。後來改版後我將其轉移到 Notion。但我發現，就算整篇文章複製過去，圖片的URL依然是綁在 Evernote 的網域！等於圖片被綁架在 Evernote 中，轉移到 Notion 花了我很多時間。
- Notion
  本質上像是「強化型的 Evernote」或是多功能協作平台。當時網路上一致推薦，我也嘗試用看看。除了單純的文字筆記外，它還有強大的資料庫與個人管理...等等的花式功能。 但對於一個「只需要純粹紀錄，且絕對不希望圖片被平台網域綁架」的人來說，Notion 在這點與 Evernote 是一樣的，一旦你想把文章發佈到其他平台 (如 WordPress)，那些帶有時效性的圖片網址依然會讓你痛不欲生。

於是我就開始找 Open Source 解決方案，最終找到了本文的方案後，趕緊作筆記以免忘記。這個方案的好處絕對大於設定複雜冗長的缺點，我個人發現有以下幾個很難替代的優點：
1. 完全本機儲存
2. (可選擇性) Github 同步與版本管理
3. 使用高速的 Github CDN當圖床
4. Markdown本身只有文字大小，圖片全部在 Github 上，被綁架機率極低。

尤其是第4項，在 Wordpress 或其他 CMS/Blog (點部落 Dotblogs) 系統支援 Markdown 的狀況下，PO文時相當於只要貼上文字，圖片來源都是從 Github CDN 讀取，極大幅度的降低主機流量與壓力。如果是放在雲端的話，每個月流量費更是可以省不少(如果網站很多人看的話)。
  
# 準備工具
再開始建立多平台同步筆記系統前，`工欲善其事 必先利其器`是必要的！
 - [Obsidian](https://obsidian.md/)
   地表最強的本機端 Markdown 筆記軟體！沒有花里胡哨的強制雲端綁架，你的檔案就是你電腦裡的 `.md` 文字檔。速度快、外掛生態系超強，它是我們這套知識庫系統的「大腦」。
 - [Git](https://git-scm.com/)
   工程師的浪漫，版本控制的神器！在這裡我們不寫 code，而是把它當作筆記的「時光機」與「跨平台搬運工」。有了它，你在 Windows 和 Linux 之間的筆記就能無縫接軌，還自帶完美的備份機制。
 - [PicList](https://github.com/Kuingsmile/piclist)
   傳圖效率的無名英雄！它是知名開源圖床軟體 PicGo 的進階強化版，負責在背景默默攔截你貼上的圖片，瞬間丟到 GitHub 並華麗轉身變成高速的 CDN 網址。沒有它，就沒有「無感貼圖」的極致爽快體驗。

# GitHub - 倉庫與 API Key
為了讓文章筆記可以有跨平台存取的能力，並使用免費高速 CDN，要先在 GitHub 建立 `文章用倉庫`、 `圖片用倉庫` 與 `API金鑰`

- ## 建立 GitHub 文章倉庫
	- 登入 GitHub，右上角點擊個人頭像，點擊`Repository`後進入倉庫頁面，點擊右上角的 `New`。
	- **Repository name**：取一個好記的名字（以筆者來說使用 `Knowledge-Base`）。
	- **Public / Private**：如果要私密性質或是草稿，可選擇 **Private (非公開)**！Obsidian 支援私密倉庫同步。
	- **Add a README file**：建議勾選，讓倉庫初始化，產生預設的 `main` 或 `master` 分支。
	- 點擊 `Create repository` 完成建立。

- ## 建立 GitHub 圖床倉庫
	- 登入 GitHub，右上角點擊個人頭像，點擊`Repository`後進入倉庫頁面，點擊右上角的 `New`。
	- **Repository name**：取一個好記的名字（以筆者來說使用`blog-img`）。
	- **Public / Private**：務必選擇 **Public (公開)**！要使用的 `jsDelivr CDN` 無法讀取私有倉庫的圖片。
	- **Add a README file**：建議勾選，讓倉庫初始化，產生預設的 `main` 或 `master` 分支。
	- 點擊 `Create repository` 完成建立。
	  
- ## 取得 Personal Access Token (API Key)
	- 登入 GitHub，右上角點擊個人頭像，點擊`Settings`後進入設定頁面。
	- 點擊 GitHub 右上角大頭貼，進入 **Settings (設定)**。
	- 左側選單拉到最底，點擊 **Developer settings**。
	- 展開 **Personal access tokens**，選擇 **Tokens (classic)**。
	- 點擊右上方 **Generate new token (classic)**。
	- **Note**：隨便填個辨識用的名稱（例如：`PicList Upload Token`）。
	- **Expiration**：建議設定為 `No expiration` (無期限)，免得以後過期還要重新設定。
	- **Select scopes**：⚠️ 找到並勾選 **`repo`** (Full control of private repositories) 這一大項即可。
	- 滑到最底點擊 `Generate token`。
	- **立刻複製這串 `ghp_` 開頭的亂碼！** (離開這個頁面後就再也看不到了，請先把它貼在記事本備用)。

# PicList & CDN 圖床設定
- ## 新增 GitHub 圖床
  打開 PicList，進入 **「圖床 PicBed」 -> 「GitHub」 -> 「新增配置 Ａdd New Configuration」**，並填寫以下對應資料：
	- **設定倉庫名**：`你的GitHub帳號/你的倉庫名` (以 BRO 的為例：`itx4-blog/blog-img`)
	- **設定分支名**：`master` 或 `main` (請去 GitHub 倉庫看一下你的預設分支是哪一個)
	- **設定 Token**：貼上剛剛複製的 `ghp_` 開頭的密鑰。
	- **指定存儲路徑**：(選填) 如果希望圖片分類存放，可以填 `img/`，留空則會直接放在倉庫根目錄。
	  
- ## 設定 jsDelivr CDN (加速關鍵)
  為了讓圖片載入飛快，請在同一個設定頁面的 **「自訂網域名稱」** 欄位填入 CDN 網址：
	- **格式**：`https://cdn.jsdelivr.net/gh/你的帳號/倉庫名`
	- **以 BRO 為例**：`https://cdn.jsdelivr.net/gh/itx4-blog/blog-img`
	- _(註：PicList 上傳時會自動把你設定的網域和圖片檔名拼起來，變成完美的 CDN 連結！)_
  設定好後，點擊 **確定 Confirm**，並將其設為**預設圖床**。
   ![image.png|630](https://cdn.jsdelivr.net/gh/itx4-blog/blog-img@master/20260320142300470.png)

- ## 開啟本地 Server (讓 Obsidian 呼叫的關鍵)
	- 進入 PicList 的 **「設定 (Settings)」**。
	- 找到 **「伺服器設定 (Server)」**。
	- 確認 **「開啟 Server」** 為開啟狀態。
	- 記下監聽連接埠 (Port)，預設應為 `36677`。

# Obsidian 自動上傳設定 (Image Auto Upload Plugin)

這一步是為了達到「無感上傳」。設定好之後在 Obsidian 裡按下 `Ctrl + V` 貼上圖片時，外掛會自動攔截圖片 -> 傳給背景的 PicList -> 上傳到 GitHub -> 最後把超快的 CDN 網址貼回你的筆記中。

- ## 安裝核心外掛
	- 打開 Obsidian 的 **設定 (Settings)** -> **第三方外掛程式 (Community plugins)**。
	- 關閉安全模式 (如果還沒關的話)，點擊 **瀏覽 (Browse)**。
	- 搜尋 **`Image auto upload Plugin`** (作者通常是 renmu123)。
	- 點擊 **安裝 (Install)**，安裝完畢後記得點擊 **啟用 (Enable)**。
	  
- ## 設定外掛與 PicList 連線
  啟動外掛後，進入它的設定頁面，確認以下幾個關鍵參數：
	- **預設上傳器 (Default uploader)**：選擇 **`PicGo(app)`**。 _(💡 筆記備註：PicList 是基於 PicGo 開發的進階版，它們的底層 API 是一樣的，所以這裡選 PicGo(app) 就能完美連動 PicList。)_
	- **PicGo server API**：填入 **`http://127.0.0.1:36677/upload`**。 _(這個 `36677` 就是我們在第一階段確認過的 PicList 伺服器監聽 Port)_
	  ![image.png|454](https://cdn.jsdelivr.net/gh/itx4-blog/blog-img@master/20260320184029613.png)


# Obsidian Git 自動同步 (跨平台終極武器)

為了讓 Windows 和 Linux 兩邊的筆記能夠完美同步，我們將使用 Git 來做版本控制與備份。

- ## 初始化本地筆記庫 (Git Init)
  如果你已經建立過 Git 倉庫，這步可以跳過。如果是全新的筆記本，請照著做：
	1. 打開終端機（Windows 用 PowerShell / Linux 用 Terminal）。
	2. 用 `cd` 指令切換到你的 Obsidian 筆記本資料夾根目錄。
	3. 依序輸入以下指令建立連線（假設你已經在 GitHub 建好一個 private 的筆記倉庫）：
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/你的帳號/你的筆記倉庫.git
git push -u origin main
```

- ## 設定 .gitignore (跨平台不衝突的關鍵)
  因為 Windows 和 Linux 的螢幕解析度、視窗大小通常不同，我們**不希望**兩邊的「工作區設定（打開了哪些分頁）」互相覆蓋。 請在筆記庫的根目錄建立一個 `.gitignore` 檔案，並加入以下內容：
  ```plaintext
# 忽略 Obsidian 的工作區狀態（這樣 Win/Linux 的視窗配置才不會打架）
.obsidian/workspace.json
.obsidian/workspace
.obsidian/workspace-mobile.json

# 忽略作業系統產生的垃圾檔
.DS_Store
Thumbs.db
  ```

- ## 安裝與設定 Obsidian Git 外掛
	- 在 Obsidian 的「第三方外掛程式 Community plugins」搜尋 **`Git (作者為 Vinzent )`** 並安裝啟用。
	- 進入 Obsidian Git 的設定頁面，找到以下幾個「自動化」核心參數：
	- **Auto commit-and-sync interval**：設為 `15` (每 15 分鐘自動備份與同步，頻率看個人)。
	- **Auto pull interval**：設為 `15` (設定自動拉取的時間，建議跟備份時間一樣)。
	- **Pull on startup**：**強烈建議開啟！** 這樣你一打開 Obsidian，它就會自動去抓另一台電腦最新的筆記進度。
	- **Push on commit-and-sync**：**務必開啟！** 確保備份時有推送到遠端倉庫。
	- **Hide notifications for no changes**：**開啟** (沒寫字時別跳通知煩我)。

- ## 解決自動同步的「密碼阻礙」(Auth 問題) 
- 因為 Obsidian Git 是在「背景」默默執行指令的，如果它遇到需要輸入帳號密碼的情況，就會直接報錯卡住。為了讓跨平台雙向同步順暢：
- **Windows 系統**：只要你平常有用 Git，系統通常已經裝了 `Git Credential Manager`。第一次 push 登入過 GitHub 授權後，以後就會自動記住。
- **Linux 系統 (專屬 SSH 金鑰免密碼設定)**： 在 Linux 上，我們強烈建議設定 **SSH Key (安全金鑰)**。這就像是給電腦打造了一把專屬鑰匙，交給 GitHub 保管，以後背景同步時就完全不需要手動打密碼！請跟著以下步驟做： 
1. **產生鑰匙**： 打開終端機 (Terminal)，輸入以下指令 (信箱請換成你註冊 GitHub 的信箱)： ```bash ssh-keygen -t ed25519 -C "你的信箱@email.com" ``` *(💡 提示：輸入後遇到任何問題或停頓，直接**一路狂按 Enter 鍵到底**就好，不需要額外設定密碼！)* 
2. **讀取公鑰**： 接著輸入以下指令，終端機裡會印出一長串 `ssh-ed25519` 開頭的亂碼，這就是你的「公鑰」，請把它**反白並完整複製**起來： ```bash cat ~/.ssh/id_ed25519.pub ``` 
3. **把鑰匙交給 GitHub**： - 登入 GitHub 網頁，點擊右上角大頭貼 -> 選擇 **Settings**。 - 左側選單找到 **SSH and GPG keys**，點擊綠色按鈕 **New SSH key**。 - **Title** 隨便取一個認得出來的名字 (例如：`My Linux PC`)，**Key** 的大框框裡貼上剛才複製的那串公鑰，點擊 `Add SSH key`。 
4. **測試連線 (見證奇蹟)**： 回到終端機，輸入以下指令測試是否連線成功： ```bash ssh -T git@github.com ``` *(💡 提示：如果第一次連線跳出問你 `Are you sure you want to continue connecting (yes/no/[fingerprint])?`，請鍵入 `yes` 並按 Enter。只要看到 `Hi 你的帳號! You've successfully authenticated...` 就代表打通任督二脈啦！)* 
5. **更改 Git 倉庫連線網址 (最後一步)**： 回到終端機，用 `cd` 指令進到你的筆記資料夾內，把原本 `https://` 的連線方式換成專屬的 SSH 網址，輸入： 
```bash
   git remote set-url origin git@github.com:Account/Repo.git
```

大功告成！以後你在 Linux 上的同步就會像呼吸一樣自然，再也不會被密碼卡住了！