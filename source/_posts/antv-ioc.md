---
title: AntV npm 投毒复盘：一次公司私服缓存恶意包引发的账号封禁事件
enlink: antv-ioc
date: 2026-05-29 16:16:07
categories:
- 大前端
tags:
- antv
- ioc
- npm
- supply-chain
- security
---

# AntV npm 投毒复盘：一次公司私服缓存恶意包引发的账号封禁事件

## 导语

事情的起点只是一次普通的 CI 打包失败，根因排查到最后却指向一个本该已经被 npm 官方下架的恶意版本：`@antv/expr@1.2.2`。更没想到的是，因为我在本地为了复现问题而使用公司私服重新安装依赖，直接触发了恶意代码并被安全软件告警，最终导致个人开发账号被临时封禁。

本文记录完整时间线、攻击机制、凭据风险以及处置与治理建议，供遇到类似问题的团队与个人参考。

> 说明：本文已将公司内部私服域名、账号名称、项目名、内网镜像地址、人员等信息做脱敏处理；图片路径保持原样，相关公开 IOC 与外部参考链接也保持原样。

## 结论摘要

公司私有 npm 仓库（下文统一称“公司私服”）上缓存的 npm 包 `@antv/expr@1.2.2` 包含恶意代码。开发者如果在本地执行如下命令，并安装到了该版本，就会触发恶意逻辑：

```bash
pnpm install --no-frozen-lockfile --registry https://***公司私服域名***/nexus/repository/npm/
```

本次恶意代码属于 AntV 相关 npm 包被投毒事件的一部分。它会在安装阶段执行 payload，扫描并收集本机和 CI 环境中的敏感凭据，并向 `m-kosche.com`、`t.m-kosche.com` 等域名发起请求。根据公开分析，外传数据会经过 gzip、AES-256-GCM 和 RSA-OAEP 混合加密，网络侧即使捕获请求，也很难直接还原明文内容。

最终影响是：我在本地为了复现打包机问题，使用公司私服重新安装依赖，触发了 `@antv/expr@1.2.2` 中的恶意逻辑。安全软件随后检测到供应链攻击行为并触发告警，我的内部开发账号被临时封禁。

## 背景资料

本次事件与 2026-05-19 AntV npm 包投毒事件一致，公开参考资料如下：

