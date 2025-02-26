---
title: Python包管理-Pip与Conda的用法
date: 2024-09-04
tags:
  - Python
categories:
  - Python
description: pip和conda的基本操作，也包含了换源、导出环境和打包虚拟环境等常用操作的说明
---

@[TOC]

python的包管理工具一般是`pip`和`conda`

`pip`和`conda`是Python开发环境中广泛使用的两大包管理工具。
`conda`是Anaconda发行版的一部分，但也可以单独安装使用，适用于所有Python环境，不仅仅是Anaconda用户。

这个博客是平时的笔记的一次整合，至于为什么连这些都有笔记，那只能是：


为了更好展现使用方法，就不分为`pip`和`conda`两部分展示。
而是**依照使用**进行区分展示。

---
## 0. 获取帮助页面
对于任何命令行工具，查阅其帮助文档通常是快速熟悉和掌握其用法的有效途径。
`pip`和`conda`可分别通过以下命令访问其帮助页面：
```bash
pip help
pip --help
# pip可以通过两种来获取帮助

conda --help
# conda只能--help
```
非常推荐在日常使用中去随时查看help文档

---
## 1. 基础操作

### 安装包

安装包的基本命令如下：
```bash
pip install <package_name>

conda install <package_name>
```
指定特定版本安装：
```bash
pip install numpy==1.21.5

conda install numpy=1.21.5
```

#### 换源
- pip
更换`pip`的默认源以加速下载，如使用清华大学镜像：
```bash
pip install numpy -i https://pypi.tuna.tsinghua.edu.cn/simple
```
使用`-i`的方法是临时性的，并不是永久的

- conda 
conda换源和切换至特定通道安装：
```bash
conda install numpy -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main

conda install conda-pack -c conda-forge
```

> **注**：“源”（`pip`中的`--index-url`或`-i`选项）与“通道”（`conda`中的`-c`或`--channel`选项）尽管在功能上都是指代软件包的来源，但它们分别对应于`pip`与`conda`各自独立的服务体系（PyPI与Anaconda Cloud）及其特有的命令行参数。

#### 国内镜像源
同时，在指定源进行安装的时候，为了避免一些安全性问题，建议选择常规可信源进行安装。

||一些常见的源|
|--|--|
|清华：|https://pypi.tuna.tsinghua.edu.cn/simple|
|阿里云|http://mirrors.aliyun.com/pypi/simple/|
|中科大|https://pypi.mirrors.ustc.edu.cn/simple/|
|豆瓣|http://pypi.douban.com/simple/|

### 更新
更新单个包：
```bash
pip install --upgrade <package_name>

conda update <package_name>
```
更新所有已安装包：
```bash
pip install --upgrade --all

conda update --all
```
### 卸载
```bash
pip uninstall <package_name>

conda remove <package_name>
```
### 查看
查看单一包的详情
```bash
pip show <package_name>

conda list <package_name>
```
查看所有已安装的包
```bash
pip list

conda list
```
---
## 2. requirements
### 依据requirements.txt安装
```bash
pip install -r requirements.txt
pip install --requirement requirements.txt

conda install --yes --file requirements.txt
conda install -y -f requirements.txt
```
### 生成requirements.txt
使用`pip`自带功能生成：
```bash
pip freeze > requirements.txt
# 自带的功能，会把所有包都生成出来
```
**使用第三方工具`pipreqs`生成：**(推荐)
```bash
pip install pipreqs

pipreqs . 
# 对当前路径下的文件进行requirements生成
pipreqs . --encoding=utf8
# 加上这个避免一些编码错误
```

<a id="RecommendPipreqs"></a>
### freeze和pipreqs的差别
`pip freeze` 用于列出当前 Python 环境中**所有已安装包及其版本**。
执行此命令时，它不分项目边界，一视同仁地捕捉环境中所有的软件包，无论这些包是否直接服务于当前项目。
也导致生成的 `requirements.txt` 文件可能**包含大量非项目必需的依赖**，提供的是全局环境快照，依赖列表过于宽泛。

`pipreqs` 是一款专门**针对项目**代码进行依赖分析的工具。
它通过静态解析项目目录下的源代码和相关文件（如 `.py`, `.ipynb`, `.yaml` 等），识别出项目实际引用的库。
基于这种精准分析，生成的 `requirements.txt` 文件**仅包含项目实际使用的依赖**，剔除了无关包。所以通常更推荐使用 `pipreqs`。

---
## 3. conda环境
- 创建
```bash
conda create -n New_env python=3.7
conda create --name New_env python=3.7
```
- 重命名env
```bash
conda create --name conda-new --clone conda-old
# 通过克隆进行重命名
```
- 查看环境
```bash
conda env list
conda info --envs
```
- 激活与退出
```bash
conda activate <env_name>

conda deactivate
```
- 删除环境
```bash
conda env remove --name <env_name>
```

### 环境迁移
- Export环境配置进行迁移
```bash
# 导出环境配置
conda env export > enviroment.yml
# 依据环境配置创建环境
conda env create -f enviroment.yml
```

- 打包环境进行迁移
```bash
conda install conda-pack -c conda-forge
# 从通道conda-forge安装conda-pack工具

conda pack -n <env_name> -o /path/to/<output_filename>.tar.gz
```
eg.
```bash
conda pack -n mask-rcnn -o /pack_envs/mask-rcnn.tar.gz
# 将环境存储到路径为<>的文件中
```

- 导入打包好的环境
将打包文件移动至目标路径，并解压。
假设目标路径为`$CondaPath/<env_name>`，操作如下：
```bash
cd $CondaPath
mkdir <env_name>
cd <env_name>
tar -xzvf /path/to/<pack_filename>.tar.gz
```
激活并验证已导入的环境：
```bash
conda env list
conda activate <env_name>
```

**注意**：`$CondaPath`代表conda环境目录，通常位于`/root/anaconda/envs`（或其他路径），具体位置可通过`conda info`命令查看。

---
## 更新说明
**2024年4月20日**
- 补充了freeze和pipreqs的区别，说明为什么更推荐pipreqs。[跳转](#RecommendPipreqs)

