+++
title = "Podman 部署 Open WebUI"
date = "2025-09-05"
[taxonomies]
categories = ["AI"]
tags = ["Open WebUI", "podman", "ollama"]
+++

Podman 部署 [Open WebUI](https://github.com/open-webui/open-webui)，并支持 Nvidia GPU，能够和本地安装的 ollama 通信。

<!-- more -->

## Podman 部署 Open WebUI
> Open WebUI 的 Github 主页提供了[基于 Docker 的部署方式，并支持 Nvidia GPU](https://github.com/open-webui/open-webui?tab=readme-ov-file#installation-with-default-configuration)：

```bash
docker run -d -p 3000:8080 --gpus all --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:cuda
```

针对 Podman 有以下几处修改：

1. 安装 [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)：

```bash
# Configure the production repository
curl -s -L https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo | sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo

# Install the NVIDIA Container Toolkit packages:
export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.17.8-1
  sudo dnf install -y \
      nvidia-container-toolkit-base-${NVIDIA_CONTAINER_TOOLKIT_VERSION}
```

这里我只安装了 ``nvidia-container-toolkit-base``。主要原因官方推荐通过 [CDI](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/cdi-support.html) 的方式使用 Nvidia GPU，而 CDI 的文档中有如下描述：

> You installed either the NVIDIA Container Toolkit or you installed the ``nvidia-container-toolkit-base`` package. The base package includes the container runtime and the ``nvidia-ctk`` command-line interface, but avoids installing the container runtime hook and transitive dependencies. The hook and dependencies are not needed on machines that use CDI exclusively.

这里也同步附上原始的安装命令作为对比：

```bash
# Install the NVIDIA Container Toolkit packages:
export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.17.8-1
  sudo dnf install -y \
      nvidia-container-toolkit-${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      nvidia-container-toolkit-base-${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      libnvidia-container-tools-${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      libnvidia-container1-${NVIDIA_CONTAINER_TOOLKIT_VERSION}
```

2. 配置 CDI：

```bash
# Generate the CDI specification file
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml

# (Optional) Check the names of the generated devices
nvidia-ctk cdi list
```

3. 验证上述步骤

```bash
podman run --rm --device nvidia.com/gpu=all --security-opt=label=disable ubuntu nvidia-smi -L
```

上述命令输出，应该和在本机直接执行 ``nvidia-smi -L`` 一致。

4. 修正后的 Podman 启动 Open WebUI 如下：

```bash
podman run -d -p 3000:8080 --device nvidia.com/gpu=all --security-opt=label=disable --add-host=host.containers.internal:host-gateway -v $HOME/open-webui:/app/backend/data --name open-webui --restart always --replace ghcr.io/open-webui/open-webui:cuda
```

查看启动日志：

```txt
❯ podman logs open-webui 
Loading WEBUI_SECRET_KEY from file, not provided as an environment variable.
Generating WEBUI_SECRET_KEY
Loading WEBUI_SECRET_KEY from .webui_secret_key
CUDA is enabled, appending LD_LIBRARY_PATH to include torch/cudnn & cublas libraries.
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> 7e5b5dc7342b, init
INFO  [alembic.runtime.migration] Running upgrade 7e5b5dc7342b -> ca81bd47c050, Add config table
INFO  [alembic.runtime.migration] Running upgrade ca81bd47c050 -> c0fbf31ca0db, Update file table
INFO  [alembic.runtime.migration] Running upgrade c0fbf31ca0db -> 6a39f3d8e55c, Add knowledge table
INFO  [alembic.runtime.migration] Running upgrade 6a39f3d8e55c -> 242a2047eae0, Update chat table
INFO  [alembic.runtime.migration] Running upgrade 242a2047eae0 -> 1af9b942657b, Migrate tags
INFO  [alembic.runtime.migration] Running upgrade 1af9b942657b -> 3ab32c4b8f59, Update tags
INFO  [alembic.runtime.migration] Running upgrade 3ab32c4b8f59 -> c69f45358db4, Add folder table
INFO  [alembic.runtime.migration] Running upgrade c69f45358db4 -> c29facfe716b, Update file table path
INFO  [alembic.runtime.migration] Running upgrade c29facfe716b -> af906e964978, Add feedback table
INFO  [alembic.runtime.migration] Running upgrade af906e964978 -> 4ace53fd72c8, Update folder table and change DateTime to BigInteger for timestamp fields
INFO  [alembic.runtime.migration] Running upgrade 4ace53fd72c8 -> 922e7a387820, Add group table
INFO  [alembic.runtime.migration] Running upgrade 922e7a387820 -> 57c599a3cb57, Add channel table
INFO  [alembic.runtime.migration] Running upgrade 57c599a3cb57 -> 7826ab40b532, Update file table
INFO  [alembic.runtime.migration] Running upgrade 7826ab40b532 -> 3781e22d8b01, Update message & channel tables
INFO  [alembic.runtime.migration] Running upgrade 3781e22d8b01 -> 9f0c9cd09105, Add note table
INFO  [alembic.runtime.migration] Running upgrade 9f0c9cd09105 -> d31026856c01, Update folder table data
INFO  [alembic.runtime.migration] Running upgrade d31026856c01 -> 018012973d35, Add indexes
INFO  [alembic.runtime.migration] Running upgrade 018012973d35 -> 3af16a1c9fb6, update user table
WARNI [open_webui.env] 

WARNING: CORS_ALLOW_ORIGIN IS SET TO '*' - NOT RECOMMENDED FOR PRODUCTION DEPLOYMENTS.

INFO  [open_webui.env] VECTOR_DB: chroma
INFO  [open_webui.env] Embedding model set: sentence-transformers/all-MiniLM-L6-v2
WARNI [langchain_community.utils.user_agent] USER_AGENT environment variable not set, consider setting it to identify your requests.
Creating knowledge table
Migrating data from document table to knowledge table
Converting 'chat' column to JSON
Renaming 'chat' column to 'old_chat'
Adding new 'chat' column of type JSON
Dropping 'old_chat' column
Primary Key: {'name': None, 'constrained_columns': []}
Unique Constraints: [{'name': 'uq_id_user_id', 'column_names': ['id', 'user_id']}]
Indexes: [{'name': 'tag_id', 'column_names': ['id'], 'unique': 1, 'dialect_options': {}}]
Creating new primary key with 'id' and 'user_id'.
Dropping unique constraint: uq_id_user_id
Dropping unique index: tag_id

██████╗ ██████╗ ███████╗███╗   ██╗    ██╗    ██╗███████╗██████╗ ██╗   ██╗██╗
██╔═══██╗██╔══██╗██╔════╝████╗  ██║    ██║    ██║██╔════╝██╔══██╗██║   ██║██║
██║   ██║██████╔╝█████╗  ██╔██╗ ██║    ██║ █╗ ██║█████╗  ██████╔╝██║   ██║██║
██║   ██║██╔═══╝ ██╔══╝  ██║╚██╗██║    ██║███╗██║██╔══╝  ██╔══██╗██║   ██║██║
╚██████╔╝██║     ███████╗██║ ╚████║    ╚███╔███╔╝███████╗██████╔╝╚██████╔╝██║
╚═════╝ ╚═╝     ╚══════╝╚═╝  ╚═══╝     ╚══╝╚══╝ ╚══════╝╚═════╝  ╚═════╝ ╚═╝


v0.6.26 - building the best AI user interface.

https://github.com/open-webui/open-webui

Fetching 30 files: 100%|██████████| 30/30 [02:26<00:00,  4.87s/it]
INFO:     Started server process [1]
INFO:     Waiting for application startup.
```

可以通过访问 http://localhost:3000 正常使用 Open WebUI。

## 解决 Open WebUI 无法访问 ollama 的问题
> 这里 ollama 是本地安装，绑定端口 11434。

主要原因是容器内无法通过 http://host.containers.internal 访问本地网络，可以直接制定 --network=host。

修改 podman 启动命令：

```bash
podman run -d --network=host --device nvidia.com/gpu=all --security-opt=label=disable --add-host=host.containers.internal:host-gateway -v $HOME/open-webui:/app/backend/data --name open-webui --restart always --replace ghcr.io/open-webui/open-webui:cuda
```

通过 http://localhost:8080 访问 Open WebUI，而不是之前的 3000 端口。

登录成功后，进入【Admin Panel】 -> 【Settings】 -> 【Connections】:

- 关闭【OpenAI API】
- 修改【Ollama API】中的 URL 为 http://localhost:11434

这样就可以调用 ollama。