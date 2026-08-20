# 学习笔记 📒

一个 15 岁初中生的折腾日记。记录每天捣鼓电脑软硬件学到的知识，从零开始，边学边写。

---

## 2026-08-14 · 第一天：OpenClaw 上线与远程访问

今天是我的 AI 助手（皮皮虾 🦐）诞生的日子，也是我第一次完整地折腾服务器环境。

### 📌 今天干了什么

1. **OpenClaw 上线**
   - 完成初始化配置，确定了助手身份（名字、性格、习惯）。
   - 明白了「工作区」的概念：助手的所有记忆都存在 `~/.openclaw/workspace` 里，随时可以翻。

2. **让 iPad 能远程聊天（折腾最久的一项）**
   - 一开始想让 iPad 直接访问局域网 IP，结果报错：
     `control ui requires device identity (use HTTPS or localhost secure context)`
   - **学到的知识：** 浏览器的「安全上下文」规则——只有 HTTPS 或 localhost 才被认为是安全环境。Control UI 必须在安全环境下才能用设备身份登录，所以明文 HTTP 从别的设备访问会被拒。
   - **最终解法：** Tailscale（一个把多台设备组成加密虚拟内网的软件）的 Serve 功能，把聊天页面以 HTTPS 形式发布到自己的设备网络里。
   - 中间还学会了：
     - `sudo` 的用途：敏感操作需要管理员权限，密码输入不显示字符是正常的
     - SSH 远程连接：从 Windows 电脑用 `ssh 用户名@IP` 连进树莓派
     - `tailscale set --operator=用户`：把 Tailscale 的管理权限交给普通用户

3. **技能大扫除（53 个技能里 32 个需要 setup）**
   - 技能（skill）= 助手的「技能卡」，每个需要对应的命令行工具才能启用。
   - 用 `openclaw skills check` 查看哪些缺依赖，缺什么一目了然。
   - **自己动手装了 4 个：** mcporter（MCP 工具管理）、oracle（AI 代码审查）、xurl（X/Twitter 工具）、session-logs（翻历史记录）
     - 装法：npm 全局安装 + 静态二进制下载到 `~/.local/bin`（用户目录，不需要管理员权限）
   - **装了 3 个系统包：** gh（GitHub 官方命令行）、tmux（终端分屏）、ffmpeg（音视频处理）
     - 踩坑：第一次 SSH 中途断线，`apt install` 被中断，一个包都没装上。第二次用 `nohup` 让命令脱离窗口后台运行，成功。
   - **学到的知识：**
     - PATH 环境变量 = 系统找命令的「搜索路径」列表
     - 网关会缓存技能状态，装完工具要重启才生效
     - GitHub 直连国内不稳定，下载失败可以换镜像或走系统源

4. **GitHub CLI 登录**
   - 用设备授权流程登录：终端生成一次性代码 → 在任意设备浏览器打开 `github.com/login/device` 输入代码 → 授权完成。
   - 明白了「设备授权」的原理：服务器没有浏览器时，用代码 + 网址让你在别的设备上确认身份。

### 💡 今日金句

> 报错不可怕，可怕的是不读报错信息。今天的三个大坑（安全上下文、SSH 断线、缓存状态）全都是靠认真读错误信息解决的。

### 📊 成果

- 29 / 53 个技能就绪（一天激活 8 个）
- iPad 随时随地远程聊天 ✅
- GitHub 账号打通 ✅
- 第一个仓库建成 ✅

---

## 2026-08-14 · 晚间加餐：从白天干到黑夜

### 🌐 5. 网络诊断实战：GitHub 打不开怎么办

- 症状：github.com 网页间歇性打不开、git 推代码超时。
- **诊断方法（先测试再下结论）**：
  1. 查 DNS 解析——`nslookup github.com` 看地址对不对（结果是：解析正常，排除 DNS 污染）
  2. 直连测试——`curl -sI https://github.com` 超时；测试 5 个 GitHub IP 全部连不上（结论：IP 层被掐，不是电话簿的问题）
- **解法：SSH-443 通道**（国内用户标准方案）
  - 生成密钥：`ssh-keygen -t ed25519`
  - 把公钥加到 GitHub（Settings → SSH keys）
  - 配置 `~/.ssh/config`：`Host github.com → HostName ssh.github.com → Port 443`
  - 从此 git 推送走 `ssh.github.com:443`，**完全绕开被墙的网页端口**，稳定如老狗
- 辅助工具：Watt Toolkit（Windows 一键加速 GitHub）；手机流量临时救急。
- 学到了：**测试 > 猜测**——改配置前先用 5 分钟定位真凶。

### 📸 6. 图片识别三档方案

- 当前 AI 模型不支持看图 → 搭了三档：
  1. **tesseract OCR**（快）：`tesseract 图片 -l chi_sim+eng out`，文字截图秒读
  2. **本地视觉模型**（Ollama qwen3-vl，慢但能看懂图）
  3. **云端视觉模型**（阿里云百炼，待注册）

### 🧠 7. 记忆搜索索引修复

- 症状：记忆搜索报错“index metadata is missing”，自动暂停。
- 根因：记忆索引默认用 OpenAI 嵌入模型，没有 key。
- 解法：改用本地 Ollama 嵌入模型 `qwen3-embedding:0.6b`，`openclaw memory index --force` 重建。
- 学到了：嵌入模型 = 把文字变成向量的“图书管理员”，负责语义检索。

### 💬 8. 微信渠道上线

- OpenClaw 官方有腾讯微信团队开发的插件（`@tencent-weixin/openclaw-weixin`），扫码登录即用。
- 微信里的我和网页里的我是**同一个大脑**（长期记忆共享），但对话上下文分开（不同“房间”）。

### 🍊 9. 明日预告：Orange Pi Zero 3 折腾计划

- 给 1.5GB 小机器刷 Armbian 精简版（无桌面）
- 项目：AdGuard Home 全屋广告拦截（DNS 层拦截，全家生效）+ Syncthing 照片备份
- 镜像已备好（U 盘 + 电脑双份），明天开干！

---

## 📅 每日记录

- [2026-08-15 · Orange Pi 变身全屋广告拦截服务器 🦐](daily/2026-08-15.md)
- [2026-08-16 · 网络调整：DNS 回退 + 无线改造](daily/2026-08-16.md)
- [2026-08-19 · InfoScreen 家庭信息屏项目开工 🖥️](daily/2026-08-19.md)
- [2026-08-20 · Orange Pi 变身电视盒子 + 自制毫秒级遥控器 📺](daily/2026-08-20.md)

---

*持续更新中…*

