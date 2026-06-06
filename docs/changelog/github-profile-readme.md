# GitHub Profile README 项目日志

> 仓库：`David-qiuwenhui/David-qiuwenhui`（main 分支 `README.md`）
> 时间：2026-06-06

---

## 项目概览

重新设计 GitHub 个人主页 README，采用 Claude Code 风格暗色终端主题。单文件方案，全部使用公开 SVG/API 服务渲染动态内容，无需 GitHub Actions、PAT 或自建服务。

---

## 设计方案

### 主题风格

Neofetch 终端信息风格 + 经典打字循环动画 + 统计卡片布局。

### 配色方案（Claude Code 品牌色）

| 用途 | 色值 | 预览 |
|------|------|------|
| 背景 | `#0D1117` | 深色底 |
| 品牌强调色 | `#D97757` | 珊瑚橙 |
| 主文本 | `#F5E6D3` | 奶油色 |
| 次要文本 | `#FFB38A` | 浅橙 |
| 成功色 | `#4EBA65` | 绿色 |
| 信息色 | `#61AFEF` | 蓝色 |
| 辅助灰 | `#B1ADA1` | 暖灰 |
| 错误色 | `#FF6B80` | 粉红 |

### ASCII Art 风格选择

提供了 6 种风格供选择（Banner 方块、Slant 斜体、Big 大字、Univers 圆润、Star Wars 星战、Chunky 厚重），最终选定 **Banner 方块风格（方案 A）**——实心 `█` 字符，像素感强，最有终端味道。

### README 布局（从上到下）

1. **终端标题栏** — 红/黄/绿圆点 + `wenhui@github ~ ./profile.sh`
2. **Neofetch 风格名称区** — `QIUWENHUI` ASCII Art + 个人信息字段
3. **打字机动效** — 4 句循环，`readme-typing-svg` 服务
4. **分隔线**
5. **GitHub Stats + Top Languages** — 双栏统计卡片
6. **连续贡献统计** — Streak Stats 卡片
7. **技术栈标签** — shields.io badges
8. **Profile Views 计数器**

### 设计文档

- 设计规格：`docs/superpowers/specs/2026-06-06-github-profile-readme-design.md`
- 实施计划：`docs/superpowers/plans/2026-06-06-github-profile-readme.md`

---

## 外部服务依赖

| 服务 | URL | 用途 | 状态 |
|------|-----|------|------|
| shields.io | `https://img.shields.io/badge/...` | 终端标题栏、技术栈 badges | 稳定 |
| readme-typing-svg | `https://readme-typing-svg.demolab.com` | 打字机动效 SVG | 稳定 |
| gh-stats.work | `https://gh-stats.work/API` | GitHub Stats + Top Languages 卡片 | 稳定（2026-06 替换） |
| streak-stats | `https://streak-stats.demolab.com` | 连续贡献统计 | 稳定 |
| komarev | `https://komarev.com/ghpvc/` | Profile Views 计数器 | 稳定 |

---

## 实施记录

### 第一阶段：初始部署（2026-06-06）

**Commit:** `0da5672edc8e57bfd9cd46705bca0b1dff8d1839`
**远程 SHA:** `e41e11a07c003b3225dc64ffa7c6686710681df4`

使用 Subagent-Driven Development 工作流，分 4 个 Task 完成：

1. **Task 1:** 创建 README.md 完整文件（终端标题栏 + ASCII Art + neofetch 信息区 + 打字机动效 + 统计卡片 + 技术栈 badges + 访问计数器）
2. **Task 2:** 验证外部服务可用性（curl 测试所有 URL 返回 200）
3. **Task 3:** 通过 GitHub API 推送到 `David-qiuwenhui/David-qiuwenhui` 仓库
4. **Task 4:** 浏览器验证最终效果（Playwright 截图 + 网络请求检查）

初始版本使用 `github-readme-stats.vercel.app` 作为统计卡片服务。

### 第二阶段：修复损坏图片（2026-06-06）

**Commit:** `c053067647bd8daae03a20b6a54cfb4853394037`
**远程 SHA:** `ee29c09568a70b6b8c834205aea942457001f995`

#### 问题

GitHub Stats 卡片和 Top Languages 卡片在 GitHub 主页上显示为断裂图片。

#### 根因分析

通过网络请求捕获发现两个 `github-readme-stats.vercel.app` 请求被浏览器 ORB（Opaque Response Blocking）机制阻断：

```
github-readme-stats.vercel.app/api?username=David-qiuwenhui...
  → [FAILED] net::ERR_BLOCKED_BY_ORB

github-readme-stats.vercel.app/api/top-langs/?username=David-qiuwenhui...
  → [FAILED] net::ERR_BLOCKED_BY_ORB
```

