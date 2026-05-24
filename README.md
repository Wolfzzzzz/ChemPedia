# 🧪 化学大全 · ChemPedia

> 全栈化学工具 — 元素周期表、方程式配平、化学反应大全、化合物数据库，iOS 液态玻璃风格 UI。

![License](https://img.shields.io/badge/license-MIT-blue)
![React](https://img.shields.io/badge/React-18-61dafb)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178c6)

---

## ✨ 功能

| 模块 | 说明 |
|------|------|
| 🔬 **元素周期表** | 完整 118 个元素，镧系锕系正确排列，点击查看详情 |
| ⚖️ **方程式配平** | 输入化学式自动配平，支持点击化合物/元素输入 |
| 🧪 **化学反应大全** | 20+ 完整反应，含现象描述、危险提示、反应条件 |
| 🔍 **实时搜索** | 输入即出结果，支持普通数字替代角标（H2O → H₂O） |
| 🏷️ **化合物数据库** | 有机/无机分类，完整化学式显示 |
| ⚠️ **危险标红** | 高危反应用红色警示 + 脉动动画 |

---

## 🎨 设计

- iOS 26 液态玻璃效果（毛玻璃 + 半透明）
- 深色主题 + 灵动岛风格圆角胶囊按钮
- 全动态效果（悬停、渐变、脉动、滑入动画）

---

## 📥 下载

| 版本 | 说明 |
|------|------|
| [化学大全 Setup 1.0.0.exe](https://github.com/Wolfzzzzz/chemistry-app/release) | Windows 安装版 |
| [化学大全_Portable.zip](https://github.com/Wolfzzzzz/chemistry-app/zip) | 免安装便携版 |
| [化学大全_exe文件（如果丢失）](https://github.com/Wolfzzzzz/chemistry-app/release/bug) | Windows 安装版（补充包） |

---

## 🚀 本地运行

```bash
# 克隆仓库
git clone https://github.com/Wolfzzzzz/chemistry-app.git
cd chemistry-app

# 安装依赖
npm install

# 启动开发服务器
npm run dev:client

# 访问
# http://localhost:5177
```

---

## 🛠 技术栈

- **前端**：React 18 + TypeScript + Vite
- **样式**：Tailwind CSS + 自定义 CSS Variables
- **图标**：Lucide React
- **构建**：Vite

---

## 📂 项目结构

```
chemistry-app/
├── src/
│   ├── App.tsx              # 主组件（周期表、配平、反应大全）
│   ├── data/
│   │   ├── periodicTable.ts # 118 元素数据
│   │   └── chemistryDB.ts   # 化合物 & 反应数据库
│   ├── utils/
│   │   └── balancer.ts      # 化学方程式配平引擎
│   └── index.css            # iOS 液态玻璃样式
├── dist/                    # 构建输出
├── package.json
└── README.md
```

---

## 📝 许可证

MIT © 2026
