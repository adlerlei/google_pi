# MQ 直立式廣告機內容管理系統

## 專案簡介

本專案旨在為 MQ 直立式廣告機提供一個後台內容管理系統。使用者可以透過網頁介面 (`admin.html`) 上傳、管理並指派圖片和影片素材到廣告機前端 (`index.html`) 的不同顯示區塊。系統支援多個輪播區塊的內容更新，以及頁首和頁尾的單一或輪播內容展示，並具備 WebSocket 即時更新功能，確保前端顯示內容在後台操作後能立即同步。

## 目前已實現功能

* **後台管理介面 (`admin.html`)**:
    * **媒體素材管理**:
        * 上傳圖片 (jpg, jpeg, png, gif) 和影片 (mp4, mov, avi) 作為獨立的媒體素材。
        * 檔案上傳時顯示進度條。
        * 素材庫列表，顯示已上傳的素材及其狀態（是否被使用）。
        * 刪除媒體素材（同時會從伺服器刪除實體檔案，並處理相關引用）。
    * **輪播圖片組管理**:
        * 建立輪播圖片組，並為其命名。
        * 編輯輪播圖片組：
            * 從素材庫中選擇圖片加入到群組。
            * 從群組中移除圖片。
            * 在群組內拖曳圖片進行排序。
            * 儲存對輪播組的修改。
        * 刪除輪播圖片組（同時會移除所有對此群組的區塊指派）。
    * **區塊內容指派**:
        * **操作類型選擇**:
            * **上傳圖片/影片並直接指派**: 允許使用者上傳新的圖片或影片，並立即將其指派給選定的前端顯示區塊（頁首、頁尾、任一中間輪播區塊）。對同一區塊的多次直接指派會形成累加效果，使該區塊輪播這些內容。
            * **指派輪播組到輪播區塊**: 允許使用者選擇一個已建立的輪播圖片組，將其指派給四個中間輪播區塊中的一個，並可設定圖片的起始偏移量。此操作會覆蓋目標輪播區塊先前所有的內容。
        * 清晰的表單欄位，根據選擇的操作類型動態顯示。
    * **列表顯示**:
        * 「媒體庫與區塊內容指派」列表：清晰展示「區塊內容指派」記錄（包括來源是單一素材還是輪播組，以及目標區塊）和「未被使用的純素材」。
        * 「現有輪播圖片組」列表：顯示已建立的輪播組名稱和包含的圖片數量。

* **後端 (`app.py` - Flask)**:
    * 提供 API 接口 (`/api/media`) 供前端獲取格式化後的媒體數據。
    * 處理檔案上傳、儲存 (唯一檔名) 及刪除。
    * 使用 `media.json` 作為數據庫，儲存媒體素材、輪播組定義和區塊內容指派等資訊。
    * 實現了清晰的數據模型，區分媒體素材、輪播組和區塊指派。
    * 實現了內容指派的累加（針對直接上傳到區塊）和覆蓋（針對輪播組指派）邏輯。
    * **WebSocket 即時更新**: 使用 Flask-SocketIO，在後台數據發生變更時，向所有連接的前端客戶端廣播 `media_updated` 事件，觸發前端自動刷新。

* **前端顯示頁面 (`index.html` & `static/js/animation.js`)**:
    * 根據從後端 API 獲取的數據，動態渲染頁首、頁尾和四個中間輪播區塊的內容。
    * 支持中間輪播區塊顯示來自輪播組的圖片，並根據偏移量正確排序和輪播。
    * 支持頁首、頁尾和中間輪播區塊顯示由多個直接指派的圖片/影片組成的輪播內容。
    * 監聽 WebSocket 事件，實現內容的即時自動更新。

## 技術棧

* **後端**: Python, Flask, Flask-SocketIO, eventlet (或 gevent)
* **前端**: HTML, CSS (Bulma 框架), JavaScript
* **數據儲存**: JSON 檔案 (`media.json`)

## 專案結構 (主要檔案)
google_pi
├── README.md
├── app.py
├── media.json
├── requirements.txt
├── static
│   ├── css
│   │   ├── bulma.css
│   │   └── style.css
│   ├── js
│   │   └── admin.js
│   └── uploads
│       ├── 1_a9da9d73.jpg
│       ├── 2_dd425e74.jpg
│       ├── BMW_069087cf.png
│       ├── BMW_ef6e87e9.jpg
│       ├── hollywood_172e21b1.jpg
│       ├── jpg_5e2eba81
│       ├── jpg_766cfc70
│       ├── jpg_a7b6ee67
│       └── jpg_d0f69405
└── templates
    └── admin.html


## 如何運行 (基本步驟)

1.  確保已安裝 Python 和 pip。
2.  安裝所需的 Python 套件：
    ```bash
    pip install Flask Flask-SocketIO Flask-CORS eventlet werkzeug
    ```
3.  在專案根目錄下運行 `app.py`：
    ```bash
    python app.py
    ```
4.  後台管理介面通常可以通過 `http://<伺服器IP>:5000/admin` 訪問。
5.  前端顯示頁面通常可以通過 `http://<伺服器IP>:5000/` (或其他指定路由) 訪問。

## 未來可能方向

* 實現「全局設定」區塊，用於統一管理各區塊內容的輪播間隔或播放時長。
* 實現「編輯指派」功能，允許修改現有的區塊內容指派（例如更改偏移量、更換素材或群組）。
* 進一步優化 `admin.html` 的使用者體驗和介面設計。
* 細化前端 `static/js/animation.js` 的輪播邏輯，例如增加過渡效果、播放控制等。
* 廣告輪播購買客戶管理。
* 增加使用者認證和權限管理。

---

**開發者註記：**

* 本專案在開發過程中，針對數據模型和核心指派邏輯進行了多次迭代和重構，以實現更靈活和直觀的內容管理。
* 前端 JavaScript 檔案的確切名稱 (例如 `animation.js`) 和前端顯示頁面模板的名稱 (例如 `index.html`) 需根據實際專案情況確認。
* 針對特定瀏覽器（如 Arc Browser）在處理 `window.alert()` 和 `window.confirm()` 時可能存在的兼容性問題，建議在執行關鍵操作時使用 Chrome 等主流瀏覽器，或檢查瀏覽器的彈出視窗權限設定。