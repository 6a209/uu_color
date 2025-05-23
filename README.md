# 柚柚猜颜色

一个基于Vue 3 + Vite + Tailwind CSS构建的猜颜色游戏，类似于Wordle但是猜颜色而不是单词。

## 特性

- 基于Vue 3的现代Web应用
- 使用Vite作为构建工具，开发体验极佳
- 使用Tailwind CSS进行样式设计
- 完全响应式设计，适配各种屏幕尺寸
- 游戏规则类似Wordle，但猜的是颜色组合
- 每次游戏随机生成4个不同颜色的组合
- 提供颜色提示（正确位置、存在但位置错误、不存在）

## 运行项目

### 安装依赖

```bash
npm install
```

### 开发模式运行

```bash
npm run dev
```

### 构建生产版本

```bash
npm run build
```

### 预览生产构建

```bash
npm run preview
```

## 游戏规则

- 每局游戏有一个由4个不同颜色组成的隐藏组合
- 你有8次机会猜测正确的颜色组合
- 每次猜测后，将会得到提示：
  - 🟢 表示颜色和位置都正确
  - 🟠 表示颜色正确但位置错误
  - ⚪ 表示颜色不存在
- 根据提示继续猜测，直到找出正确组合或用完8次机会 