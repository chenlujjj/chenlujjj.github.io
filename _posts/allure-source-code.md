alluer by default 会 load 一些插件，见类 `ConfigurationBuilder` 代码：
```Java
public ConfigurationBuilder useDefault() {
    fromExtensions(Arrays.asList(
            new JacksonContext(),
            new MarkdownContext(),
            new FreemarkerContext(),
            new RandomUidContext(),
            new MarkdownDescriptionsPlugin(),
            new RetryPlugin(),
            new RetryTrendPlugin(),
            new TagsPlugin(),
            new SeverityPlugin(),
            new OwnerPlugin(),
            new IdeaLinksPlugin(),
            new HistoryPlugin(),
            new HistoryTrendPlugin(),
            new CategoriesPlugin(),
            new CategoriesTrendPlugin(),
            new DurationPlugin(),
            new DurationTrendPlugin(),
            new StatusChartPlugin(),
            new TimelinePlugin(),
            new SuitesPlugin(),
            new ReportWebPlugin(),
            new TestsResultsPlugin(),
            new AttachmentsPlugin(),
            new MailPlugin(),
            new InfluxDbExportPlugin(),
            new PrometheusExportPlugin(),
            new SummaryPlugin(),
            new ExecutorPlugin(),
            new LaunchPlugin(),
            new Allure1Plugin(),
            new Allure1EnvironmentPlugin(),
            new Allure2Plugin(),
            new GaPlugin()
    ));
```
另有 allure_home 目录下的 plugins，也会默认 load：
```bash
$ pwd
/usr/local/Cellar/allure/2.12.1/libexec/plugins
$ ls
README.txt         jira-plugin        screen-diff-plugin xray-plugin
behaviors-plugin   junit-xml-plugin   trx-plugin         xunit-xml-plugin
custom-logo-plugin packages-plugin    xctest-plugin
```
插件按功能可分为两类：
1. 实现 `Aggregator` 接口，职责是对 `List<LaunchResults>` 做分析，将分析结果以文件形式写入 allure 报告目录中
2. 实现 `Reader` 接口，职责是对 test results 做处理，通过 `ResultsVisitor` 将信息存入 `LaunchResults` 中



重要插件列举如下：
* `SummaryPlugin`：将汇总信息写入 `allure-report/widgets/summary.json`
* `DurationPlugin`：将每个测试用例的执行时间信息写入 `allure-report/widgets/duration.json`
* `DurationTrendPlugin`：
  * 作为 `Reader`，读取 test results 中的 `history/duration-trend.json` 文件（如果有）；
  * 作为 `Aggregator`，将执行总时间的历史趋势信息写入 `allure-report/widgets/duration-trend.json`
* `CategoriesPlugin`：
  * 作为 `Aggregator`，将 fail 和 broken 的测试数量及其详情写入 `allure-report/data/categories.json`、`allure-report/data/categories.csv`、`allure-report/widgets/categories.json`；
  * 作为 `Reader`，读取 test results 中的 `categories.json` 文件（如果有）。
* `CategoriesTrendPlugin`：将 fail 和 broken 的测试总数量的历史趋势信息写入 `allure-report/widgets/categories-trend.json`
* `SuitesPlugin`：将信息写入 `allure-report/data/suites.json`、`allure-report/data/suites.csv`、`allure-report/widgets/suites.json`，信息包括每个测试用例的名称、所属类名、执行状态、起止时间、是否脆弱、是否新失败等，并按包、类的层级分类。
* `LaunchPlugin`：
  * 作为 `Reader`，读取 test results 中的 `launch.json` 文件（如果有）；
  * 作为 `Aggregator`，更新读取结果中的统计信息，写入 `allure-report/widgets/launch.json`
* `ExecutorPlugin`：
  * 作为 `Reader`，读取 test results 中的 `executor.json` 文件（如果有）；
  * 作为 `Aggregator`，将执行者信息写入 `allure-report/widgets/executor.json`
* `Allure1EnvironmentPlugin`：将执行环境信息写入 `allure-report/widgets/environment.json`
* `RetryTrendPlugin`：
  * 作为 `Reader`，读取 test results 中的 `history/retry-trend.json` 文件（如果有）；
  * 作为 `Aggregator`，将测试用例重试计数写入 `allure-report/widgets/retry-trend.json` 和 `allure-report/history/retry-trend.json` 中
* `SeverityPlugin`：将各个测试用例的名称、耗时、执行状态、严重程度写入 `allure-report/widgets/severity.json` 中
* `StatusChartPlugin`：将各个测试用例的名称、耗时、执行状态、严重程度写入 `allure-report/widgets/status-chart.json` 中 （似乎和 severity 一样，不知为何）
* `HistoryTrendPlugin`：
  * 作为 `Reader`，读取 test results 中的 `history/history-trend.json` 文件（如果有）；
  * 作为 `Aggregator`，将历史的测试用例执行结果的计数写入 `allure-report/widgets/history-trend.json` 和 `allure-report/history/history-trend.json` 中
* `ReportWebPlugin`：
  * 将 `index.html` 文件写入 `allure-report` 目录中
  * 将 `app.js, favicon.ico, styles.css` 写入 `allure-report` 目录中
  * 将各个插件的静态文件分插件 id 写入 `allure-report/plugins` 目录中
* `AttachmentsPlugin`：将 attachments 存入 `allure-report/data/attachments` 目录中
* `TestsResultsPlugin`：将各个测试用例的全量信息（包括用例名、时间、状态、参数、所属类和包、历史记录等）存入 `allure-report/data/test-cases` 目录中，每个测试用例对应一个 json 文件，以 uid 命名。
* `TimelinePlugin`：将测试用例基本信息按时间排序后写入 `allure-report/data/timeline.json` 中
* `MailPlugin`,`InfluxDbExportPlugin`,`PrometheusExportPlugin`：分别写 `allure-report/export/mail.html`,`influxDbData.txt`,`prometheusData.txt`文件
* `Allure2Plugin`：仅作为 `Reader`，读取分析 test results （文件名形式为`*-result.json`，比如 `pytest --alluredir=./allure_results tests` 生成的测试结果），这类 json 文件内容的数据结构和 `TestResult` 类是一致的。

---

* `BehaviorsPlugin`：将信息写入 `allure-report/data/behaviors.json`、`allure-report/data/behaviors.csv`、`allure-report/widgets/behaviors.json`，信息包括每个测试用例的名称、执行状态、起止时间、是否脆弱、是否新失败等。
* `PackagesPlugin`：将测试用例的基本信息按包->类->方法的树形结构存入 `allure-report/data/packages.json` 中
* `JunitXmlPlugin`：仅作为 `Reader`，读取分析 test results （junit style 的 xml 文件，也就是使用 JUnit 框架编写的测试运行后产生的文件）。