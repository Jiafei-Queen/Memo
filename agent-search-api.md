# Agent 用搜索 API 横向测评

本次测评的选手有： `tavily`、`exa`、`firecrawl`、`serper`、`searxng` 

## 一、不适合 Agent 使用

### 1. SearXNG

这是一个元搜索引擎，通过调用 Google、Bing、DuckduckGo 这类搜索引擎来汇总结果

- 特点：开源、可自建

- 缺点！：
    1. 绝大多数的搜索引擎会频发出现人机验证和限制
    2. 搜索实效性和命中率很低

> 总结：几乎没有实用价值

### 2. Serper

这是一个 Google 搜索 API

- 特点：Google 搜索
- 免费额度：2500 次/月

- 缺点！：
    1. 返回的 `snippet` 很短，模型基本无法获取有效信息！

## 二、不错的搜索 API 介绍

### 1. Tavily

这是一个老牌 Agent 搜索工具

- 特点：可靠；综合
- 免费额度：1000 credits/月（基础/进阶搜索：1/2 credit）

- 缺点！：
    1. 似乎找不到，硬要说就是免费额度有点少（但注册很简单）

### 2. Firecrawl

专为 AI 打造的网页抓取与爬虫引擎

- 特点：搜索功能丰富（但很多跟本次话题无关，例如地图、页面交互）
- 免费额度：1035 次/月

- 缺点！：
    1. 搜索命中率相比较低

### 3. Exa

针对 AI 的神经搜索引擎

- 特点：搜索质量超高！！；慷慨大方；搜索实效性和命中率极高
- 免费额度：注册送 $20（2800次搜索），每月自动打 $10（1400次搜索）

- 缺点！：
    1. 对于日常使用，返回的结果太过详细丰富了，结果占用太多上下文
    2. 返回结果的权威性不够，导致出现很多不想干内容... 污染较大

## 三、不错的搜索 API 对比

### 1. Tavily vs. Firecrawl vs. Exa 搜索质量对比

- Q1: AI 最新新闻 2026年8月
- Q2: Python 3.14 官方文档 新特性
- Q3: open-webui github issue bug 搜索工具 报错
- Q4: stack overflow python asyncio aiohttp proxy error
- Q5: github open-webui tools.py fetch_url 源码

| 主题 | FIRECRAWL | TAVILY | EXA |
| :--- | :--- | :--- | :--- |
| 1 时效新闻 | RFI/BBC 但要过滤网易/YT 噪音 | yam 深度文+RFI，权威性最好 | 多且新，但全为自媒体/聚合，权威性最弱 |
| 2 官方文档 | 官方主站命中准 | 台湾镜像，官方主站靠后 | 官方主站中英+CPython 源码，最强 |
| 3 GitHub issue | #25038 等+官方文档 | 聚焦最新搜索回归 | 数量最多，全为真实 bug |
| 4 Stack Overflow | 直接命中 SO #61278106 | 未直接命中 SO | 直接命中 5 个 SO 页，最强 |
| 5 源码 | 只到项目级 | 直接命中 tools.py | 直接命中 tools.py + fetch_url 相关 #26135 |

> 在技术性相关的工作中：Exa > Tavily > Firecrawl

### 2. Tavily vs. Exa 返回 token 数对比

- Q1: Mamba state space model vs Transformer comparison
- Q2: DeepSeek 新模型 开源 2026

| 查询 | Tavily（上轮实测） | Exa（本轮实测） | Exa / Tavily |
|---|---|---|---|
| Q1 英文深度研究 | 903–1,170 tok | **18,600–32,900 tok** | **≈25 倍**（16–36x） |
| Q2 中文查询 | 1,884–2,943 tok | **4,000–6,400 tok** | ≈2.2 倍（1.4–3x） |

> 小结：平时用 Tavily 顾家，干活就用 Exa
