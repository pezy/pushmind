---
layout: post
title: Tesseract OCR 研究笔记
categories: [cv]
description: Tesseract OCR 使用的经验总结
keywords: tesseract
---

在最近一个项目里, 初识 Tesseract OCR 的威力. 虽然项目已经有了初步成效, 但仍有两个目标在心中无法释怀:

1. 它内部实现, 如何与深度学习算法结合.
1. 目前结果对于训练之外的数据, 仍会有很大的偏差, 有没有方法更进一步提高结果的精确性.

对于第一个目标, 实际上是有私心的, 关于深度学习的了解总觉得流于泛泛, 停留在一些课程介绍的概念阶段. 这么优秀的一个库摆在面前, 不失为一个从实际角度探索深度学习的入口. 当然其内部的实现, 无论从算法角度, 还是编程技巧角度, 都是十分值得学习的.

第二个目标, 实际是对项目要求的精益求精. 我觉得达到这个目标应该从两个方面入手:

1. 好好研读其文档: <https://github.com/tesseract-ocr/tesseract/wiki>
1. 从源码里探寻真相.

第二个方面就和第一个目标相辅相成了, 而第一个方面, 则更容易入手一些. 下面就从第一个方面进行一些浅层次的探索.

## 改进图像质量

想要更好的 OCR 结果, [README][0] 中重点强调的一点是: 在交给 Tesseract 之前, 改进图像的质量.

我最早接触 Tesseract 的时候, 最重视的方法就是设置参数, 几乎是摸黑尝试. 后来才发现, 参数的设置起到的作用十分有限, 还不如训练数据来的重要. 当然, [官方][1]专门列出了针对日语与中文的有效参数, 由于我目前不涉及中文的识别, 所以并未深究, 也许正是关键所在.

### 大小重置

Tesseract 处理 300 dpi 以上的图片会更加出色, 所以要对图片的大小有起码的要求. 分辨率和 point size 必须要考虑, 低于 `10pt * 300dpi` 的会被筛掉, 低于 `8pt * 300dpi` 的筛除地更快. 快速对图片进行检查, 是为了计算字符 `x` 的高度(像素). 在 `10pt * 300dpi` 的情况下, `x`的高度通常为 20 像素(字体差异上下浮动). 低于 10 像素的 `x` 字符高度的识别, 很难做到准确了, 如果低于 8 像素, 那么这些文本将在 '去噪' 环节被过滤掉.

### 二值化

二值化的过程, 实际上 Tesseract 内置了, 但处理的应该比较粗暴, 类似 opencv 里的:

```cpp
cv::threshold(m_mat, m_mat, 0, 255, cv::THRESH_BINARY | cv::THRESH_OTSU);
```

这样的话, 可能就会受到光线明暗的极大影响了. 我的经验是, 这个二值化的过程, 尽量由自己进行, 选取一个尽量去除光照影响的算法, 例如: [Adaptive Thresholding][2].

### 去噪

噪点, 往往是二值化过程中, 处理亮度与颜色时遗留下来的. Tesseract 对这些噪点不会去除, 从而影响了结果的准确率.

### 旋转/去偏移

如果目标文字出现倾斜, Tesseract 的 line segmentation 效果会大打折扣. 如果可能的话,应该提前将文字扶正, 保证水平.

### 去边缘

无论是扫描件, 还是照片, 往往在二值化之后, 残留大量的黑线/黑框. 这些会被 Tesseract 错误地拾取, 造成干扰. 最好能够截取目标文字区域, 然后交给 Tesseract.

上述提到的这几项中, 后两项是我所欠缺的. 接下来的改进, 应该从这两点入手. 当然这些会涉及 opencv 的一些细节, 将另起文章记录.

未完待续.

[0]: https://github.com/tesseract-ocr/tesseract/blob/master/README.md
[1]: https://github.com/tesseract-ocr/tesseract/wiki/ControlParams#useful-parameters-for-japanese-and-chinese
[2]: http://docs.opencv.org/trunk/d7/d4d/tutorial_py_thresholding.html
