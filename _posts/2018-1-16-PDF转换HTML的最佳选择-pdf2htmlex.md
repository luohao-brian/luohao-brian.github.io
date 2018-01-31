---
layout: post
title: PDF转换HTML的最佳选择:pdf2htmlex
categories: pdf2htmlex
tags: [Linux]
---
> 先直接上转换后截图，PDF是Intel的EPT编程手册，采用pdf2htmlex转换以后，可以看到完全保留了原pdf的目录，排版，格式，字体等，非常完美。

![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2.jpeg)

------
### 安装
**Mac OS**
```sh
brew install pdf2htmlex
```
**Docker**
```sh
docker pull bwits/pdf2htmlex-alpine
alias pdf2htmlEX="docker run -ti --rm -v ~/pdf:/pdf bwits/pdf2htmlex-alpine pdf2htmlEX"
```
### 使用
```sh
pdf2htmlEX  --zoom 1.5  --split-pages 1 aaa.pdf --page-filename aaa_%d.html
```
* zoom 1.5: 将默认输出放大为原来的150%
* split-pages 1: 默认为单页面，导致产生的html页面太大，该参数将输出拆分为多个页面。
* page-filename aaa_%d.html: 比如第二页，生成的页面是aaa_2.html

### 发布
```sh
python -m SimpleHTTPServer
```

用上面的python命令开启一个http服务，浏览器打开[http://localhost:8000](http://localhost:8000), 选择aaa.html，就可以看到发布后的效果了。