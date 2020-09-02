---
layout: single
title:  "SonarQube + Swift/Objective-C 实践"
date:   2020-09-02 22:00:00 +0800
categories: devops
tags: [sonarqube, swift, codequality]
---


SonarQube 社区版不支持对 Swift 和 Objective-C 的扫描，需要借助于开源插件 [sonar-swift](https://github.com/Idean/sonar-swift) 来实现。

## 环境准备

### 安装 sonar-swift 插件
把 插件的 jar 包放入 /opt/sonarqube/extensions/plugins 目录，然后重启 SonarQube Server。

也可以使用 docker 部署 SonarQube：
```shell
docker run -d --name sonarqube -p 9000:9000 -v $(pwd)/backelite-sonar-swift-plugin-0.4.6.jar:/opt/sonarqube/extensions/plugins/backelite-sonar-swift-plugin-0.4.6.jar sonarqube
```

### 安装各种代码检查工具
```shell
brew install swiftlint
brew install tailor
brew install oclint
gem install xcpretty  # 要先安装 ruby，并且把 gem executable 目录，比如 /usr/local/lib/ruby/gems/2.7.0/bin/，加入 $PATH 中
gem install slather
pip install lizard --user
```

## 使用

拷贝一份 [sonar-project.properties](https://raw.githubusercontent.com/Backelite/sonar-swift/master/sonar-project.properties) 放在待扫描的代码仓库的根路径下，并按照需要做修改。

这里用 [SwiftyJSON](https://github.com/SwiftyJSON/SwiftyJSON) 仓库为例做测试。
对于该仓库需要在 `sonar-project.properties` 修改或者添加这些配置：

```
sonar.projectKey=SwiftyJSON
sonar.projectName=SwiftyJSON
sonar.swift.project=SwiftyJSON.xcodeproj
sonar.swift.workspace=SwiftyJSON.xcworkspace
sonar.swift.simulator=platform=iOS Simulator,name=iPhone 11,OS=13.6
sonar.swift.appScheme=SwiftyJSON iOS
sonar.sources=Source
sonar.tests=Tests
# 添加 sonarqube server 信息
sonar.host.url=http://localhost:9000
sonar.login=admin
sonar.password=admin
```

拷贝一份 [run-sonar-swift.sh](https://raw.githubusercontent.com/Backelite/sonar-swift/master/sonar-swift-plugin/src/main/shell/run-sonar-swift.sh) 到代码仓库根目录。
执行扫描：

```
bash run-sonar-swift.sh -nounittests -v
```

完成后即可在 SonarQube 网站上看到扫描结果。

注：脚本运行过程中会产生 lint 结果，如 `swiftlint.txt`，`tailor.txt` 等，都放在 `sonar-reports` 目录中。


## 问题和解决方法

（1）xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance
解决：sudo xcode-select -s /Applications/Xcode.app/Contents/Developer

(2) 如何知道 appScheme：
执行 xcodebuild -list

（3）xcodebuild: error: Unable to find a destination matching the provided destination specifier:
		{ platform:iOS Simulator, OS:latest, name:iPhone X }

根据报错信息修改 `sonar.swift.simulator` 属性。


（4）appScheme 里如果有空格，`run-sonar-swift.sh` 脚本会报错，需要修改脚本的 315 行，给 `$appScheme` 加上双引号：`buildCmd+=(-scheme "$appScheme")`

（5）slather 报找不到 coverage 文件，暂时没找到解决方法。 workaround：不执行测试，`bash run-sonar-swift.sh -nounittests -v`



