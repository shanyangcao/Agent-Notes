### 一、什么是 FastAPI

`FastAPI` 是一个现代、快速（高性能）的 Web 框架，用于构建 API。它基于 Python 3.6+ 的类型提示，具有以下特点：

**核心优势**：

- **快速**：性能媲美 NodeJS 和 Go（基于 `Starlette` 和 `Pydantic`）
- **高效开发**：开发速度提升约 200-300%
- **更少错误**：减少约 40% 的人为错误
- **直观**：强大的编辑器支持，自动补全
- **简单**：易于学习和使用
- **简洁**：减少代码重复
- **标准化**：基于 `OpenAPI` 和 JSON Schema

### 二、安装

```bash
pip install fastapi
pip install "uvicorn[standard]"  # ASGI 服务器
```

### 三、第一个 FastAPI 应用

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}

# 运行：uvicorn main:app --reload
```

