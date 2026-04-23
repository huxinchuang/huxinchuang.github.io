# 济南泉水探索之旅 - 配置指南

## 前置准备

需要注册两个平台账号（均免费）：

| 平台 | 用途 | 注册地址 |
|------|------|----------|
| 高德地图开放平台 | 地图显示、地址搜索 | https://lbs.amap.com/ |
| Google Firebase | 多人实时数据同步 | https://console.firebase.google.com/ |

---

## 一、高德地图 API 配置

### 1. 注册并创建应用

1. 登录 [高德地图开放平台](https://lbs.amap.com/)
2. 点击右上角「控制台」→「应用管理」→「我的应用」
3. 点击「创建新应用」
   - **应用名称**: 任意填写（如"济南旅游规划"）
   - **应用类型**: 选择「Web端（JS API）」
4. 创建成功后，点击「添加 Key」:
   - **Key 名称**: 任意填写
   - **服务平台**: 选择「Web端（JS API）」
   - **域名白名单**: 开发阶段留空，上线后改为你的域名
5. 复制生成的 **Key**

### 2. 填入项目

打开 `jinan-travel.html`，找到第 12 行，将 `你的高德Key` 替换为粘贴你的 Key：

```html
<script src="https://webapi.amap.com/maps?v=2.0&key=这里粘贴你的Key"></script>
```

### 3. 验证地图

直接打开 `jinan-travel.html`，如果地图正常显示济南区域，则配置成功。

---

## 二、Firebase 配置

### 1. 创建 Firebase 项目

1. 访问 [Firebase Console](https://console.firebase.google.com/)
2. 点击「创建项目」
   - 输入项目名称（如 `jinan-travel`）
   - Google Analytics 可选择「不启用」（免费计划不需要）
3. 等待项目创建完成（约10秒）

### 2. 注册 Web 应用

1. 在项目概览页，点击 **Web 图标**（`</>`）注册 Web 应用
2. 输入应用昵称（如 `济南旅游规划`）
3. **不要勾选** "Firebase Hosting"
4. 点击「注册应用」
5. 在弹出的配置信息中，复制完整的 `firebaseConfig` 对象：

```javascript
const firebaseConfig = {
  apiKey: "AIzaSyxxxxxxxxxxxxxxxxxxx",
  authDomain: "jinan-travel-xxxxx.firebaseapp.com",
  projectId: "jinan-travel-xxxxx",
  storageBucket: "jinan-travel-xxxxx.appspot.com",
  messagingSenderId: "123456789012",
  appId: "1:123456789012:web:abc123def456"
};
```

### 3. 填入项目

打开 `jinan-travel.html`，搜索 `FIREBASE_CONFIG`（约第555行），将上述配置替换进去：

```javascript
const FIREBASE_CONFIG = {
  apiKey: "AIzaSyxxxxxxxxxxxxxxxxxxx",
  authDomain: "jinan-travel-xxxxx.firebaseapp.com",
  projectId: "jinan-travel-xxxxx",
  storageBucket: "jinan-travel-xxxxx.appspot.com",
  messagingSenderId: "123456789012",
  appId: "1:123456789012:web:abc123def456"
};
```

### 4. 创建 Firestore 数据库

1. 在 Firebase 左侧菜单点击 **Firestore Database**
2. 点击「创建数据库」
3. **选择位置**: 选 `asia-east1`（台湾，离中国大陆最近）
4. **安全规则**: 选择「测试模式」（后续可收紧）
5. 点击「启用」

### 5. 设置安全规则

创建数据库后，点击「规则」选项卡，粘贴以下内容：

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if true;
    }
  }
}
```

点击「发布」。

> **⚠️ 安全说明**：上述规则允许任何人读写。因为是**小团体内部使用**，且没有敏感数据，暂时够用。如需更严格的控制，后续可加入身份验证。

### 6. 验证 Firebase

1. 打开 `jinan-travel.html`
2. 输入昵称进入
3. 添加一个标记点
4. 打开浏览器开发者工具（F12）→ Console
5. 看到 `Firebase 已连接` 和 `已同步` 说明配置成功
6. 可以在 Firebase Console → Firestore → 数据 中看到新增的数据

---

## 三、部署上线（可选）

### 方式一：GitHub Pages（推荐）

1. 将整个项目推送到 GitHub 仓库
2. 仓库 → Settings → Pages
3. Source 选 `main`，文件夹选 `/ (root)`
4. 访问 `https://你的用户名.github.io/jinan-travel.html`

### 方式二：Firebase Hosting

```bash
npm install -g firebase-tools
firebase login
firebase init hosting
  # 选择你的项目
  # 构建目录填 `.`
  # 是否单页应用填 `n`
firebase deploy
```

---

## 四、常见问题

### 地图不显示
- 检查 Key 是否正确粘贴
- 检查高德地图控制台 → 应用管理 → 是否选择「Web端（JS API）」
- 本地打开时（file://），域名白名单需要留空

### Firebase 同步失败
- 检查 `FIREBASE_CONFIG` 的 6 个字段是否都正确粘贴
- 检查 Firestore 数据库是否已创建
- 检查安全规则是否已发布
- 如果在中国大陆无法访问 Google，需要科学上网

### 在线用户数不更新
- 检查 Firestore 中是否已有 `jinan_travel_presence` 集合
- 检查 Console 是否有报错

### 数据丢失
- Firebase 免费计划有 1GB 存储，通常够用
- 定期使用「导出数据」按钮备份
- 多人同时删除可能产生冲突，建议分工协作

---

## 五、配置检查清单

- [ ] 高德地图 Key 已替换（第 12 行）
- [ ] Firebase 配置已替换（FIREBASE_CONFIG 的6个字段）
- [ ] Firestore 数据库已创建
- [ ] Firestore 安全规则已发布
- [ ] 多人同时打开页面测试同步正常
- [ ] 在线用户数显示正常
