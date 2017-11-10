---
layout: post
title: 自动领取 V2EX 每日登录奖励
categories: [py, gist]
description: v2ex mission selenium python
keywords: v2ex auto
---

下半年来, 最经常上的网站就是 [V2EX](https://www.v2ex.com/) 了, 每天登录都习惯性的先领取一下"每日登录奖励", 不知不觉都连续一百多天了. 还蛮有成就感的.

恰逢最近想抽空把 [Automate The boring stuff with Python](https://automatetheboringstuff.com) 扫完, 看到爬虫那一章, 讲到如何用 selenium 模拟操控网页. 灵光一闪决定用 V2EX 这个每日奖励来练个手.

遇到的第一个难题是验证码, 其实 V2EX 的验证码算是清晰简单的了, 可以考虑用 tesseract-ocr 来解决掉. 后来觉得这有点偏离主题了, 而且登录这种事并非很频繁, 登录过一次, 就可以借用 cookies 反复登录了.

于是我开始找利用 cookies 登录的方案, 首先就是如何拿到 cookies. 通过 Chrome 的 F12 似乎也很容易就拿到 cookies 字符串, 但问题是怎么把这玩意传给 selenium 的 driver?

查阅 selenium 的文档, 发现 `add_cookie` 和 `get_cookies` 的存在, 于是想到一个最笨的方法: 先手动登录一次, 把 cookies 记录下来, 然后后续的登录直接加载该 cookies 即可.

当然得给手动输入预留一点时间, 我在代码里 `sleep(30)`, 应该足够了.

剩下的工作就比较简单了, 找到关键按钮的特征, 可以是 class name, 可以是 id, 甚至可以是 HTML content, selenium 都可以为你定位到目标 tag 上.

代码请见: <https://gist.github.com/pezy/28344f157f97a1d1569a3c6803c198e4>

直接运行, 第一次会跳转到登录界面, 填写登录信息以及验证码. 然后再次运行一次, 就可以自动领取奖励了. 只要 cookies 持续时间够长, 每天点一下该脚本就可以了.

虽然这本书越读越没劲, 但这个小练习还挺好玩, 以后可以开发更多类似的应用, 节约大家的时间.