---
title: 我的网安实训第一周
published: 2026-10-10
description: 第一周接触了 DVWA 靶场、邮件原理和博客搭建，记下真正卡住我的地方和解决过程。
image: ''
tags: [网安实训, DVWA, 学习笔记]
category: 学习笔记
draft: false
lang: ''
slug: week-one
---

这周是网安实训的第一周。接触的东西比想象中多，也比想象中乱 —— 靶场、邮件协议、建站全挤在一起。趁着还记得，把过程和卡点写下来。

## 这周碰了什么

- **DVWA 靶场**：目标是搭起来，并跑通里面的漏洞模块（SQL 注入、XSS、命令注入这些）
- **邮箱的工作原理**：SMTP 发信、POP3 / IMAP 收信，还有 MX 记录、SPF / DKIM / DMARC 这些听起来很唬人的名字
- **搭一个自己的博客**：用 Astro + Firefly 主题，托管到 Cloudflare Pages 上

## 印象最深的一次排错：DVWA 报错 2054

靶场装好之后访问首页，直接甩给我一句话：

```
Database error: Cannot connect to the database (error 2054)
```

一开始我以为是密码错了，改了好几遍 root 密码都没用。后来才想明白：**问题不在密码，在认证方式。**

我的机器上有两套 MySQL：

| 数据库 | 端口 | 认证方式 |
| --- | --- | --- |
| 我自己装的 MySQL 8 | 3306 | caching_sha2_password |
| phpStudy 自带的 MySQL 5.7 | 3307 | mysql_native_password |

而 DVWA 跑在 phpStudy 的 PHP 7.3 上，它的 mysqlnd 驱动**不认识 MySQL 8 的 caching_sha2_password**，所以怎么改密码都连不上。

**解决办法**：把 DVWA 的数据库配置从 3306 改到 3307，指向 phpStudy 那套 5.7 —— 改一个端口号就通了。

```php
// config/config.inc.php
$_DVWA[ 'db_port' ] = '3307';
```

这次让我记住两件事：

1. **报错号比报错描述有用** —— 2054 说明的是"认证插件不被支持"，不是"密码错"
2. **排错前先看环境** —— 机器上装了两套数据库，DVWA 连的到底是哪一套？

## 卡住的另一次：代码克隆不下来

`git clone` 一直卡在 0%，最后报连接超时。折腾半天才确认是**网络对 github.com 的干扰**（DNS 解析被污染 + 连接被重置），不是仓库的问题。

最后的解决方式有两条路：给 git 单独指定一个可用的 GitHub IP，或者开启代理后让 git 走代理端口。

## 下周想搞明白的

- DVWA 里的漏洞，能不能不照教程、自己写一遍利用过程
- SQL 注入的 payload 为什么长那样，数据库内部到底怎么执行的
- 邮件的 SPF / DKIM 到底怎么验签

写下来才发现，这一周真正学到的东西，基本都是**报错教会我的**。
