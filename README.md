# HPDIC MOD

```bash
# 安装 SQLite3 and SQLiteCpp
cd
sudo apt install libsqlite3-dev -y
git clone https://github.com/SRombauts/SQLiteCpp.git
cd SQLiteCpp
mkdir build
cd build
# -DSQLITECPP_INTERNAL_SQLITE=OFF 表示使用系统刚才装的 libsqlite3
cmake -DSQLITECPP_INTERNAL_SQLITE=OFF ..
sudo make install

# 安装 other dependencies
cd
sudo apt install -y libantlr4-runtime-dev
sudo apt install -y protobuf-compiler libprotobuf-dev libgrpc++-dev protobuf-compiler-grpc
sudo apt install -y libtbb-dev
sudo apt install nlohmann-json3-dev
sudo apt install -y libboost-all-dev
sudo apt install -y libgflags-dev libgoogle-glog-dev libopenblas-dev libfmt-dev libzstd-dev libssl-dev
sudo apt install -y libxsimd-dev libmarisa-dev libopenblas-dev libgtest-dev
sudo apt install -y libyaml-cpp-dev
sudo apt install rapidjson-dev
sudo apt install -y libroaring-dev
sudo apt install libcurl4-openssl-dev

# 安装 libevent
cd ~
git clone https://github.com/libevent/libevent.git
cd libevent
git checkout release-2.1.12-stable
mkdir build && cd build
cmake .. 
make -j$(nproc)
sudo make install

# 安装 folly
sudo apt install -y \
    libdouble-conversion-dev \
    libgoogle-glog-dev \
    libgflags-dev \
    libfmt-dev \
    libboost-all-dev \
    libevent-dev \
    libiberty-dev \
    liblz4-dev \
    liblzma-dev \
    libsnappy-dev \
    make \
    zlib1g-dev \
    binutils-dev \
    libjemalloc-dev \
    libssl-dev \
    pkg-config \
    libunwind-dev
cd ~
git clone https://github.com/facebook/folly.git
cd folly
git checkout v2024.01.01.00
mkdir _build && cd _build
# -DFOLLY_USE_JEMALLOC=OFF: 有时候 jemalloc 会导致链接错误，关掉比较稳
# -DCMAKE_POSITION_INDEPENDENT_CODE=ON: 必须开，否则后面链接动态库会报错
cmake .. -DCMAKE_CXX_STANDARD=17 -DCMAKE_POSITION_INDEPENDENT_CODE=ON -DFOLLY_USE_JEMALLOC=OFF
make -j$(nproc)
sudo make install

# 安装 Apache Arrow C++库
cd ~
git clone https://github.com/apache/arrow.git
cd arrow
git checkout apache-arrow-15.0.0  # 选一个较新的稳定版，Milvus通常兼容12.0+
cd cpp
mkdir build && cd build
cmake .. \
  -DARROW_PARQUET=ON \
  -DARROW_COMPUTE=ON \
  -DARROW_CSV=ON \
  -DARROW_FILESYSTEM=ON \
  -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)
sudo make install

# 安装 Prometheus C++库
cd ~
git clone --recursive https://github.com/jupp0r/prometheus-cpp.git
cd prometheus-cpp
mkdir _build && cd _build
# -DENABLE_PUSH=OFF: Milvus Lite 通常只需要 Pull 模式，关掉 Push 可以少依赖一个 curl
# -DENABLE_TESTING=OFF: 关掉测试省时间
cmake .. -DENABLE_TESTING=OFF
make -j$(nproc)
sudo make install

# 安装 marisa-trie
cd ~
git clone https://github.com/s-yata/marisa-trie.git
cd marisa-trie
mkdir build && cd build
cmake .. 
make -j$(nproc)
sudo make install
sudo mkdir -p /usr/local/lib/cmake/marisa
sudo vim /usr/local/lib/cmake/marisa/marisa-config.cmake
# 在文件中添加以下内容
# Minimal marisa config
set(MARISA_INCLUDE_DIRS "/usr/local/include")
set(MARISA_LIBRARIES "/usr/local/lib/libmarisa.a")
set(marisa_FOUND TRUE)
add_library(marisa::marisa STATIC IMPORTED)
set_target_properties(marisa::marisa PROPERTIES
    IMPORTED_LOCATION "${MARISA_LIBRARIES}"
    INTERFACE_INCLUDE_DIRECTORIES "${MARISA_INCLUDE_DIRS}"
)

# 编译 milvus-lite
cd
sudo apt install cmake -y
cd ~/milvus-lite
git submodule update --init --recursive
mkdir build && cd build
cmake .. -DBUILD_TESTING=OFF
make -j$(nproc)
sudo make install
```
---

<div align="center">
    <img src="https://raw.githubusercontent.com/milvus-io/milvus-lite/refs/heads/main/milvus_lite_logo.png#gh-light-mode-only" width="60%"/>
