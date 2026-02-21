# MeetPoint - 智能会面点推荐

MeetPoint 是一个基于 Vue 3 和高德地图 API 的会面点推荐工具，旨在帮助用户找到多方参与者的公平中点，并推荐附近的聚会场所。

## 功能特性

- **多方参与者**：支持 2-10 人输入位置，通过搜索添加。
- **智能中点计算**：计算所有参与者的几何中心。
- **场景筛选**：支持多种场景类型（咖啡馆、餐厅、KTV等）。
- **周边推荐**：基于中点推荐附近的优质场所。
- **分享功能**：一键生成会面点信息卡片。

## 快速开始

### 1. 安装依赖

```bash
npm install
```

### 2. 配置高德地图 API Key

在项目根目录下找到 `.env` 文件，填入您的高德地图 API Key（Web端 JS API Key）：

```bash
VITE_AMAP_KEY=your_amap_api_key_here
```

如果没有 Key，请前往 [高德开放平台](https://console.amap.com/dev/key/app) 申请。

**注意**：申请 Key 时请勾选 Web端(JS API)。

### 3. 启动开发服务器

```bash
npm run dev
```

### 4. 构建生产版本

```bash
npm run build
```

## 技术栈

- Vue 3
- Tailwind CSS
- AMap JSAPI (高德地图)
- Vite
