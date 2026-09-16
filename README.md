# Ultimate Android Container

一個以 WebView 為主的 Android container app。啟動時（`onCreate`）會建立一個全螢幕 WebView，並固定載入：

```
https://ultimatecardgame.zeabur.app
```

以下說明 `app/src/main/java/com/ultimate/cardgame/MainActivity.kt` 的功能。

## WebView 初始化

- **JavaScript**：啟用（`settings.javaScriptEnabled = true`），讓網頁可以執行 JS
- **DOM Storage**：啟用（`settings.domStorageEnabled = true`），讓網頁可以使用 `localStorage` 等儲存機制
- **WebViewClient**：設定 `WebViewClient()`，確保站內連結都在 WebView 內開啟，不會跳轉到外部瀏覽器
- **固定 URL**：載入 `HOME_URL`（定義在 `companion object`，目前為 `https://ultimatecardgame.zeabur.app`）

## 隱藏系統 Navigation Bar

使用 `WindowCompat.getInsetsController` 隱藏系統 navigation bar：

- 行為模式為 `BEHAVIOR_SHOW_TRANSIENT_BARS_BY_SWIPE`
- 平常隱藏，從螢幕底部上滑時會暫時顯示，幾秒後自動再隱藏
- 僅隱藏 navigation bar（`Type.navigationBars()`），status bar 維持顯示

## Back 鍵處理

透過 `onBackPressedDispatcher` 攔截返回鍵，流程如下：

1. 先透過 `evaluateJavascript` 詢問網頁：呼叫網頁定義的 `window.__onAndroidBack()`
2. 回傳 `true` → 表示網頁已自行處理（例如遊戲中彈出「離開遊戲？」確認視窗），Android 不做任何事
3. 回傳 `false` 或函式不存在 → WebView 有瀏覽歷史就 `goBack()`，否則 `finish()` 關閉 app

## 網頁端配合方式

網站 JS 需定義 `window.__onAndroidBack`，回傳 `true` 表示「已處理，Android 別動」：

```javascript
window.__onAndroidBack = () => {
  // 例如：遊戲進行中，彈出「確定離開？」確認視窗
  if (gameInProgress && !confirmLeaveDialogShown) {
    showLeaveDialog();
    return true; // consumed, Android does nothing
  }
  return false; // not handled, Android does goBack() or finish()
};
```

## 前置條件

- `AndroidManifest.xml` 需包含 INTERNET permission，WebView 才能連網：

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## Build 方式

### 環境需求

- JDK 17 或以上（Android Studio 內建的 JDK 即可）
- Android SDK（`compileSdk 37`，會由 `sdk.dir` 指向的 SDK 自動解析）
- Gradle 不需自行安裝，使用專案內的 Gradle Wrapper（Gradle 9.6.0），首次執行會自動下載

### Command Line（Windows PowerShell）

在專案根目錄執行：

```powershell
# Debug APK
.\gradlew.bat assembleDebug

# Release APK
.\gradlew.bat assembleRelease

# 直接安裝到已連接的裝置或模擬器
.\gradlew.bat installDebug
```

APK 輸出位置：

- Debug：`app/build/outputs/apk/debug/app-debug.apk`
- Release：`app/build/outputs/apk/release/app-release.apk`（未設定 signing config，上架前需自行簽章）

### Android Studio

1. 用 Android Studio 開啟專案根目錄
2. 等待 Gradle Sync 完成
3. 選擇模擬器或實機，點擊 Run 即可
