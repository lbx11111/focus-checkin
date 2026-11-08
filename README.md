# 专注打卡 · 安卓 APK 打包说明

这个包里是一个完整的安卓工程（WebView 套壳），装好后**完全离线、不依赖任何网站**，
不会再出现「网址失效」的问题。

## 目录结构

```
android/                                   安卓工程（用这个）
  settings.gradle
  build.gradle
  gradle.properties
  app/build.gradle
  app/src/main/AndroidManifest.xml
  app/src/main/java/com/focus/checkin/MainActivity.java
  app/src/main/res/values/styles.xml
  app/src/main/res/values/strings.xml
  app/src/main/res/drawable/icon.png
  app/src/main/assets/www/index.html       应用本体（已内置中文修正 + 原生保存桥接）
.github/workflows/build-apk.yml            云端自动编译配置
focus-checkin-10min.html                   同一份页面，可直接用浏览器打开试用
```

## 路线一：GitHub 云端编译（不需要装任何东西，约 10 分钟）

1. 注册/登录 GitHub，新建一个 repository（Private 也可以）
2. 上传本包**全部内容**。`.github` 是隐藏文件夹，网页上传可能看不到，两个办法：
   - 办法 A：用 GitHub Desktop 客户端，直接拖整个文件夹进来（最省事）
   - 办法 B：在网页上依次点 Add file → Create new file，文件名填
     `.github/workflows/build-apk.yml`，把本包里同名文件的内容粘进去，提交
3. 打开仓库顶部 **Actions** 标签 → 左侧选 `build-apk` → 右侧 **Run workflow**
4. 等 3-5 分钟，进这次运行记录，页面底部 **Artifacts** 下载 `focus-checkin-apk.zip`
5. 解压得到 `app-debug.apk`，传到手机点击安装
   （手机需允许「安装未知来源应用」，一般在 设置 → 应用 → 特殊权限 里）

## 路线二：Android Studio（约 15 分钟）

1. Android Studio → Open → 选中 `android` 文件夹
2. 等 Gradle Sync 完成（首次要下载依赖，几百 MB，需网络）
3. 菜单 Build → Build App Bundle(s) / APK(s) → Build APK(s)
4. 产物在 `android/app/build/outputs/apk/debug/app-debug.apk`

## 这个版本修了什么

- **中文编码**：已按原始 UTF-8 字节写入，`专` = `e4 b8 93`，不会再乱码
- **导出 CSV / JSON 现在能用**：原本 WebView 不接管 blob 下载，导出会静默失败。
  这次加了原生桥接（`window.AndroidSave`），点导出会直接写入手机存储并弹出 Toast 提示路径，
  一般位于：`/sdcard/Android/data/com.focus.checkin/files/Documents/`
- 状态栏/导航栏颜色与应用一致，无白边
- 返回键优先在 WebView 内回退

## 已知限制（诚实说明）

- 这是 WebView 套壳，不是原生 App；内核是手机自带的 WebView，体验接近浏览器
- **我无法在这里编译验证**，AGP 8.5.2 / Gradle 8.7 是我按当前版本配的，
  如果构建报错，通常是这两者版本不匹配，按报错提示改 `android/build.gradle` 里的版本号即可
- 产物是 **debug 签名**的 APK，自己装没问题；要发给别人或上架，需要自己生成 keystore 并配置签名
- 数据存在 App 私有目录的 localStorage 里。**卸载 App 会清空记录**，建议定期导出 JSON 备份

## 数据迁移

新装的 App 里数据是空的，和之前网页版不互通。
先在网页版「导出 JSON」，再在 App 里「导入 JSON」即可搬过来。

## 改代码后重新打包

只改 `android/app/src/main/assets/www/index.html`，然后重新编译即可。
根目录那份 `focus-checkin-10min.html` 是同一内容的副本，方便先用浏览器预览，改的时候两份一起改。