---
title: conda管理命令
date: 2020-4-6
tags: conda
categories: notes
photos:
    - /blog/img/conda.jpg
---

<br>

<!--more-->

> 以下均针对于python3

# conda管理虚拟环境

```
# 创建虚拟环境
conda create -n python3 python=3.7.6

# 退出当前虚拟环境
conda deactivate

# 查看当前所有虚拟环境
conda info --envs
或者
conda env list

# 进入指定虚拟环境
conda activate 环境

# 安装python库
conda install 库1 库2

# 安装库到指定环境
conda install -n 环境 库

# 安装指定版本库到指定环境
conda install -n 环境 库=版本号

# 更新库版本
conda update 库

# 移除库
conda remove -n 环境 库

# 删除虚拟环境
conda remove -n 环境 --all

# 删除虚拟环境的库
conda remove --name $环境 $库

```