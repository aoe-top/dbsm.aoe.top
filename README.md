# BDSM 倾向人格测试

这是一个简单的前端小应用，基于 Vite + Vue 3，提供非诊断性的问卷测试，用于自我探索。测试结果仅供参考，所有真实活动需建立在成年、知情、双方自愿与安全原则之上。

快速开始（Windows PowerShell）：

```powershell
# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

默认会在本地打开一个开发地址（通常 http://localhost:5173）。

注意：本示例不包含后端或数据上报。若需保存结果或增加更多题目，请编辑 `src/data/questions.js`。

部署到 GitHub Pages
-------------------

项目包含方便的一键部署脚本（使用 `gh-pages`）：

1. 本地快速部署（将 `dist/` 推送到 `gh-pages` 分支）：

```powershell
npm install
npm run deploy
```

2. 自动部署：已添加 GitHub Actions 工作流，推送到 `main` 分支时会自动构建并将 `dist/` 发布到 `gh-pages` 分支。

注意：仓库设置中需启用 GitHub Pages，选择 `gh-pages` 分支作为发布源（通常 Actions 会自动创建并推送该分支）。
