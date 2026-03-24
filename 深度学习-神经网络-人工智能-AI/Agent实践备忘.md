<!--
 * @Author: zhuqingquan zqq_222@163.com
 * @Date: 2026-02-10 
 * @FilePath: /notes/深度学习-神经网络-人工智能-AI/Agent实践备忘.md
 * @Description: 各种开源Agent技术的部署，测试或者重要知识的备忘
-->
# Dify
## 通过docker本地部署
### 设置docker
1. 修改docker的image的缓存路径。创建软连接到/var/lib/docker
```
sudo mv /var/lib/docker /data/
sudo ln -s /data/docker /var/lib/docker
```
2. 设置docker镜像。不然在国内大概率下载不到镜像。
```
sudo vim /etc/docker/daemon.json
#输入以下内容，保存
{
   "registry-mirrors": [
       "https://docker.1ms.run",
       "https://docker.xuanyuan.me",
       "https://mirror.ccs.tencentyun.com"
   ]
}

```
3. 启动docker
```
cd dify/docker
cp .env.template .env
sudo docker compose up -d
```
如果出现80端口被占用，则可以修改docker-compose.yaml中的端口映射。
```
sudo docker compose down
vim docker-compose.yaml
# 找到ports节点添加
- "8899:80"
```
4. 启动dify管理web页面
http://localhost:8899

# LangChain & LangGraph
[官方文档](https://docs.langchain.com/oss/python/langgraph)

## LangGraph
### 设计原则
When you build an agent with LangGraph, you will first break it apart into discrete steps called **nodes**. Then, you will describe the different **decisions and transitions** from each of your nodes. Finally, you connect nodes together through **a shared state** that each node can read from and write to.

### StateGraph.add_conditional_edges
添加支持条件判断的边，比如当LLM返回调用Tools时，则下一步进入ToolNode。如果LLM返回了最终结果则进入"__end__"。

### 安装
#### 代码安装
```shell
git clone https://github.com/langchain-ai/langgraph.git
cd langgraph
make install

# 安装后使用激活uv虚拟环境
source .venv/bin/activate
# 运行一个文件查看是否正常
uv python <python file>
```