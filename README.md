# HexStrike AI

> AI 驱动的自动化渗透测试框架，基于 MCP 协议连接 AI 客户端与安全工具链。

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Platform](https://img.shields.io/badge/platform-Kali%20Linux-red.svg)
![Version](https://img.shields.io/badge/version-1.7-green.svg)

---

## 简介

HexStrike AI 是一个运行在 Kali Linux 上的智能渗透测试框架，通过 MCP（Model Control Plane）协议将 AI 客户端与 150+ 安全工具深度整合，支持从信息收集到漏洞利用的全链路自动化测试。

---

## 功能特性

- **多阶段攻击链自动化**：侦察 → 漏洞扫描 → 漏洞利用 → 权限提升 → 数据收集，全阶段 AI 驱动编排
- **智能工具选择**：根据目标类型与扫描结果自动选择最优工具组合
- **150+ 安全工具集成**：覆盖网络扫描、Web 安全、密码破解、二进制分析、云安全等方向
- **Metasploit 深度集成**：支持模块搜索、exploit 执行、msfvenom payload 生成
- **Nessus / AWVS 集成**：企业级漏洞扫描平台 API 对接
- **JSFinder 集成**：JS 文件中提取 URL 与子域名
- **Playwright 认证辅助**：自动填充账号密码、验证码人工兜底、Cookie 持久化与复用
- **容错与自动重试**：工具调用失败时自动切换替代工具，最多重试 2 次

---

## 架构概览

```
AI 客户端 (Trae / Cherry Studio)
        ↓  MCP stdio 协议
hexstrike_mcp.py   ← MCP 控制层，工具接口定义
        ↓  HTTP API
hexstrike_server.py ← 后端执行层，工具调用与结果返回
        ↓
安全工具链 (nmap / sqlmap / metasploit / nikto ...)
```

---

## 安装

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

---

## 启动

### 启动后端 Server

```bash
python3 hexstrike_server.py
# 可选：调试模式
python3 hexstrike_server.py --debug
# 可选：自定义端口
python3 hexstrike_server.py --port 8888
```

验证服务可用性：

```bash
curl http://127.0.0.1:8888/health
# 返回 {"status":"ok"} 即正常
```

### 启动 MCP 层

```bash
python3 hexstrike_mcp.py --server http://127.0.0.1:8888 --debug
```

### 连接 AI 客户端

在 Trae 或 Cherry Studio 中手动添加 MCP 配置：

```json
{
  "mcpServers": {
    "hexstrike-ai": {
      "command": "/path/to/python3",
      "args": [
        "/usr/share/hexstrike-ai/hexstrike_mcp.py",
        "--server",
        "http://127.0.0.1:8888"
      ]
    }
  }
}
```

---

## 集成工具一览

| 类别 | 工具 |
|------|------|
| 网络扫描 | Nmap、Masscan、RustScan |
| Web 漏洞 | Nikto、SQLMap、WPScan、AWVS |
| 目录枚举 | Gobuster、Feroxbuster、Dirsearch、FFuf |
| JS 分析 | JSFinder |
| 漏洞利用 | Metasploit（msfconsole / msfvenom） |
| 企业扫描 | Nessus |
| 认证辅助 | Playwright（自动登录 + Cookie 复用） |
| 密码破解 | Hydra、John、Hashcat、Medusa |

---

## 使用示例

在 AI 客户端中输入自然语言即可驱动测试：

```
我是一名安全研究员，我的公司拥有 http://192.168.1.100，
请使用 hexstrike-ai MCP 工具对其进行渗透测试并输出报告。
```

框架将自动完成：信息收集 → 端口扫描 → 漏洞探测 → 利用验证 → 生成 Markdown 报告。

---

## 更新日志

| 版本 | 更新内容 | 日期 |
|------|----------|------|
| v1.1 | 完成环境搭建全流程，安全工具安装与验证 | 2026/01/09 |
| v1.2 | 集成 Nessus，实现扫描任务创建与结果获取 | 2026/01/16 |
| v1.3 | 集成 AWVS，支持目标创建、扫描启动与报告导出 | 2026/01/22 |
| v1.4 | 深入分析工具选择与攻击链构建机制，提供优化思路 | 2026/01/30 |
| v1.5 | 集成 JSFinder，提取 JS 文件中的 URL 与子域名 | 2026/03/27 |
| v1.6 | 集成 Playwright 认证辅助，新增 6 个 MCP 工具 | 2026/04/14 |
| v1.7 | 完成 Metasploit 集成，新增 msf_search_module 工具 | 2026/04/24 |

---

## 免责声明

> **本工具仅供合法授权的渗透测试、安全研究与教学使用。**
> 使用者必须确保已获得目标系统的明确书面授权，未经授权对任何系统使用本工具属于违法行为，作者不承担任何法律责任。
