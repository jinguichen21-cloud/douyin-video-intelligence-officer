---
name: douyin-video-intelligence-officer
description: >
  【MCP配置读取】本技能运行时必须先读取同目录下的 mcp-config.json 文件获取 MCP 服务 URL，
  然后使用这些 URL 调用 MCP 服务。严禁在未读取 mcp-config.json 的情况下直接调用 MCP。

  抖音视频情报官。全面获取抖音视频的关键数据，包括视频文案、音乐链接、封面图、
  分享链接、点赞数、播放量、发布时间、弹幕数、分享数、下载链接、收藏量、评论数、
  回复评论数等。支持通过视频链接或分享口令获取完整视频情报。
  每当用户说「抖音」「分析视频」「视频数据」「抖音链接」「视频情报」「视频播放量」
  「抖音口令」时应使用本技能。
metadata:
  label: 抖音视频情报官
---

# 抖音视频情报官

抖音视频数据情报助手。输入视频链接或分享口令，一键获取视频全维度数据。

## MCP 服务配置

本技能依赖以下 MCP 服务，运行时**必须**先读取 `mcp-config.json` 获取服务 URL：

| 环境变量名 | MCP 服务 | mcpId | 用途 |
|-----------|----------|-------|------|
| `$MCP_DOUYIN_VIDEO_URL` | 获取抖音视频数据 | 1076 | 获取视频详情、互动数据、下载链接等 |

### 配置读取方式

**约束：调用方 agent 必须在执行任何 MCP 调用前，先读取 mcp-config.json 文件**

```python
import json

with open("mcp-config.json", "r", encoding="utf-8") as f:
    config = json.load(f)

douyin_video_url = config["MCP_DOUYIN_VIDEO_URL"]["url"]
```

## 工作流程

### 第一步：获取视频标识

从用户输入中提取抖音视频链接或分享口令。常见格式：
- 完整链接：`https://www.douyin.com/video/...`
- 短链接：`https://v.douyin.com/...`
- 分享口令：包含链接的文字段落

### 第二步：调用视频数据接口

1. **读取配置**：首先读取 `mcp-config.json` 获取 MCP 服务的 URL
2. **验证配置**：检查 URL 是否已配置
3. **调用获取**：

```bash
# 先发现可用工具
python3 scripts/call_mcp.py list "$MCP_DOUYIN_VIDEO_URL"

# 获取视频数据
python3 scripts/call_mcp.py call "$MCP_DOUYIN_VIDEO_URL" <发现的工具名> \
  --params '{"video_url": "<视频链接>"}'
```

### 第三步：整理输出情报

将获取到的原始数据整理为结构化情报报告。

## 输出格式

```
## 抖音视频情报：<视频标题摘要>

### 基本信息
- 视频文案：<完整文案>
- 发布时间：<时间>
- 作者：<昵称>

### 互动数据
| 指标 | 数值 |
|------|------|
| 播放量 | X |
| 点赞数 | X |
| 评论数 | X |
| 分享数 | X |
| 收藏量 | X |
| 弹幕数 | X |

### 资源链接
- 封面图：<URL>
- 音乐链接：<URL>
- 分享链接：<URL>
- 下载链接：<URL>
```

## 注意事项

- 从用户输入中自动提取视频链接，无需用户手动格式化
- 如果用户给的是分享口令，先提取其中的链接
- 数据较大时对数字做可读化处理（如 12345 → 1.2万）
- 如果接口返回失败，提示用户检查链接是否有效
