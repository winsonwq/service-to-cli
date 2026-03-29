# CLI 框架对比

## 推荐：Click（Python）

最成熟、文档最完善，AI agent 场景下最常用。

```bash
pip install click
```

```python
import click
import json

@click.group()
def cli():
    """Agent-native CLI wrapper."""
    pass

@cli.command()
@click.option("--name", required=True, help="用户名，自解释参数")
@click.option("--expires-at", help="过期时间，ISO 格式")
def onboard_sale(name, expires_at):
    """新销售入职任务。幂等：重复执行返回已有数据。"""
    # 幂等逻辑：检查是否已存在
    existing = get_sale(name)
    if existing:
        print(json.dumps({"status": "ok", "data": existing}))
        return
    
    # 执行业务逻辑
    result = create_sale(name, expires_at)
    print(json.dumps({"status": "ok", "data": result}))

if __name__ == "__main__":
    cli()
```

**输出格式约定**：
```json
// 成功
{"status": "ok", "data": {...}}

// 失败（exit code 非 0）
{"status": "error", "message": "具体错误"}
```

## 其他选择

| 框架 | 语言 | 适用场景 |
|------|------|---------|
| **Click** | Python | 最推荐，生态最完整 |
| **Typer** | Python | Click 的进阶版，基于 type hint |
| **oclif** | Node.js | 如果系统用 JS/TS |
| **Cobra** | Go | 如果系统用 Go |
| **Argparse** | Python | 内置，但不如 Click 方便 |

## Subprocess 安全调用

如果 CLI 包装的是另一个可执行文件：

```python
import subprocess
import json

def run_external(cmd: list, timeout=120) -> dict:
    result = subprocess.run(
        cmd,
        capture_output=True,
        text=True,
        timeout=timeout
    )
    if result.returncode == 0:
        try:
            return {"status": "ok", "data": json.loads(result.stdout)}
        except json.JSONDecodeError:
            return {"status": "ok", "data": result.stdout}
    else:
        return {"status": "error", "message": result.stderr}
```

**原则**：
- 始终 `capture_output=True`
- 通过 stdin 传递脚本（不写临时文件）
- 设置 timeout 防止阻塞
