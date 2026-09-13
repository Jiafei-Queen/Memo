# 更新 Open WebUI

## 一、停止/删除原有容器

```bash
docker stop open-webui
docker rm open-webui
```

## 二、拉取 Docker 镜像

```bash
docker pull ghcr.m.daocloud.io/open-webui/open-webui:main

# 打包/加载镜像（如果需要）
# docker save -o open-webui.tar ghcr.m.daocloud.io/open-webui/open-webui:main 
# docker load -i open-webui.tar
```

## 三、构建容器

```
docker run -d \
  -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  -v /opt/open-webui/custom.css:/app/build/static/custom.css:ro \
  --name open-webui \
  --restart always \
  --network open-webui-net \
  -e RAG_EMBEDDING_ENGINE=openai \
  -e AUDIO_STT_ENGINE=webapi \
  -e DATABASE_POOL_SIZE=8 \
  -e DATABASE_SQLITE_PRAGMA_CACHE_SIZE=-2000 \
  -e DATABASE_SQLITE_PRAGMA_MMAP_SIZE=0 \
  -e DATABASE_ENABLE_SESSION_SHARING=false \
  -e ENABLE_AUTOCOMPLETE_GENERATION=false \
  -e ENABLE_IMAGE_GENERATION=false \
  -e ENABLE_IMAGE_PROMPT_GENERATION=false \
  -e CHAT_RESPONSE_STREAM_DELTA_CHUNK_SIZE=10 \
  -e GLOBAL_LOG_LEVEL=WARNING \
  -e ENABLE_COMPRESSION_MIDDLEWARE=false \
  -e UVICORN_WS_PER_MESSAGE_DEFLATE=false \
  -e ENABLE_RAG_LOCAL_WEB_FETCH=true \
  ghcr.m.daocloud.io/open-webui/open-webui:main
```
