---
title: git 操作
date: 2020-4-6
tags: conda
categories: notes
photos:
    - /blog/img/git.jpg
---

<br>

<!--more-->

# 提交 pull request 的操作

1. Fork 代码

2. 创建自己的分支

```bash
git checkout -b feat/xxx
```

3. 提交修改

```bash
git commit -am 'feat(function): add xxx'
```

4. 推送分支

```bash
git push origin feat/xxx
```

5. 提交 `pull request`

# 提交的一些规范

- feat 增加新功能
- fix 修复问题/BUG
- style 代码风格相关无影响运行结果的
- perf 优化/性能提升
- refactor 重构
- revert 撤销修改
- test 测试相关
- docs 文档/注释
- chore 依赖更新/脚手架配置修改等
- workflow 工作流改进
- ci 持续集成
- types 类型定义文件更改
- wip 开发中