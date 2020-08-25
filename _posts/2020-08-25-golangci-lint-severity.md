---
layout: single
title:  "如何设置 golangci-lint 报告中的 severity"
date:   2020-08-25 23:00:00 +0800
categories: go
tags: [go, golangci-lint]
---

记录工作中遇到的一个小问题。

问题来源：将运行 golangci-lint 产生的 checkstyle 报告上传到 SonarQube 后（通过 sonar-scanner 指定报告路径），所有的 issue 都会被归类为 BUG，严重等级都是 MAJOR。希望能够按照业务需要调整 issue 的类别和等级。

网上也有人有类似的[问题](https://community.sonarsource.com/t/change-external-issues-language-type-and-severity/17031/5)，根据解答，SonarQube 判断的逻辑是：
（1）如果 checkstyle 报告中的 severity 是 error，则 issue 类别是 BUG，否则是 CODE SMELL；
（2）如果 severity 是 info，则 issue 严重级别是 MINOR，否则是 MAJOR。

这处逻辑可见[相应的代码](https://github.com/SonarSource/slang/blob/ecefaee360329a052164201e4ae5509c3692ae9c/checkstyle-import/src/main/java/org/sonarsource/slang/externalreport/CheckstyleFormatImporter.java#L171)。

golangci-lint 默认生成的报告中的 severity 都是 error，所以对应于 SQ 上都是 MAJOR 级别的 BUG。


因此问题转化为如何调整 golangci-lint checkstyle 报告中 的 severity 级别?

浏览官方文档，发现是提供了相应[配置](https://golangci-lint.run/usage/configuration/#config-file)的，可以配置默认的 severity，也可以细化到每个 linter 的 severity。对应的 [github issue](https://github.com/golangci/golangci-lint/issues/127) 和 [PR](https://github.com/golangci/golangci-lint/pull/1155)。注意，这个 PR 在 v1.28.0 才合入，所以这些配置只对于 v1.28.0 及以上版本才生效。

还存在一个问题没弄明白：
将 `severity.default-severity` 设置为 `info`，产生的 checkstyle 报告中的 severity 还是 `error`。只有将用到的 linter 的 `severity` 都设置为 `info` 后才生效了。