- [AntV npm Compromise: How Cremit's Argus Pipeline Surfaced 324 Mini Shai-Hulud Catches Within 30 Minutes](https://www.cremit.io/blog/antv-mini-shai-hulud-2026)
- [AntV npm Account Compromise: Mini Shai-Hulud Wave Hits 323 Packages](https://incidents.cremit.io/incidents/antv-mini-shai-hulud-2026)
- [Mini Shai-Hulud Strikes Again: 317 npm Packages Compromised](https://safedep.io/mini-shai-hulud-strikes-again-314-npm-packages-compromised/)
- [Mini Shai-Hulud Hits @antv Ecosystem, 639 Compromised npm Package Versions](https://socket.dev/blog/antv-packages-compromised)
- [AntV GitHub 组织公告](https://github.com/antvis)

AntV 官方在 GitHub 组织首页发布了如下公告：

> 2026-05-19 早上 10 点 `@antv` 相关的 NPM 包遭受外部蠕虫攻击投毒。我们已经在 1 小时内 deprecate 所有受感染的包，在 4 小时内联系上 NPM 官方删除所有受感染的包，目前已经完全解决相关漏洞和安全风险，请受波及的用户及时清除本地缓存重新安装依赖。

这说明 npm 官方源上的受感染版本理论上已经在 2026-05-19 当天被处理。但公司私服在 2026-05-26 仍同步并缓存了 `@antv/expr@1.2.2`，导致 2026-05-28 内部安装依赖时仍可命中恶意版本。

## 事件时间线

### 2026-05-28 10:56：前端项目打包失败

上午 11 点多，同事反馈某前端项目打包失败。查看 Jenkins 日志后，发现报错如下：

```bash
ERROR  Command failed with exit code 128: git fetch --depth 1 origin 1916faa365f2788b6e193514872d51a242876569
ssh: connect to host github.com port 22: Connection refused
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

pnpm: Command failed with exit code 128: git fetch --depth 1 origin 1916faa365f2788b6e193514872d51a242876569
ssh: connect to host github.com port 22: Connection refused
fatal: Could not read from remote repository.
```

该 Jenkins 环境没有外网访问权限，正常情况下不应该访问 `github.com`。结合以往经验，初步判断是某个依赖包中引入了类似如下形式的 GitHub 依赖：

```json
"xxx": "git+ssh://git@github.com/xxx/xxx.git#1916faa..."
```

因此当时的处理思路是：先识别具体依赖，再通过升级或回退版本，找到不依赖 GitHub 的可用版本。

### 2026-05-28 11:30：定位有问题的依赖包

为了找出具体依赖，我分两条路径排查：

1. 在打包机上手动执行 `pnpm install --reporter`，输出详细安装日志，分析 `github.com` 请求来自哪个依赖。
2. 在本地开发电脑执行 `pnpm install`，通过网络嗅探工具监听 `github.com` 请求，辅助定位目标依赖包。

打包机上通过 Docker 执行的命令（示例）如下：

```bash
docker run --rm -it --user root -v $(pwd):/app -v /data/pnpm-store:/data/pnpm-store -w /app ***/registry/library/gplane/pnpm:10.28-node22 bash -c 'pnpm config set store-dir /data/pnpm-store && git config --global url."https://github.com/".insteadOf "git@github.com:" && git config --global url."https://github.com/".insteadOf "ssh://git@github.com/" && git config --global url."https://github.com/".insteadOf "git+ssh://git@github.com/" && PNPM_DEBUG_LEVEL=debug pnpm install --no-frozen-lockfile --registry https://***公司私服域名***/nexus/repository/npm/ --reporter ndjson 2>&1 | tee /app/pnpm-install-debug.log'
```

等待打包机输出报告期间，我先在本地用官方 npm 源执行 `pnpm install`。本地没有出现 `github.com` 请求，因此基本排除了项目自身依赖变更导致的问题，进一步怀疑是公司私服上的包内容异常。

随后我为了模拟打包机环境，在本地执行了：

```bash
pnpm install --no-frozen-lockfile --registry https://***公司私服域名***/nexus/repository/npm/
```

执行后，本地也复现了 `github.com` 请求。也正是在这个过程中，本机安装到了公司私服中的恶意包，触发了供应链攻击。

### 2026-05-28 13:50：确认依赖链并临时修复打包问题

下午分析打包机上的 `pnpm install` 报告后，定位到问题依赖链如下：

```bash
@antv/expr@1.2.2
  ↓
依赖 @antv/setup
  ↓
@antv/setup 不是普通 npm 包
  ↓
而是引用 GitHub commit 1916faa365f2788b6e193514872d51a242876569
```

当时只从构建失败角度处理，认为是 `@antv/expr@1.2.2` 引入了无法访问的 GitHub 依赖，因此通过固定旧版本解决打包问题：

```json
{
  "pnpm": {
    "overrides": {
      "@antv/expr": "1.0.2",
      "@antv/vendor": "1.0.11"
    }
  }
}
```

结合后续公开分析，这里的 `@antv/setup` 并不是普通依赖错误，而是 Mini Shai-Hulud 攻击中的备用执行路径。恶意包通过 `optionalDependencies` 引入 `github:antvis/G2#1916faa365f2788b6e193514872d51a242876569`，GitHub commit 中包含带 `prepare` 生命周期脚本的 payload。即使主路径 `preinstall` 被限制，仍可能通过 GitHub 依赖触发安装期执行。

### 2026-05-28 14:33：安全软件告警，账号被封

主管收到安全平台告警邮件，邮件显示检测到我的电脑遭遇供应链攻击。出于安全策略，我的内部开发账号被临时封禁，需要重装系统后才能解封。

邮件中的 IOC 信息提到了 `m-kosche.com`。在此之前，我没有主动访问过该域名，因此开始反查本机访问来源。

### 2026-05-28 14:33：紧急止损

第一时间在本机 `/etc/hosts` 中添加如下配置，将相关域名指向本地，阻断继续访问：

```bash
127.0.0.1 m-kosche.com
127.0.0.1 t.m-kosche.com
```

需要注意的是，结合 Socket 和 SafeDep 的分析，`t.m-kosche.com` 只是主要外传通道之一。payload 还可能通过 GitHub API 创建仓库并把数据提交到 `results/` 目录作为 fallback 外传通道。因此，仅阻断域名并不能证明风险已经完全消除。

### 2026-05-28 14:40：排查攻击来源

确认 IOC 域名后，排查分为三步：

1. 使用 AI 辅助排查系统异常访问记录。
2. 根据告警邮件分析，最早检测到恶意行为的时间是 `11:34`，与本地使用公司私服执行 `pnpm install` 的时间高度重合。
3. 检索 `m-kosche.com` 和 `antv` 相关信息，找到 Cremit、SafeDep、Socket 等公开分析文章，确认这是 AntV 相关 npm 包投毒事件。

公开资料中提到，本次攻击发生于 2026-05-19，AntV 官方已经在当天对 npm 官方源上的受感染包执行 deprecate 和删除处理。理论上，2026-05-28 从官方 npm 源已经不应该安装到恶意包。

结合我本地只有在切换到公司私服后才复现问题，基本可以确认：公司私服缓存了已经从官方源移除的受感染版本。

随后下载并比对 `@antv/expr@1.2.2`，确认其内容与公开文章描述的攻击特征一致，最终锁定本次攻击来源为公司私服中的 `@antv/expr@1.2.2`。

![](https://cdn.jsdelivr.net/gh/yelog/assets/images/202605291632216.png)

## 攻击机制分析

### 触发路径

本次事件中最直接的触发路径是：

```bash
pnpm install
  ↓
安装 @antv/expr@1.2.2
  ↓
执行恶意安装脚本或 GitHub optionalDependency 中的 prepare 脚本
  ↓
扫描本机和 CI 环境中的凭据
  ↓
通过 m-kosche.com / GitHub fallback 通道外传
```

公开分析显示，受感染包通常会包含两类执行路径：

1. `preinstall` 生命周期脚本：通过 `bun run index.js` 执行根目录下混淆后的 `index.js`。
2. `optionalDependencies` GitHub 依赖：通过 `@antv/setup` 指向 `antvis/G2` 中的 orphan commit，并在该依赖的 `prepare` 阶段执行 payload。

本次 Jenkins 打包失败暴露出来的 `1916faa365f2788b6e193514872d51a242876569`，正是公开 IOC 中出现频率最高的 imposter commit。

### 凭据风险

根据 Cremit、SafeDep 和 Socket 的分析，该 payload 目标不是单一密钥，而是广泛扫描开发者电脑和 CI/CD 环境中的敏感信息，主要包括：

- GitHub PAT、GitHub App token、GitHub Actions OIDC token、`gh` CLI 配置。
- npm token、`.npmrc`、npm publish token。
- AWS、GCP、Azure 等云厂商凭据。
- SSH 私钥、Kubernetes service account、Vault token、Docker auth。
- 数据库连接串、Slack token、Stripe key、通用 API key。
- 1Password、Bitwarden、`pass`、`gopass` 等本地密码管理器相关数据。

因此，只要在受影响机器上执行过恶意包安装，就应该按照“本机可访问凭据均存在泄漏风险”处理，而不是只轮换某一个被明确捕获的 token。

### 持久化风险

公开资料还提到，该 payload 可能写入 AI 编程工具和 IDE 配置，从而在后续打开项目或启动 AI 会话时再次执行。需要重点检查以下路径：

```bash
.claude/settings.json
.claude/setup.mjs
.claude/index.js
.vscode/tasks.json
.vscode/setup.mjs
~/.claude/package/index.js
~/.codex/package/index.js
```

其中 `.claude/settings.json` 可能包含 `SessionStart` hook，`.vscode/tasks.json` 可能包含 `runOn: folderOpen` 任务。这意味着攻击可能不只发生在执行 `pnpm install` 的当下，还可能通过仓库文件或本地配置形成后续持久化。

## 攻击流程复盘

结合内部时间点和公开事件时间线，影响链路如下：

1. 2026-05-19 10:00 左右，AntV 相关 npm 包遭遇外部蠕虫攻击投毒。AntV 团队在 1 小时内 deprecate 受感染包，并在 4 小时内联系 npm 官方删除受感染包。
2. 2026-05-26 11:15，公司私服从上游同步了受感染的 `@antv/expr@1.2.2`。
3. 2026-05-28 10:56，前端项目打包时从公司私服更新到 `@antv/expr@1.2.2`。由于该包引入 `github:antvis/G2#1916faa365f2788b6e193514872d51a242876569`，而打包机没有 GitHub 访问权限，导致打包失败，并出现 `ssh: connect to host github.com port 22: Connection refused`。
4. 2026-05-28 11:30，我在本地为了复现打包机环境，使用公司私服执行 `pnpm install --no-frozen-lockfile --registry https://***公司私服域名***/nexus/repository/npm/`，从而触发恶意代码。
5. 2026-05-28 14:33，安全平台发出告警邮件，我的内部账号被封禁。

## 影响范围评估

### 打包机

打包机每次打包都通过 Docker 沙盒运行，并且没有外网访问权限。从现有现象看，恶意包在打包机上主要表现为尝试访问 GitHub 后失败，没有成功完成外传请求，因此打包机风险相对较低。

但仍建议清理打包缓存和 pnpm store，避免后续构建继续命中受感染包。

### 本地开发机

我的本地开发机已经执行过公司私服安装，并触发安全告警。因此应按高风险处理：

- 重装系统后再恢复内部开发账号。
- 轮换本机可访问的服务器、服务、CI/CD、云平台、Git、npm 等凭据。
- 检查并清理 AI 编程工具、IDE 配置和本地持久化文件。
- 审计 GitHub、GitLab、npm、云平台等账号在告警窗口后的异常行为。

### 其他开发者

如果其他人在 `2026-05-26 11:15` 到 `2026-05-28 14:36` 之间执行过如下命令，也需要按受影响处理：

```bash
pnpm install --no-frozen-lockfile --registry https://***公司私服域名***/nexus/repository/npm/
```

尤其需要关注是否安装到 `@antv/expr@1.2.2`，以及 lockfile 中是否出现以下 IOC：

```bash
@antv/expr@1.2.2
@antv/setup
github:antvis/G2#1916faa365f2788b6e193514872d51a242876569
t.m-kosche.com
m-kosche.com
```

## 处置建议

### 立即处置

1. 从公司私服中删除或隔离 `@antv/expr@1.2.2` 及相关受感染版本。
2. 清理受影响时间窗口内同步的 AntV 相关恶意包缓存。
3. 在公司网络出口和 CI egress 策略中阻断 `*.m-kosche.com`。
4. 排查公司网络、终端和 CI 日志中对 `m-kosche.com`、`t.m-kosche.com` 的访问记录。
5. 通知在风险窗口内执行过公司私服安装的开发者，进行本机排查和凭据轮换。

### 依赖治理

1. 在 CI 中尽量使用锁定依赖安装方式，避免无审查执行 `--no-frozen-lockfile`。
2. 对新增的 `preinstall`、`postinstall`、`prepare` 生命周期脚本建立审查机制。
3. 对 `optionalDependencies` 中的 `github:`、`git+ssh:`、`git+https:` 依赖建立告警或阻断策略。
4. 对刚发布的新版本设置冷却期，避免第一时间自动安装高风险新包。
5. 私服同步上游包时接入 OSV、OSSF malicious-packages、Socket、SafeDep 等恶意包情报。

### 终端排查

1. 检查本机是否存在 `.claude/setup.mjs`、`.vscode/setup.mjs`、`~/.claude/package/index.js`、`~/.codex/package/index.js` 等可疑文件。
2. 检查 `.claude/settings.json` 中是否存在异常 `SessionStart` hook。
3. 检查 `.vscode/tasks.json` 中是否存在 `runOn: folderOpen` 且执行 `setup.mjs` 的任务。
4. 检查 GitHub 账号或组织中是否出现异常仓库（例如 `<word>-<word>-<digits>` 命名风格），以及 `results/results-*.json` 文件。
5. 检查是否存在异常访问 GitHub Search API 且包含特定关键字的行为。

## 经验教训

这次事件最开始表现为一次普通的 Jenkins 打包失败，但根因不是依赖版本不兼容，而是公司私服缓存了已经从 npm 官方源移除的恶意包。

本次事件暴露出几个问题：

1. 私服缓存需要具备恶意包下架和重新扫描能力，不能只依赖首次同步时的状态。
2. `pnpm install --no-frozen-lockfile` 会扩大供应链风险，尤其是在私服缓存存在污染时。
3. GitHub 依赖、生命周期脚本和 optional dependency 都可能成为安装阶段执行 payload 的入口。
4. 安全软件告警中的 IOC 域名非常关键，应尽快反向关联本地操作时间线和依赖安装日志。
5. 开发机上的 AI 编程工具和 IDE 配置也已经成为供应链攻击的持久化目标，需要纳入常规安全排查。

后续需要把“依赖安装失败访问 GitHub”这类异常从普通构建问题提升为供应链风险信号处理，尤其是当异常 commit SHA 与公开 IOC 匹配时，应优先隔离环境并暂停继续安装，而不是直接在本地复现。
