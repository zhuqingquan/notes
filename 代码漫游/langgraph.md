# langgraph

## 编译构建安装
```shell
git clone https://github.com/langchain-ai/langgraph.git
cd langgraph
make install
# 激活python环境
source .venv/bin/activate
```
### 代码构建安装脚本分析
使用Makefile的方式进行安装，都是python代码，所以基本不涉及到编译的过程。可以查看`langgraph/Makefile`文件。
python环境依赖的管理使用uv来完成。这个可以通过`Makefile`的install目标来确定。
```langgraph/Makefile
# Install dependencies for all projects
.PHONY: install
install:
	@echo "Creating virtual environment..."
    # 先使用uv创建虚拟环境venv
	@uv venv
    # 之后进入libs/*下面的目录，使用pip安装依赖。具体每个目录依赖的包可以查看pyproject.toml文件。
	@for dir in $(LIBS_DIRS); do \
		if [ -f $$dir/pyproject.toml ]; then \
			echo "Installing dependencies for $$dir"; \
            # 从下面命令可以看出，这里不只是安装依赖，还进行了开发模式的安装。
			uv pip install -e $$dir; \
		fi; \
	done
```