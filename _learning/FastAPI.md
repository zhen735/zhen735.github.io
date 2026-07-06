# FastAPI`

这个文档主要是为了方便下一次开启一个新的hettp服务调用本地模型，算是一个模板吧

## 核心部分

1. 实例创建
```
app = FastAPI() # 可以通过title参数来设置实例的名字
```
2. 全局资源设置
声明全局资源，如model、相机等

3. 接口设置

`/health`: 健康检查接口
`/detect`：业务接口，真正走模型推理的地方，包含五个步骤：
a. 读取请求体
b. 参数校验
c. 解码输入
d. 模型调用
e. 返回json

4. 错误码返回
```raise HTTPException(status_code=400, detail="empty request body")```
提示错在哪里

5. 异步推理model_lock
```with model_lock:
    result = model(...)
```

## 整体模板

```
#!/usr/bin/env python3

from __future__ import annotations

import argparse
import threading
import time
from typing import Any

import uvicorn
from fastapi import FastAPI, HTTPException, Request


app = FastAPI(title="My HTTP Service")

model = None
model_lock = threading.Lock()


def load_model(path: str) -> Any:
    # 启动时加载模型、配置、相机、SDK 等重资源
    return 


def parse_input(body: bytes, request: Request) -> Any:
    # 把 HTTP 请求转换成你的业务输入
    # 例如：JSON、图片、二进制、文本、传感器数据
    if not body:
        raise ValueError("empty request body")
    return body


def run_business_logic(input_data: Any, threshold: float) -> Any:
    # 真正的核心业务逻辑
    # 例如：模型推理、路径规划、OCR、检测、分类
    global model

    if model is None:
        raise RuntimeError("model is not loaded")

    with model_lock:
        result = model(input_data, threshold=threshold)

    return result


def serialize_output(result: Any, elapsed_ms: float) -> dict[str, Any]:
    # 把业务结果转换成 JSON 可返回的数据
    return {
        "ok": True,
        "elapsed_ms": round(elapsed_ms, 3),
        "result": result,
    }


@app.get("/health")
def health() -> dict[str, Any]:
    return {"ok": model is not None}


@app.post("/run")
async def run(request: Request, threshold: float = 0.5) -> dict[str, Any]:
    request_t0 = time.perf_counter()

    try:
        body = await request.body()
        input_data = parse_input(body, request)
    except ValueError as exc:
        raise HTTPException(status_code=400, detail=str(exc)) from exc

    try:
        result = run_business_logic(input_data, threshold)
        elapsed_ms = (time.perf_counter() - request_t0) * 1000
        return serialize_output(result, elapsed_ms)
    except RuntimeError as exc:
        raise HTTPException(status_code=503, detail=str(exc)) from exc
    except Exception as exc:
        raise HTTPException(status_code=500, detail=str(exc)) from exc


def main() -> None:
    parser = argparse.ArgumentParser(description="Run my HTTP service.")
    parser.add_argument("--model", required=True, help="Model/config/resource path.")
    parser.add_argument("--host", default="127.0.0.1")
    parser.add_argument("--port", type=int, default=8000)
    args = parser.parse_args()

    global model
    model = load_model(args.model)

    uvicorn.run(app, host=args.host, port=args.port)


if __name__ == "__main__":
    main()
```