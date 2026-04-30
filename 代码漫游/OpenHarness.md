## 前端交互
### ReactBackendHost
当使用React UI与Agent进行交互时，`ReactBackendHost`负责完成Agent与UI之间的通信。使用pipeline进行消息交互。

### runtime
代码文件：`src/openharness/ui/runtime.py
主要代码`build_runtime`,`RuntimeBundle`

## 调试
### React交互模式下调试
```shell
# 使用这个命令会触发启动React UI，用于交互
.venv/bin/oh --debug 
```
app.py --> run_repl --> launch_react_tui
在launch_react_tui中会使用npm环境中的node启动 `frontend/terminal/src/index.tsx`

React UI中的`frontend/terminal/src/App.tsx`会启动一个新的python进程运行Agent，启动命令：`.venv/bin/python3 -m openharness --backend-only --cwd /home/zhuqingquan/OpenHarness/`

如果需要调试agent，可以修改启动Agent的命令行：
修改`src/openharness/ui/react_launcher.py`中的launch_react_tui方法。
`.venv/bin/python3 -m debugpy --listen 5678 --wait-for-client -m openharness --backend-only --cwd /home/zhuqingquan/OpenHarness/`

然后再在vscode中添加调试配置：
```json
        {
            "name": "Python: Attach to Module",
            "type": "debugpy",
            "request": "attach",
            "connect": {
                "host": "localhost",
                "port": 5678
            }
        },
```