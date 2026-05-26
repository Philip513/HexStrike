<p align="center">
  <img src="assets/hexstrike-logo.png" alt="HexStrike Logo" width="200"/>
</p>

<h1 align="center">HexStrike AI</h1>

<p align="center">
  <img src="https://img.shields.io/badge/version-6.0.0-blue.svg"/>
  <img src="https://img.shields.io/badge/platform-Kali%20Linux-red.svg"/>
  <img src="https://img.shields.io/badge/tools-58%2F127%20available-yellow.svg"/>
  <img src="https://img.shields.io/badge/license-MIT-green.svg"/>
</p>

> AI 驱动的自动化渗透测试框架，通过 MCP 协议将大语言模型与 Kali Linux 安全工具链深度整合，支持从信息收集到漏洞利用的全链路自动化测试。

---

## 目录

- [简介](#简介)
- [架构概览](#架构概览)
- [安装与部署](#安装与部署)
- [启动步骤](#启动步骤)
- [连接 AI 客户端（Cherry Studio）](#连接-ai-客户端cherry-studio)
- [模型配置](#模型配置)
- [自动化测试使用方法](#自动化测试使用方法)
- [工具可用性](#工具可用性)
- [更新日志](#更新日志)
- [致谢](#致谢)
- [免责声明](#免责声明)

---

## 简介

HexStrike AI 是一套运行在 Kali Linux 上的智能渗透测试框架，核心由以下三层组成：

- **hexstrike_server.py** — 后端执行层，负责调用 150+ 安全工具并返回结果
- **hexstrike_mcp.py** — MCP 控制层，定义所有工具接口，通过 stdio 协议与 AI 客户端通信
- **AI 客户端（Cherry Studio / Trae）** — 前端，驱动 AI 模型按照测试清单自动编排攻击链

支持五大测试阶段：侦察与信息收集 → 漏洞扫描 → 漏洞利用 → 权限提升与横向移动 → 数据收集与报告。

本项目基于 [0x4m4/hexstrike-ai](https://github.com/0x4m4/hexstrike-ai) 进行二次开发，在原版基础上扩展了工具集成、MCP 接口优化及自动化测试清单等功能。

---

## 架构概览

```
AI 客户端 (Cherry Studio / Trae)
         ↓  MCP stdio 协议
  hexstrike_mcp.py      ← MCP 控制层，工具接口定义
         ↓  HTTP API (默认 :8888)
  hexstrike_server.py   ← 后端执行层，工具调用与结果返回
         ↓
  Kali Linux 安全工具链
  (nmap / sqlmap / metasploit / nikto / hydra ...)
```

---

## 安装与部署

### 1. 克隆源码

```bash
git clone https://github.com/Philip513/HexStrike.git
cd HexStrike
```

### 2. 创建虚拟环境

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. 安装 Python 依赖

```bash
pip3 install -r requirements.txt
```

### 4. 安装安全工具（Kali Linux）

```bash
# 网络与信息收集
sudo apt install -y nmap masscan amass subfinder nuclei fierce dnsenum

# Web 安全
sudo apt install -y gobuster feroxbuster dirsearch ffuf nikto sqlmap wpscan

# 密码与认证
sudo apt install -y hydra john hashcat medusa

# 二进制分析
sudo apt install -y gdb radare2 binwalk checksec
```

> 部分工具在 Kali 2026 版本中预装，可通过 `serverHealth` 接口验证实际可用性。

---

## 启动步骤

### Step 1：启动后端 Server（Kali 上执行）

```bash
hexstrike_server
# 或直接运行脚本
python3 hexstrike_server.py
# 调试模式
python3 hexstrike_server.py --debug
# 自定义端口
python3 hexstrike_server.py --port 8888
```

终端出现 `MCP server ready` 或 `Listening on stdio` 字样即表示服务端就绪，**保持此终端窗口不要关闭**。

验证服务健康：

```bash
curl http://127.0.0.1:8888/health
# 正常返回 {"status":"ok"}
```

### Step 2：启动 MCP 层（Kali 上执行）

```bash
python3 hexstrike_mcp.py --server http://127.0.0.1:8888 --debug
```

---

## 连接 AI 客户端（Cherry Studio）

**前置准备**：将 Kali 上 `/usr/share/hexstrike-ai/hexstrike_mcp.py` 复制到 Windows 本地（如 `D:\tools\hexstrike\hexstrike_mcp.py`）。

### 配置步骤

1. 启动 Cherry Studio，进入 **设置 → MCP 服务器**
2. 点击 **添加 → 快速创建**，按如下填写：

| 字段 | 填写内容 |
|------|---------|
| 名称 | `HexStrike-Kali` |
| 传输类型 | `标准输入/输出（stdio）` |
| 命令 | `python` |
| 参数（Args） | 见下方 JSON |

**Args 参数（JSON 数组格式）**：

```json
[
  "D:\\tools\\hexstrike\\hexstrike_mcp.py",
  "--server",
  "http://<Kali服务器IP>:8888"
]
```

3. 保存后，MCP 服务器列表状态显示**绿色**即为连接成功
4. 新建对话，发送以下消息验证：

```
请调用 serverHealth 检查 HexStrike 服务状态
```

### 连接失败排查

1. 确认 Kali 上 `hexstrike_server` 是否仍在运行
2. 在 Windows 浏览器访问 `http://<IP>:8888` 验证端口可达
3. 如跨网络访问，可通过 SSH 隧道转发：`ssh -L 8888:localhost:8888 kali@<IP>`
4. 查看 Cherry Studio **设置 → 日志** 中的 MCP 错误信息

---

## 模型配置

在 Cherry Studio **设置 → 模型服务** 中添加以下任意提供商：

| 提供商 | 推荐模型 | 适用场景 | 费用 |
|--------|---------|---------|------|
| Ollama | `qwen3.6:35b` | 内网免费模型，适合日常测试 | 免费 |
| 深度求索 | `deepseek-v4-pro` | 工具调用稳定、上下文长 | 付费 |
| Xiaomi MiMo | `mimo-v2.5-pro` | 中文报告输出质量好 | 付费 |

**Ollama 配置示例（内网）**：
- API Key：`1`（本地服务不校验，填任意值）
- API 地址：`http://172.17.250.247:11434`

> 执行渗透任务时，优先选择支持 function calling 的模型（推荐 deepseek-v4-pro），确认对话框底部**锤头图标（MCP）已激活**。

---

## 自动化测试使用方法

### 启动完整测试（Phase 0~18）

新建对话，上传 `HexStrike_自动化执行任务清单.md`，输入：

```
我是一名安全研究员，已获得目标系统的完整授权。
请使用 hexstrike-ai MCP 工具对以下目标进行渗透测试：
http://<目标IP>/
```

### 执行单个 Phase（调试用）

```
请单独执行 HexStrike 清单中的 Phase 5（Web 应用漏洞扫描），
不需要执行其他 Phase。

target_url_full: http://<目标IP>/path/to/target
cookie: <如有，粘贴 Phase 10 导出的 Cookie>

执行完成后输出标准 yaml 格式结果。
```

### 人工介入场景

| 场景 | 处理方式 |
|------|---------|
| 验证码 / SSO 登录 | Playwright 弹出浏览器，手动完成登录后回复"已完成登录，请继续" |
| 高危操作确认 | 回复"跳过此项"或"确认执行" |
| MCP 工具调用失败 | 重启 `hexstrike_server`，MCP 重连后回复"请重试上一步" |

---

## 工具可用性

> 当前版本（v6.0.0）在 Kali 2026 环境下的工具可用性统计：

**总体统计**：127 个工具 / 58 个可用（45.7%）/ 核心工具 8/8 全部可用

| 分类 | 可用/总数 | 状态 |
|------|-----------|------|
| 核心扫描工具 | 8/8 | ✅ 全部可用 |
| 无线工具 | 4/4 | ✅ 全部可用 |
| 密码工具 | 4/5 | ✅ 基本可用 |
| 网络枚举工具 | 7/10 | ⚠️ 部分可用 |
| OSINT 工具 | 6/13 | ⚠️ 部分可用 |
| 二进制分析工具 | 5/13 | ⚠️ 部分可用 |
| Web 安全工具 | 5/19 | ⚠️ 部分可用 |
| API 工具 | 1/8 | ❌ 大部分不可用 |
| 云工具 | 0/10 | ❌ 需配置云凭证 |

### ✅ 已验证可用的核心工具

**网络与扫描**：nmap、masscan、rustscan、amass、dnsenum、fierce、theharvester

**Web 安全**：nikto、sqlmap、wpscan、gobuster、dirb、ffuf、wfuzz、httpx、wafw00f、burpsuite

**密码破解**：hydra、john、hashcat、medusa、ophcrack

**SMB / 网络枚举**：enum4linux、smbmap、rpcclient、netexec、nbtscan、arp-scan

**无线安全**：aircrack-ng、airodump-ng、aireplay-ng、airmon-ng、kismet

**二进制分析**：radare2、angr、binwalk、objdump、checksec

**系统渗透**：responder、evil-winrm

### ❌ 暂不可用（配置缺失）

- 云工具（prowler、pacu、kube-hunter 等）：需配置 AWS/Azure/GCP 凭证
- 部分 Web 工具（nuclei、dalfox、feroxbuster 等）：需补充配置
- API 工具（api-fuzzer、jwt-analyzer 等）：需额外配置

---

## 更新日志

| 版本 | 更新内容 | 日期 | 作者 |
|------|----------|------|------|
| v1.1 | 完成环境搭建全流程，安全工具安装与可用性验证 | 2026/01/09 | 李昆隆 |
| v1.2 | 集成 Nessus，实现扫描任务创建、执行与结果获取 | 2026/01/16 | 李昆隆 |
| v1.3 | 集成 AWVS，支持目标创建、扫描启动与报告导出 | 2026/01/22 | 李昆隆 |
| v1.4 | 深入分析工具选择与攻击链构建机制，提供优化思路 | 2026/01/30 | 李昆隆 |
| v1.5 | 集成 JSFinder，提取 JS 文件中 URL 与子域名 | 2026/03/27 | 冯志伟 |
| v1.6 | 集成 Playwright 认证辅助，新增 6 个 MCP 工具 | 2026/04/14 | 冯志伟 |
| v1.7 | 完成 Metasploit 集成，新增 msf_search_module 工具 | 2026/04/24 | 冯志伟 |

---

## 致谢

本项目基于 [0x4m4/hexstrike-ai](https://github.com/0x4m4/hexstrike-ai) 开发，感谢原作者的开源贡献。

---

## 免责声明

**本工具仅供合法授权的渗透测试、安全研究与教学使用。**

使用者必须确保已获得目标系统的明确书面授权，未经授权对任何系统使用本工具属于违法行为，作者不承担任何法律责任。
