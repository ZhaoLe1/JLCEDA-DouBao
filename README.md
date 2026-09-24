# 嘉立创 EDA 自动化・新电脑完整安装教程

> 目标：在一台全新的电脑上，装好能让 AI 直接
>
> **操作**
>
> 嘉立创 EDA 专业版的全部环境。
> 全程只需手动操作约 5 分钟，之后 AI 就能实时控制 EDA（加元件到库、画图、改封装等）。



***

## 〇、准备



| 物品   | 说明                           |
| ---- | ---------------------------- |
| 安装包  | `lceda-eda-auto-kit.zip`（本包） |
| 操作系统 | Windows 10/11                |
| 网络   | 能访问官网下载 + npm 安装依赖           |

**安装包内容预览：**



```
lceda-eda-auto-kit.zip

├── 安装说明.md              ← 精简版说明

├── 启动桥.bat               ← 一键启动桥接服务器

├── 检查连接.bat             ← 随时查连接状态

└── skills/

&#x20;   ├── easyeda-api/         ← 控制 EDA 的核心技能

&#x20;   └── lceda-symbol-generator/ ← 离线生成符号/封装文件的技能
```

**原理速览**（理解后好排查问题）：



```
你(对话) ──► AI技能(easyeda-api) ──► 桥接服务器(Node.js, 端口49620)

&#x20;               ▲                                │

&#x20;               └──── EDA扩展(Run API Gateway) ◄──┘
```

AI 通过「桥接服务器 ↔ EDA 扩展」的 WebSocket 连接，直接读写你正在运行的 EDA。



***

## 第 1 步：安装 Node.js（提供桥接服务器运行环境）



1. 打开 [https://nodejs.org/zh-cn/download](https://nodejs.org/zh-cn/download)

2. 下载 **Windows Installer (.msi)**，注意选 **LTS 版本**（22 LTS 或更高）

3. 双击 msi，一路「Next」，保持默认选项即可

4. 验证是否装好：

* 按 `Win+R`，输入 `cmd` 回车

* 输入 `node -v` 回车，看到类似 `v22.x.x` 即成功

* 再输入 `npm -v`，看到版本号即成功

> 若提示 "不是内部或外部命令"：说明没装成功或没在 PATH 里，重装一次 msi 即可。



***

## 第 2 步：安装嘉立创 EDA 专业版



1. 打开 [https://pro.lceda.cn](https://pro.lceda.cn) ，点「下载」获取 Windows 客户端

2. 安装并打开，**登录你的账号**（新电脑第一次要登录，后面全自动）

3. 确认能正常进入软件主界面



***

## 第 3 步：在 EDA 里安装扩展「Run API Gateway」

这是桥接的关键：没有它，AI 摸不到你的 EDA。



1. 打开嘉立创 EDA 专业版 → 顶部菜单 **「扩展」→「扩展管理器」**

2. 搜索 **Run API Gateway**

* 或直接用官方地址安装：[https://jlc-ext.com/item/oshwhub/run-api-gateway](https://jlc-ext.com/item/oshwhub/run-api-gateway)

1. 点击安装，安装完成后**在扩展设置里勾选两项**：

* ✅ **允许外部交互**

* ✅ **显示在顶部菜单**

1. 重启 EDA 或刷新扩展

2. 确认成功：顶部菜单栏出现 **「API Gateway」** 菜单



***

## 第 4 步：把技能放进 AI 的技能目录

把安装包里的两个技能文件夹，拷到 AI 工具加载技能的地方。



1. 解压 `lceda-eda-auto-kit.zip`

2. 找到 AI 工具的**全局技能目录**，例如本环境是：



```
\<AI工作目录>\\.user\_skills\\
```

（不同 AI 工具路径不同：OpenCode 是 `~/.config/opencode/skills`，Claude Code 是 `~/.claude/skills`，豆包 agent 是 `.user_skills`，放在它扫描 Skills 的目录即可）



1. 把包内 `skills\easyeda-api` 和 `skills\lceda-symbol-generator` **整个文件夹**拷进该目录

2. 最终效果：



```
<技能目录>\easyeda-api\SKILL.md

<技能目录>\lceda-symbol-generator\SKILL.md
```



***

## 第 5 步：一键启动桥接服务器



1. 找到安装包里的 `启动桥.bat`，双击

2. **首次运行**会自动执行 `npm install`（装依赖，需联网，约 10\~60 秒）

3. 脚本自动检查 / 启动桥接服务器，并显示连接状态

**成功标志**：看到 `"edaConnected":true`



```
{"service":"easyeda-bridge","status":"ok","edaConnected":true,...}
```

如果显示 `edaConnected:false`，做两步：



* 确认 EDA 已打开、顶部有 API Gateway 菜单

* 在 EDA 顶部点 **「API Gateway → Reconnect」**



***

## 第 6 步：验证 + 开始使用



1. 随时可双击 `检查连接.bat` 查看状态

2. 打开一个新对话，对 AI 说 **"操作 EDA"**，或直接给任务：

* "把 ESP32-C3 加到我的元件库"

* "检查当前原理图"

* "帮我画一个 NE555 最小系统"

AI 会自动连接桥并开始干活，全程不用你碰 EDA。



***

## 故障排查表



| 现象                       | 原因           | 解决                              |
| ------------------------ | ------------ | ------------------------------- |
| `node -v` 无输出            | Node.js 没装好  | 重装 Node.js 22+                  |
| 启动桥.bat 报 "未检测到 Node.js" | 同上           | 装 Node.js 后重跑                   |
| 首次 npm install 失败        | 网络问题         | 检查网络，重跑启动桥.bat                  |
| `edaConnected:false`     | EDA 扩展没连上桥   | EDA 顶部点 API Gateway → Reconnect |
| 看不到 API Gateway 菜单       | 扩展没启用        | 扩展管理器里启用，勾 "允许外部交互"" 显示在顶部菜单 "  |
| AI 说 "连接不上"              | 桥没跑 / EDA 没开 | 双击启动桥.bat；确认 EDA 开着             |
| 端口 49620 被占              | 其他程序占用       | 桥会自动在 49620-49629 里换端口，无需手动改    |
| 电脑重启后失效                  | 桥是后台进程，不会自启  | 每次开机后双击一次 启动桥.bat 即可            |



***

## 提醒



* **桥接服务器是后台进程**：`启动桥.bat` 的窗口可以关闭，桥继续在后台跑。

* **EDA 重开 / 桥被杀**：重开 EDA 后，必要时点一次 Reconnect。

* 两个技能分工：


  * `easyeda-api` → 实时控制 EDA（加库 / 画图 / 改封装）

  * `lceda-symbol-generator` → 离线生成符号 / 封装文件

  * 两者可打通：生成的数据直接喂给 API 写进库里。

  ---
### 联系我
![我的微信二维码](wx-qrcode.png)
扫码微信交流嘉立创EDA自动化相关问题