</div>

<h3 align="center">
    <p style="text-align: center;"> <span style="font-weight: bold; font: Arial, sans-serif;"></span>A lightweight version of Milvus</p>
</h3>

<div class="column" align="middle">
  <a href="https://www.apache.org/licenses/LICENSE-2.0">
    <img src="https://img.shields.io/badge/license-apache2.0-green?style=flat" alt="license"/>
  </a>
  <a href="https://pypi.org/project/milvus-lite/">
    <img src="https://img.shields.io/pypi/v/milvus-lite?label=Release&color&logo=Python" alt="github actions">
  </a>
    <a href="https://pypi.org/project/milvus-lite/">
    <img src="https://img.shields.io/pypi/dm/milvus-lite.svg?color=bright-green&logo=Pypi" alt="github actions">
  </a>
</div>


# Introduction
Milvus Lite is the lightweight version of [Milvus](https://github.com/milvus-io/milvus), a high-performance vector database that powers AI applications with vector similarity search. This repo contains the core components of Milvus Lite. 

With Milvus Lite, you can start building an AI application with vector similarity search within minutes! Milvus Lite is good for running in the following environment:
- Jupyter Notebook / Google Colab
- Laptops
- Edge Devices

Milvus Lite can be imported into your Python application, providing the core vector search functionality of Milvus. Milvus Lite is already included in the [Python SDK of Milvus](https://github.com/milvus-io/pymilvus). To use it, you just need `pip install pymilvus`. 

Milvus Lite uses the same API as Milvus Standalone and Distributed, providing a consistent experience across environments. Develop your GenAI applications once and run them anywhere: on a laptop or Jupyter Notebook with Milvus Lite, in a Docker container with Milvus Standalone, or on a K8s cluster with Milvus Distributed for large-scale production.

<div align="center">
    <img src="https://milvus.io/docs/v2.5.x/assets/select-deployment-option.png" width="80%"/>
</div>

Milvus Lite is only suitable for small scale prototyping (usually less than a million vectors) or edge devices. For large scale production, we recommend using [Milvus Standalone](https://milvus.io/docs/install-overview.md#Milvus-Standalone) or [Milvus Distributed](https://milvus.io/docs/install-overview.md#Milvus-Distributed). You can also consider the fully-managed Milvus on [Zilliz Cloud](https://zilliz.com/cloud).

# Requirements
Milvus Lite currently supports the following environments:
- Ubuntu >= 20.04 (x86_64 and arm64)
- MacOS >= 11.0 (Apple Silicon M1/M2 and x86_64)

***Note:*** Windows is not yet supported.

# Installation
```shell
pip install -U pymilvus[milvus-lite]
```
We recommend using `pymilvus`. You can `pip install` with `-U` to force update to the latest version and `milvus-lite` will be automatically installed.

If you want to explicitly install `milvus-lite` package, or you have installed an older version of `milvus-lite` and would like to update it, you can do `pip install -U milvus-lite`.

# Usage
In `pymilvus`, specify a local file name as uri parameter of MilvusClient will use Milvus Lite.
```python
from pymilvus import MilvusClient
client = MilvusClient("./milvus_demo.db")
```

> **_NOTE:_**  Note that the same API also applies to Milvus Standalone, Milvus Distributed and Zilliz Cloud, the only difference is to replace local file name to remote server endpoint and credentials, e.g. 
`client = MilvusClient(uri="http://localhost:19530", token="username:password")` for self-hosted Milvus server.

# Examples
Following is a simple demo showing how to use Milvus Lite for text search. There are more comprehensive [examples](https://github.com/milvus-io/bootcamp/tree/master/bootcamp/tutorials) for using Milvus Lite to build applications
such as [RAG](https://github.com/milvus-io/bootcamp/blob/master/bootcamp/tutorials/quickstart/build_RAG_with_milvus.ipynb), [image search](https://github.com/milvus-io/bootcamp/blob/master/bootcamp/tutorials/quickstart/image_search_with_milvus.ipynb), and using Milvus Lite in popular RAG framework such as [LangChain](https://github.com/milvus-io/bootcamp/blob/master/bootcamp/tutorials/integration/rag_with_milvus_and_langchain.ipynb) and [LlamaIndex](https://github.com/milvus-io/bootcamp/blob/master/bootcamp/tutorials/integration/rag_with_milvus_and_llamaindex.ipynb)!

```python
from pymilvus import MilvusClient
import numpy as np

client = MilvusClient("./milvus_demo.db")
client.create_collection(
    collection_name="demo_collection",
    dimension=384  # The vectors we will use in this demo has 384 dimensions
)

# Text strings to search from.
docs = [
    "Artificial intelligence was founded as an academic discipline in 1956.",
    "Alan Turing was the first person to conduct substantial research in AI.",
    "Born in Maida Vale, London, Turing was raised in southern England.",
]
# For illustration, here we use fake vectors with random numbers (384 dimension).

vectors = [[ np.random.uniform(-1, 1) for _ in range(384) ] for _ in range(len(docs)) ]
data = [ {"id": i, "vector": vectors[i], "text": docs[i], "subject": "history"} for i in range(len(vectors)) ]
res = client.insert(
    collection_name="demo_collection",
    data=data
)

# This will exclude any text in "history" subject despite close to the query vector.
res = client.search(
    collection_name="demo_collection",
    data=[vectors[0]],
    filter="subject == 'history'",
    limit=2,
    output_fields=["text", "subject"],
)
print(res)

# a query that retrieves all entities matching filter expressions.
res = client.query(
    collection_name="demo_collection",
    filter="subject == 'history'",
    output_fields=["text", "subject"],
)
print(res)

# delete
res = client.delete(
    collection_name="demo_collection",
    filter="subject == 'history'",
)
print(res)
```

# Supported Features

## Functionality
Milvus Lite shares the same API as Milvus Standalone, Milvus Distributed and Zilliz Cloud, offering core features including:
- Insert/upsert operations
- Vector data persistence and collection management
- Dense, sparse, and hybrid vector search
- Metadata filtering
- Multi-vector support 

## Index Type
Milvus Lite has limited index type support compared to other Milvus deployments:

- Prior to version 2.4.11:
  - Only supports the [FLAT](https://milvus.io/docs/index.md?tab=floating#FLAT) index type
  - Uses FLAT index regardless of the specified index type in collection creation

- Version 2.4.11 and later:
  - Supports both [FLAT](https://milvus.io/docs/index.md?tab=floating#FLAT) and [IVF_FLAT](https://milvus.io/docs/index.md?tab=floating#IVFFLAT) index types
  - For IVF_FLAT indexes:
    - Data size < 100,000: Automatically uses FLAT index internally for better performance
    - Data size ≥ 100,000: Constructs and uses IVF_FLAT index as specified

# Known Limitations

Milvus Lite does not support partitions, users/roles/RBAC, alias. To use those features, please choose other Milvus deployment types such as [Standalone](https://milvus.io/docs/install-overview.md#Milvus-Standalone), [Distributed](https://milvus.io/docs/install-overview.md#Milvus-Distributed) or [Zilliz Cloud](https://zilliz.com/cloud) (fully-managed Milvus).

# Migrating data from Milvus Lite

All data stored in Milvus Lite can be easily exported and loaded into other types of Milvus deployment, such as Milvus Standalone on Docker, Milvus Distributed on K8s, or fully-managed Milvus on [Zilliz Cloud](https://zilliz.com/cloud).

Milvus Lite provides a command line tool that can dump data into a json file, which can be imported into self-hosted [Milvus](https://github.com/milvus-io/milvus) or fully-managed Milvus on [Zilliz Cloud](https://zilliz.com/cloud). 

```shell
pip install -U "pymilvus[bulk_writer]"
# The `milvus-lite` command line tool is already included in `milvus-lite` package which is part of "pymilvus", but it also needs some dependencies from `pymilvus[bulk_writer]` for dumping data.

milvus-lite dump -h

usage: milvus-lite dump [-h] [-d DB_FILE] [-c COLLECTION] [-p PATH]

optional arguments:
  -h, --help            show this help message and exit
  -d DB_FILE, --db-file DB_FILE
                        milvus lite db file
  -c COLLECTION, --collection COLLECTION
                        collection that need to be dumped
  -p PATH, --path PATH  dump file storage dir
```
The following example dumps all data from `demo_collection` collection that's stored in `./milvus_demo.db` (a user specified local file that persists data for Milvus Lite)

To export data:

```shell
milvus-lite dump -d ./milvus_demo.db -c demo_collection -p ./data_dir
# ./milvus_demo.db: milvus lite db file
# demo_collection: collection that need to be dumped
#./data_dir : dump file storage dir
```

You can use the dump file as input to upload data to Zilliz Cloud via [Data Import](https://docs.zilliz.com/docs/data-import), or a self-hosted Milvus server via [Bulk Insert](https://milvus.io/docs/import-data.md).
# Contributing
If you want to contribute to Milvus Lite, please read the [Contributing Guide](https://github.com/milvus-io/milvus-lite/blob/main/CONTRIBUTING.md) first.

# Report a bug
For any bug or feature request, please report it by submitting an issue in [milvus-lite](https://github.com/milvus-io/milvus-lite/issues/new/choose) repo.

# License
Milvus Lite is under the Apache 2.0 license. See the [LICENSE](https://github.com/milvus-io/milvus-lite/blob/main/LICENSE) file for details.
