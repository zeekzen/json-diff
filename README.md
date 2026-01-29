# DiffJson

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

DiffJson 是一个简单易用的 JSON 比较工具，支持 JSON 对象和数组的比较，提供严格模式和宽松模式，多语言支持。

## 功能特性

- 📝 **JSON 比较**：支持 JSON 对象和数组的深度比较
- 🔄 **严格/宽松模式**：数组比较支持严格模式（按顺序）和宽松模式（不按顺序）
- 🌍 **多语言支持**：支持中文、英文、日语、韩语、法语
- 📋 **JSON 格式化**：一键格式化 JSON 内容
- 📄 **行号显示**：输入框显示行号，方便定位
- 📌 **实时比较**：输入时自动进行 JSON 比较
- 📤 **复制功能**：一键复制 JSON 内容
- 🎨 **响应式设计**：适配不同屏幕尺寸
- 🔍 **SEO 优化**：支持搜索引擎索引

## 在线体验

项目部署在 Vercel 上，您可以直接访问：

TODO

## 本地安装

### 前置要求

- Node.js 16.0 或更高版本
- npm 或 yarn 包管理器

### 安装步骤

1. 克隆仓库

```bash
git clone https://github.com/zeekzen/diff-json.git
cd diff-json
```

1. 安装依赖

```bash
npm install
# 或
yarn install
```

1. 启动开发服务器

```bash
npm run dev
# 或
yarn dev
```

1. 构建生产版本

```bash
npm run build
# 或
yarn build
```

## 使用方法

1. 在左侧输入框中输入原始 JSON
2. 在右侧输入框中输入要比较的 JSON
3. 系统会自动进行比较并显示差异
4. 可以点击 "加载示例1" 或 "加载示例2" 查看示例
5. 可以切换数组模式（严格/宽松）来改变数组比较的行为
6. 可以点击 "格式化" 按钮美化 JSON 格式
7. 可以点击 "复制" 按钮复制 JSON 内容
8. 可以在右上角切换语言

## 技术栈

- **前端框架**：Vue 3
- **构建工具**：Vite
- **语言**：JavaScript
- **样式**：CSS3
- **国际化**：自定义 i18n 实现

## 项目结构

```
diff-json/
├── src/
│   ├── i18n/            # 多语言文件
│   │   ├── en-US.js     # 英文
│   │   ├── fr-FR.js     # 法语
│   │   ├── index.js     # 语言管理
│   │   ├── ja-JP.js     # 日语
│   │   ├── ko-KR.js     # 韩语
│   │   └── zh-CN.js     # 中文
│   ├── App.vue          # 主应用组件
│   ├── main.js          # 入口文件
│   └── style.css        # 全局样式
├── index.html           # HTML 模板
├── package.json         # 项目配置
├── vite.config.js       # Vite 配置
└── README.md            # 项目说明
```

## 浏览器支持

- Chrome (最新版)
- Firefox (最新版)
- Safari (最新版)
- Edge (最新版)

## 贡献

欢迎提交 Issue 和 Pull Request 来改进这个项目！

## 许可证

本项目采用 MIT 许可证 - 详情请参阅 [LICENSE](LICENSE) 文件

## 联系我们

如有问题或建议，请通过以下方式联系我们：

- GitHub Issues: [https://github.com/your-username/diff-json/issues](https://github.com/your-username/diff-json/issues)

---

**DiffJson** © 2026 by DiffJson Team. All rights reserved.
