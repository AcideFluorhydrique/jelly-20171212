# Jelly (2017) on a modern toolchain

把 2017 年 12 月的 [LineageOS Jelly](https://github.com/LineageOS/android_packages_apps_Jelly) 瀏覽器，用現代的 Gradle 工具鏈在 GitHub Actions 上編譯成可安裝的 APK。

這是一個「把舊程式碼救活」的實驗，不是 Jelly 的維護分支。想要日常使用的 Jelly，請直接用上游的現行版本。

## 這是什麼

Jelly 是 LineageOS 內建的輕量瀏覽器。它本身不含瀏覽器引擎，只是系統 WebView 外面的一層殼，所以這份 2017 年的程式碼在新手機上跑的仍是最新的 Chromium 內核。

本 repo 的源碼快照：

- 版本 `20171212`（`versionCode 2`）
- 包名是 `at.xtools.lineageos.jelly` 而非官方的 `org.lineageos.jelly`。匯入時就是這個值，推測源自某個第三方重新打包的獨立編譯版本，可與系統內建的 Jelly 並存安裝
- 保留了原本樹內編譯用的 `Android.mk` 與 `java_lineage/`，但沒有用到；Gradle 編譯走的是 `java_studio/`

## 下載 APK

每次 push 到 `master` 都會觸發編譯。到 [Actions](../../actions) 頁面點進最新一次成功的 run，在 Artifacts 下載 `jelly-browser-debug`（保留 14 天）。

這是 debug 簽名的 APK，僅供測試。

## 自行編譯

需要 JDK 17 與 Android SDK（platform 34）。

```bash
./gradlew assembleDebug
```

產物在 `app/build/outputs/apk/debug/`。

## 相對於原始快照改了什麼

| 項目 | 原本 | 現在 |
|---|---|---|
| Gradle | 3.3 | 8.4 |
| Android Gradle Plugin | 2.3.3 | 8.1.4 |
| JDK | 8 + Jack | 17 |
| compileSdk / targetSdk | 25 | 34 |
| 依賴庫 | `com.android.support` 25.3.1 | androidx / Material Components |
| 套件倉庫 | jcenter | google + mavenCentral |

程式碼層面的修改：

- `android.support.*` 全面遷移到 androidx（Java import、layout XML、Manifest）
- R class 的 `namespace` 設為 `org.lineageos.jelly`，與 `applicationId` 分離
- ContentProvider 的 authority 改為 `at.xtools.lineageos.jelly.*`，避免與內建 Jelly 衝突
- 移除 SDK 33 已刪除的 AppCache API
- 動態註冊的 BroadcastReceiver 加上 `RECEIVER_NOT_EXPORTED`（Android 14+ 必需）
- `PendingIntent` 加上可變性旗標（Android 12+ 必需）
- Manifest 補上 `android:exported`
- 設定 `android.nonFinalResIds=false`，讓 `switch (R.id.xxx)` 繼續可用

## 狀態

已在 Redmi Note 15（Android 16）上確認可以安裝、啟動、開啟網頁。

除此之外的功能都沒有驗證過。以下是從程式碼判斷很可能壞掉的地方：

- **下載檔案**：依賴 `WRITE_EXTERNAL_STORAGE`，在 targetSdk 34 下已無效
- **新增桌面捷徑**：使用 Android 8 之後已失效的 `INSTALL_SHORTCUT` 廣播
- **設定頁**：使用已棄用的框架 `PreferenceFragment`

## 授權

源碼檔頭標示為 Apache License 2.0，Copyright (C) 2017 The LineageOS Project。
