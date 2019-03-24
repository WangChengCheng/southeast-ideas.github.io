---
layout: post
title:  Modify Git User
date:   2019-03-24 17:08:00 +0800
categories: Project
tag: Project
author: Michael Wang
source_id: 190324170800
---

* content
{:toc}

查看当前项目的git user
```bash
cd project_dir
git config user.name
git config user.email

```

修改当前项目的git user
```bash
cd project_dir
git config user.name "username"
git config user.email "example@example.com"

```

修改全局git user
```bash
git config --global user.name "username"
git config --global user.email "example@example.com"

```