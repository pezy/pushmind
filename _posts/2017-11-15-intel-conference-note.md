---
layout: post
title: 2017 Intel China AI 大会简记
categories: [conference, note]
description: Intel AI conference
keywords: AI, Intel
---

- 上午百度分享了它们的分布式深度学习平台: [PaddlePaddle](http://www.paddlepaddle.org/). 应该算是国内稍微全面一点的框架了.
- [Intel Nervana](https://www.intelnervana.com/) Intel 的人工智能处理器.
- 科大讯飞分享了它们的人工智能平台: [讯飞开放平台](http://www.xfyun.cn/)
- [MKL](https://software.intel.com/en-us/mkl) 的强大有目共睹, 虽然可以免费使用二进制包, 但看不到源码, 依旧隔靴搔痒. [MKL-DNN](https://01.org/mkl-dnn) 在某种程度上弥补了这个遗憾.
- 针对 [Nervana](https://www.intelnervana.com/) 的深度学习框架: [neon](https://github.com/NervanaSystems/neon).
- 提到了对 [torch](http://torch.ch/) 的支持.
- Intel 提供的 AI 开发云平台: [Intel AI DevCloud](https://software.intel.com/en-us/ai-academy/tools/devcloud)
- 提到一家名为 [Movidius](https://www.movidius.com/) 的公司. 刚刚与 Google 有合作.
- 提到了微软的 [brainwave](https://www.microsoft.com/en-us/research/blog/microsoft-unveils-project-brainwave/) 项目, 基于 Intel 新一代的 Stratix 10 FPGA.
- 美团分享了关于AI方面的实践, [黄金眼](http://hjy.meituan.com/)比较有意思.
- Intel 分享了深度学习在大数据上的运用: [BigDL](https://github.com/intel-analytics/BigDL) - 针对 Spark 的分布式深度学习库.
- 分享了一些论文, 其中有一篇是: [HoloNet: towards robust emotion recognition in the wild](https://dl.acm.org/citation.cfm?id=2993148.2997639).
- 提到了 [ImageNet](http://www.image-net.org/) 相关内容.
- Intel AI 大学: <https://software.intel.com/zh-cn/ai-academy>
- [Caffe 的 Intel Fork](https://github.com/intel/caffe)
- Intel 的在 ImageNet 上的训练性能报告: <https://blog.surf.nl/en/imagenet-1k-training-on-intel-xeon-phi-in-less-than-40-minutes/>
- 如何在 [Tensorflow](https://www.tensorflow.org/) 中使用 Intel MKL: 需要自行编译 CPU 版本的 Tensorflow, 步骤详见: <https://software.intel.com/en-us/articles/build-and-install-tensorflow-on-intel-architecture>
- 分享了 Tensorflow 中如何通过调参提高性能, 其中提到用 NCWH 代替默认的 NHWC, 以及在 OPA 上使用 verbs 协议.
- [Intel 定制版 Python](https://software.intel.com/en-us/distribution-for-python) 将极大的提升性能(近20倍).