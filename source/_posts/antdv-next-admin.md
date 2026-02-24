---
title: 别再从 0 造后台了：antdv-next-admin，开箱即用的 Vue 3 中后台脚手架
enlink: antdv-next-admin
date: 2026-02-24 15:28:51
categories:
- 大前端
tags:
- antdv-next
- vue3
- ant-design
---

![antdv-next-admin](https://cdn.jsdelivr.net/gh/yelog/assets/images/202602241531080.png)

做中后台项目，最容易“耗时间但不出成果”的，不是业务功能，而是重复的基础建设：登录鉴权、权限控制、菜单路由、表格表单、主题切换、国际化、Mock 联调。

`antdv-next-admin` 就是为了解决这个问题而做的一个现代化脚手架：让你少花时间搭地基，把精力集中到真正有价值的业务实现上。

- GitHub：[`yelog/antdv-next-admin`](https://github.com/yelog/antdv-next-admin)  
- 在线体验：[Demo](https://antdv-next-admin.yelog.org/dashboard)  
- 默认账号：`admin / 123456`、`user / 123456`

## 这个脚手架到底能帮你省什么时间？

### 1. 一套成熟的技术栈，开箱即用
- Vue 3.4 + TypeScript 5 + Vite 5
- Pinia + Vue Router 4
- `antdv-next` 组件体系
- Axios、vue-i18n、ECharts 等常用能力全接好

### 2. 权限体系不是“样子货”
- RBAC 权限模型
- 动态路由与菜单控制
- 按钮级权限和指令权限
- 角色/用户/权限等系统管理页面示例

### 3. 中后台常见体验，基本都内置了
- 多标签页（KeepAlive 缓存）
- 全局搜索（`Ctrl/Cmd + K`）
- 亮色/暗色/跟随系统
- 垂直/水平布局切换
- 中英文国际化切换

### 4. 不只是“壳子”，还有可复用的业务能力
- ProTable（查询、分页、列配置、值类型渲染）
- ProForm（配置化表单、验证、布局）
- ProModal（拖拽、全屏、表单集成）
- 富文本编辑器、验证码组件、图标选择器、水印组件等

### 5. 联调效率很高
- 开发环境支持完整 Mock
- 常见 CRUD 接口都能直接跑
- 前后端可并行开发，减少等待

## 5 分钟上手

```bash
git clone https://github.com/yelog/antdv-next-admin.git
cd antdv-next-admin
npm install
npm run dev
```

打开 `http://localhost:3000` 即可体验。

常用命令：

```bash
npm run type-check
npm run build
npm run build:check
npm run preview
```

## 适合哪些团队和项目？

- 想快速启动中后台项目的个人开发者
- 需要统一工程规范的中小团队
- 希望沉淀“可复用管理端底座”的业务线
- 需要权限、主题、多语言、Mock 一次性配齐的项目

## 为什么我会推荐它？

我更看重脚手架的两个指标：

- 能不能快速进入业务开发，而不是陷入“搭架子”
- 能不能在后续迭代中保持可维护、可扩展

`antdv-next-admin` 在这两点上都做得比较扎实：基础能力全，目录和职责清晰，示例场景覆盖也比较完整，适合作为长期演进的中后台基座。

## 最后

如果你正在做 Vue 3 中后台，这个项目值得试一下：

- 仓库地址：[`https://github.com/yelog/antdv-next-admin`](https://github.com/yelog/antdv-next-admin)

如果这个项目帮你省下了时间，欢迎点一个 **Star**。  
你的每一颗 Star，都是项目持续迭代最直接的动力。谢谢支持。

