# SearXNG

SearXNG 是一款元搜索引擎，依赖 Docker 在本地运行，因此使用它搜索就不用担心搜索 API 的免费额度或者账单了。

## Docker 部署

```bash
# 创建目录
mkdir searxng && cd searxng

# 创建 docker-compose 配置文件
cat << 'EOF' > docker-compose.yml
version: '3.8'

services:
  searxng:
    image: searxng/searxng:latest
    container_name: searxng
    ports:
      - ":7344"
    volumes:
      - ./searxng-data:/etc/searxng:rw
    environment:
      - SEARXNG_BASE_URL=http://localhost:7344/
    restart: unless-stopped
EOF

# 初始化
mkdir -p ./searxng-data
cat << 'EOF' > ./searxng-data/settings.yml
use_default_settings: true

server:
  secret_key: "4PVP89BdcVoHX0E43oXZigtP03wrPEc"
  image_proxy: true

search:
  safe_search: 0
  autocomplete: ""
  formats:
    - html
    - json

engines:
  - name: google
    disabled: false
  - name: bing
    disabled: false
  - name: yandex
    disabled: false
  - name: baidu
    disabled: false
  - name: brave
    disabled: true
  - name: duckduckgo
    disabled: true
  - name: startpage
    disabled: true
  - name: qwant
    disabled: true
  - name: "google cse"
    disabled: true
  - name: mojeek
    disabled: true
EOF

docker compose up -d
docker compose down
```

## Docker 中 Open WebUI 设置

Searxng 查询接口地址：`http://host.docker.internal:7344/search?q=<query>`