**深层原因：** `github-readme-stats.vercel.app` 公共实例已不稳定。官方维护者于 2026 年 1 月确认公共服务因 Vercel 赞助停止而 paused（503 DEPLOYMENT_PAUSED），且推荐自部署或 GitHub Actions 方案。GitHub 的 camo 代理在转发 SVG 响应时触发 ORB，导致图片无法显示。

#### 修复方案

将统计卡片服务从 `github-readme-stats.vercel.app` 替换为 `gh-stats.work`（基于 Cloudflare Workers 的社区部署）。

**具体变更：**

```diff
- https://github-readme-stats.vercel.app/api?username=David-qiuwenhui&show_icons=true&theme=chartreuse-dark&title_color=D97757&...
+ https://gh-stats.work/API?username=David-qiuwenhui&show_icons=true&title_color=D97757&...

- https://github-readme-stats.vercel.app/api/top-langs/?username=David-qiuwenhui&layout=compact&theme=chartreuse-dark&...
+ https://gh-stats.work/API/top-langs?username=David-qiuwenhui&layout=compact&title_color=D97757&...
```

同时移除了 `theme=chartreuse-dark` 参数（自定义颜色参数已覆盖所有颜色，无需额外主题）。

#### 验证

Playwright 浏览器验证，所有 15 张图片均 `complete: true`：

| 图片 | 尺寸 | 状态 |
|------|------|------|
| terminal title bar | 222x20 | OK |
| Typing SVG | 600x40 | OK |
| GitHub Stats | 467x195 | OK |
| Top Languages | 300x165 | OK |
| GitHub Streak | 495x195 | OK |
| Vue.js | 63x20 | OK |
| TypeScript | 87x20 | OK |
| Python | 67x20 | OK |
| Java | 53x20 | OK |
| Spring Boot | 93x20 | OK |
| Webpack | 77x20 | OK |
| Claude | 65x20 | OK |
| Deep Learning | 109x20 | OK |
| AI Agent | 59x20 | OK |
| Profile Views | 112x20 | OK |

---

## 维护指南

### 日常检查

如果图片再次出现断裂，按以下步骤排查：

1. **确认服务可用性：**
   ```bash
   # 测试各服务（应返回 200）
   curl -s -o /dev/null -w "%{http_code}" "https://gh-stats.work/API?username=David-qiuwenhui"
   curl -s -o /dev/null -w "%{http_code}" "https://streak-stats.demolab.com?user=David-qiuwenhui"
   curl -s -o /dev/null -w "%{http_code}" "https://readme-typing-svg.demolab.com?font=Fira+Code&size=18&lines=Hello"
   ```

2. **确认 GitHub camo 缓存问题：** camo 代理缓存约 10 分钟，更新后可能需要等待。可以尝试在 URL 后加 `&v=2` 参数强制刷新。

3. **浏览器验证：** 打开 `https://github.com/David-qiuwenhui`，检查所有图片是否正常渲染。

### 更新 README

1. 编辑本地 `README.md`
2. 获取当前远程 SHA：
   ```bash
   gh api repos/David-qiuwenhui/David-qiuwenhui/contents/README.md --jq '.sha'
   ```
3. 推送到远程（或通过 GitHub API `create_or_update_file`）

### 如果 gh-stats.work 也不稳定

按优先级选择替代方案：

1. **首选：GitHub Actions 生成静态 SVG**
   - 官方推荐，最可靠
   - 参考：https://github.com/anuraghazra/github-readme-stats#github-actions
   - 需在仓库中添加 workflow，定时生成 SVG 并 commit

2. **次选：自部署 Vercel 实例**
   - Fork `anuraghazra/github-readme-stats`
   - 在 Vercel 部署，设置 `GITHUB_TOKEN` 环境变量
   - 将 README 中的域名替换为自己的 Vercel 域名

3. **临时：使用其他社区部署**
   - `https://github-readme-stats-sigma-five.vercel.app`
   - 或搜索 GitHub Issues 获取最新可用部署

### 修改个人信息

个人信息集中在 neofetch 区域（第 22-29 行）：

```
Title:   Full-Stack Developer & AI Engineer
Focus:   Deep Learning / Semantic Segmentation
Stack:   Vue.js, TypeScript, Python, Java
Tools:   Claude Code, AI Agent, Webpack
Uptime:  5+ years
```

技术栈 badges 在第 53-63 行，按需增删。

打字机循环文本在 typing-svg URL 的 `lines=` 参数中（第 36 行），用 `;` 分隔多行，`+` 替代空格，`%7C` 替代 `|`。

---

## 文件清单

| 文件 | 说明 |
|------|------|
| `README.md`（本地） | 源文件，编辑后推送至远程 |
| `docs/superpowers/specs/2026-06-06-github-profile-readme-design.md` | 设计规格文档 |
| `docs/superpowers/plans/2026-06-06-github-profile-readme.md` | 实施计划文档 |
| `docs/changelog/github-profile-readme.md` | 本文件，项目日志 |
