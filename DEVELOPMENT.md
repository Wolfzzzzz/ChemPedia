# 🧪 化学大全 · 开发文档

## 目录

- [项目概述](#项目概述)
- [环境要求](#环境要求)
- [快速开始](#快速开始)
- [项目结构](#项目结构)
- [架构设计](#架构设计)
- [核心模块](#核心模块)
- [数据流](#数据流)
- [UI 设计规范](#ui-设计规范)
- [构建与部署](#构建与部署)
- [开发约定](#开发约定)
- [常见问题](#常见问题)

---

## 项目概述

化学大全（ChemPedia）是一个全栈化学工具 Web 应用，涵盖元素周期表、化学方程式配平、化学反应大全、化合物数据库等功能。采用 React + TypeScript + Vite 构建，UI 遵循 iOS 液态玻璃设计语言。

**技术选型理由：**

| 技术 | 理由 |
|------|------|
| React 18 | 生态成熟，组件化开发 |
| TypeScript | 化学数据类型复杂，必须类型安全 |
| Vite | 开发秒启，HMR 极快 |
| Tailwind CSS | 原子化样式，配合 CSS Variables 实现主题 |
| 无框架（纯 React） | 轻量，无额外抽象层 |

---

## 环境要求

- **Node.js** ≥ 18
- **npm** ≥ 9
- **Windows / macOS / Linux**

---

## 快速开始

```bash
# 1. 克隆仓库
git clone https://github.com/你的用户名/chemistry-app.git
cd chemistry-app

# 2. 安装依赖
npm install

# 3. 启动前端开发服务器
npm run dev:client
# → http://localhost:5177

# 4. （可选）启动后端
npm run dev:server
# → http://localhost:3000
```

> **提示**：仅前端即可使用全部化学功能。后端为可选扩展（CodeBuddy SDK 集成）。

---

## 项目结构

```
chemistry-app/
├── src/
│   ├── App.tsx                    # 主应用组件（约 1500 行）
│   │   ├── PeriodicTableModal     # 元素周期表弹窗
│   │   ├── BalanceModal           # 方程式配平弹窗
│   │   ├── ReactionModal          # 反应详情弹窗
│   │   └── CompoundModal          # 化合物详情弹窗
│   │
│   ├── data/
│   │   ├── periodicTable.ts       # 118 元素完整数据
│   │   └── chemistryDB.ts         # 化合物 & 反应数据库
│   │       ├── inorganicCompounds
│   │       ├── organicCompounds
│   │       ├── reactions (20+)
│   │       ├── searchCompounds()
│   │       └── searchReactions()
│   │
│   ├── utils/
│   │   └── balancer.ts            # 化学方程式配平引擎
│   │
│   ├── main.tsx                   # React 入口
│   ├── index.css                  # iOS 液态玻璃全局样式
│   └── vite-env.d.ts
│
├── server/                        # 可选后端（CodeBuddy SDK）
│   ├── index.ts
│   └── db.ts
│
├── dist/                          # 构建输出
├── release/                       # 发布包
├── package.json
├── vite.config.ts
├── README.md
└── DEVELOPMENT.md
```

---

## 架构设计

### 整体架构

```
┌─────────────────────────────────────────┐
│                  App.tsx                  │
│  ┌─────────────────────────────────┐    │
│  │         View: 'home'            │    │
│  │  ┌──────────┐  ┌────────────┐   │    │
│  │  │ 周期表    │  │ 化合物列表  │   │    │
│  │  │ 弹窗      │  │ + 反应列表  │   │    │
│  │  └──────────┘  └────────────┘   │    │
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │         View: 'balance'         │    │
│  │  BalanceModal（独立配平界面）    │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
         │                    │
         ▼                    ▼
┌─────────────┐    ┌──────────────────┐
│ periodicTable│    │   chemistryDB    │
│  · 118 元素  │    │  · 化合物数据    │
│  · 分类查找  │    │  · 反应数据      │
└─────────────┘    │  · 搜索函数      │
                   └──────────────────┘
                            │
                            ▼
                   ┌──────────────────┐
                   │    balancer.ts    │
                   │  · 元素计数       │
                   │  · 矩阵消元       │
                   │  · 系数求解       │
                   └──────────────────┘
```

### 状态管理

应用使用 React `useState` + `useMemo` 管理状态，无需第三方库：

```typescript
// 核心状态
const [view, setView] = useState<'home' | 'balance' | 'periodic'>('home');
const [activeTab, setActiveTab] = useState<'compounds' | 'reactions'>('compounds');
const [searchQuery, setSearchQuery] = useState('');
const [activeMainCategory, setActiveMainCategory] = useState<string | null>(null);

// 派生状态
const filteredCompounds = useMemo(() => { ... }, [searchQuery, activeMainCategory]);
const filteredReactions = useMemo(() => { ... }, [searchQuery]);
```

---

## 核心模块

### 1. 元素周期表 (`periodicTable.ts`)

每个元素的数据结构：

```typescript
interface PeriodicElement {
  atomicNumber: number;
  symbol: string;
  name: string;
  nameCN: string;
  group: number;        // 族（1-18）
  period: number;       // 周期（1-7）
  category: 'alkali-metal' | 'alkaline-earth' | 'transition-metal'
           | 'post-transition' | 'metalloid' | 'nonmetal'
           | 'halogen' | 'noble-gas' | 'lanthanide' | 'actinide';
  atomicMass: number;
  electronegativity: number | null;
  electronConfiguration: string;
  phase: 'solid' | 'liquid' | 'gas';
  // ...
}
```

**镧系/锕系定位算法：**

```typescript
// 镧系 (58-71) → 第 8 行，列 4-17
if (el.category === 'lanthanide' && el.atomicNumber >= 58 && el.atomicNumber <= 71) {
  gridCol = 4 + (el.atomicNumber - 58);
  gridRow = 8;
}
// 锕系 (90-103) → 第 9 行，列 4-17
if (el.category === 'actinide' && el.atomicNumber >= 90 && el.atomicNumber <= 103) {
  gridCol = 4 + (el.atomicNumber - 90);
  gridRow = 9;
}
```

### 2. 配平引擎 (`balancer.ts`)

**算法流程：**

```
输入: "H2 + O2 = H2O"
  ↓
1. 解析反应物和产物 → ['H2', 'O2'], ['H2O']
  ↓
2. 提取所有元素 → { H: 2, O: 2 } = { H: 2, O: 1 }
  ↓
3. 构建线性方程组矩阵
  ↓
4. 高斯消元求解 → 系数 [2, 1, 2]
  ↓
输出: "2H₂ + O₂ → 2H₂O"
```

**关键导出：**

```typescript
export interface BalancedResult {
  equation: string;      // 格式化方程式
  coefficients: number[];
  reactants: string[];
  products: string[];
}

export function balanceEquation(
  reactants: string[],
  products: string[]
): BalancedResult;
```

### 3. 化学数据库 (`chemistryDB.ts`)

**数据结构：**

```typescript
// 化合物
interface ChemicalCompound {
  id: string;
  name: string;
  nameEN: string;
  formula: string;       // 带角标：H₂O
  formulaRaw: string;     // 原始格式：H2O
  category: 'inorganic' | 'organic';
  subcategory: string;
  molarMass: string;
}

// 化学反应
interface ChemicalReaction {
  id: string;
  name: string;
  type: string;              // 反应类型
  reactants: string[];
  products: string[];
  equationBalanced: string;
  conditions: string;        // 反应条件
  catalyst: string;          // 催化剂
  phenomenon: string;        // 现象描述
  dangerLevel: 'safe' | 'warning' | 'danger' | 'extreme';
  dangerNote: string;        // 危险提示
  energyChange: string;      // 能量变化
  reactionRate: string;      // 反应速率
  applications: string[];    // 实际应用
}
```

**搜索支持：**

```typescript
// 普通数字自动转换为角标
function normalizeSearch(str: string): string {
  return str.replace(/(\d+)/g, (_, d) =>
    d.split('').map(c => '₀₁₂₃₄₅₆₇₈₉'[parseInt(c)]).join('')
  );
}
```

### 4. App.tsx 组件树

```
App
├── Header（导航栏 + 搜索框）
├── 首页 (view === 'home')
│   ├── 分类浏览（有机/无机）
│   ├── 快速工具
│   │   ├── 元素周期表 按钮
│   │   ├── 化学方程式配平 按钮
│   │   └── 化学反应大全 按钮
│   ├── 选项卡切换（化合物 | 反应）
│   ├── 化合物列表（filteredCompounds）
│   └── 反应列表（filteredReactions）
│
├── 配平页 (view === 'balance')
│   └── BalanceModal
│
├── 周期表弹窗 (showPeriodicTable)
│   └── PeriodicTableModal
│
├── 反应弹窗 (selectedReaction)
│   └── ReactionModal
│
└── 化合物弹窗 (selectedCompound)
    └── CompoundModal
```

---

## 数据流

```
用户输入搜索词 "H2O"
    │
    ▼
setSearchQuery("H2O")
    │
    ▼
useMemo → filteredCompounds / filteredReactions
    │
    ├─ searchCompounds("H2O")
    │   └─ normalizeSearch("H2O") → "H₂O"
    │   └─ 匹配 formula / name / nameEN
    │
    └─ searchReactions("H2O")
        └─ 匹配 name / reactants / products
    │
    ▼
React 重渲染 → 显示匹配结果
```

---

## UI 设计规范

### CSS 变量（`index.css`）

```css
:root {
  --bg-primary: #000000;
  --bg-glass: rgba(255, 255, 255, 0.05);
  --bg-glass-hover: rgba(255, 255, 255, 0.08);
  --text-primary: rgba(255, 255, 255, 0.92);
  --text-secondary: rgba(255, 255, 255, 0.6);
  --text-tertiary: rgba(255, 255, 255, 0.38);
  --accent: #0a84ff;
  --danger: #ff3b30;
  --success: #30d158;
  --warning: #ffd60a;
  --blur: saturate(180%) blur(20px);
}
```

### 动画

| 动画名 | 用途 |
|--------|------|
| `fadeIn` | 弹窗渐入 |
| `slideUp` | 弹窗上滑 |
| `danger-pulse` | 高危反应呼吸 |
| `hover-lift` | 卡片悬停上浮 |

### 设计原则

- 深色主题优先，禁止白色背景
- 所有卡片使用 `var(--bg-glass)` + `backdrop-filter: var(--blur)`
- 圆角统一使用 `12px` / `16px` / `20px` 三级
- 高危险内容用 `var(--danger)` 色 + 脉冲动画

---

## 构建与部署

### 开发

```bash
npm run dev:client     # 前端 http://localhost:5177
npm run dev:server     # 后端 http://localhost:3000
npm run dev            # 前后端同时启动
```

### 生产构建

```bash
npm run build          # TypeScript 编译 + Vite 构建
# 输出到 dist/
```

### 预览

```bash
npm run preview        # Vite 预览服务器
npm run deploy         # 静态服务器 http://localhost:8080
```

### Vite 配置

```typescript
// vite.config.ts
export default defineConfig({
  base: './',           // 相对路径，支持 file:// 协议
  plugins: [react()],
  server: {
    host: '0.0.0.0',
    port: 5173,
  },
});
```

---

## 开发约定

### 代码风格

- **TypeScript strict mode**，避免 `any`
- 组件文件使用 `PascalCase`，工具函数使用 `camelCase`
- 化学式渲染统一使用 `formatFormula()` 函数
- 内联样式优先于 className（组件 self-contained）

### 化学数据

- 所有化学式内部存储使用**原始格式**（如 `H2O`），显示时转换为角标
- 新增反应数据必须包含完整的 `dangerLevel` 和 `phenomenon`
- 危险等级：`safe` → `warning` → `danger` → `extreme`

### Git 提交规范

```
feat: 新增元素周期表弹窗
fix: 修复镧系元素定位错误
style: 美化反应卡片悬停效果
refactor: 重构配平引擎矩阵求解
docs: 更新开发文档
```

---

## 常见问题

### Q: 搜索不显示结果？

确保 Vite 配置中的 `base: './'` 已设置。检查浏览器 DevTools Console 是否有 JavaScript 错误。

### Q: 元素周期表镧系/锕系位置错乱？

检查 `periodicTable.ts` 中元素的 `atomicNumber` 和 `category` 字段，确认定位逻辑：
- 镧系 (58-71): gridRow=8, gridCol=4+(an-58)
- 锕系 (90-103): gridRow=9, gridCol=4+(an-90)

### Q: 如何添加新的化学反应？

编辑 `src/data/chemistryDB.ts`，在 `reactions` 数组中添加：

```typescript
{
  id: 'rxn-XXX',
  name: '反应名称',
  type: '反应类型',
  reactants: ['反应物1', '反应物2'],
  products: ['产物1', '产物2'],
  equationBalanced: '配平后的方程式',
  conditions: '反应条件',
  catalyst: '催化剂',
  phenomenon: '现象描述',
  dangerLevel: 'danger',
  dangerNote: '危险提示（可选）',
  energyChange: '放热/吸热',
  reactionRate: '快/中/慢',
  applications: ['应用1', '应用2'],
}
```

### Q: 本地运行端口被占用？

Vite 会自动切换到下一个可用端口（5174、5175...）。或手动指定：

```bash
npx vite --port 6000
```

---

## 许可证

MIT © 2026
