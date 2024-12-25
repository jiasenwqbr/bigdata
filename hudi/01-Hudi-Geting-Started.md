# Hudi

## Overview 概述

Hello there! This overview will provide a high level summary of what Apache Hudi is and will orient you on how to learn more to get started.

你好！本概述将提供 Apache Hudi 的高级摘要，并指导您如何了解更多信息以开始使用。

こんにちは! この概要では、Apache Hudi の概要を説明し、使用を開始するためにさらに学習する方法について説明します。

### What is Apache Hudi 什么是Apache Hudi Apache Hudiとは

Apache Hudi (pronounced "hoodie") pioneered the concept of "[transactional data lakes](https://www.uber.com/blog/hoodie/)", which is more popularly known today as the data lakehouse architecture. Today, Hudi has grown into an [open data lakehouse platform](https://hudi.apache.org/cn/blog/2021/07/21/streaming-data-lake-platform), with a open table format purpose-built for high performance writes on incremental data pipelines and fast query performance due to comprehensive table optimizations.

Apache Hudi（发音为“hoodie”）率先提出了“[事务性数据湖](https://www.uber.com/blog/hoodie/)”的概念，如今更广为人知的名称是数据湖屋架构。如今，Hudi 已发展成为一个[开放数据湖平台](https://hudi.apache.org/cn/blog/2021/07/21/streaming-data-lake-platform)，其开放表格式专为在增量数据管道上实现高性能写入和通过全面的表优化实现快速查询性能而构建。

Apache Hudi (「フーディー」と発音) は、「[トランザクション データ レイク](https://www.uber.com/blog/hoodie/)」という概念の先駆者であり、これは今日ではデータ レイクハウス アーキテクチャとしてよく知られています。今日、Hudi は [オープン データ レイクハウス プラットフォーム](https://hudi.apache.org/cn/blog/2021/07/21/streaming-data-lake-platform) に成長し、包括的なテーブル最適化による増分データ パイプラインでの高パフォーマンス書き込みと高速クエリ パフォーマンスを実現するために特別に構築されたオープン テーブル形式を備えています。

Hudi brings core database functionality directly to a data lake - [tables](https://hudi.apache.org/cn/docs/next/sql_ddl), [transactions](https://hudi.apache.org/cn/docs/next/timeline), [efficient upserts/deletes](https://hudi.apache.org/cn/docs/next/write_operations), [advanced indexes](https://hudi.apache.org/cn/docs/next/indexes),[ingestion services](https://hudi.apache.org/cn/docs/hoodie_streaming_ingestion), data [clustering](https://hudi.apache.org/cn/docs/next/clustering)/[compaction](https://hudi.apache.org/cn/docs/next/compaction) optimizations, and [concurrency control](https://hudi.apache.org/cn/docs/next/concurrency_control) all while keeping your data in open file formats. Not only is Apache Hudi great for streaming workloads, but it also allows you to create efficient incremental batch pipelines. Apache Hudi can easily be used on any [cloud storage platform](https://hudi.apache.org/cn/docs/cloud). Hudi’s advanced performance optimizations, make analytical queries/pipelines faster with any of the popular query engines including, Apache Spark, Flink, Presto, Trino, Hive, etc.

Hudi 将核心数据库功能直接带到数据湖 - [表](https://hudi.apache.org/cn/docs/next/sql_ddl)、[事务](https://hudi.apache.org/cn/docs/next/timeline)、[高效的 upserts/deletes](https://hudi.apache.org/cn/docs/next/write_operations)、[高级索引](https://hudi.apache.org/cn/docs/next/indexes)、[提取服务](https://hudi.apache.org/cn/docs/hoodie_streaming_ingestion)、数据[聚类](https://hudi.apache.org/cn/docs/next/clustering)/[压缩](https://hudi.apache.org/cn/docs/next/compaction)优化和[并发控制](https://hudi.apache.org/cn/docs/next/concurrency_control)，同时将数据保存为开放文件格式。Apache Hudi 不仅非常适合流式工作负载，还允许您创建高效的增量批处理管道。Apache Hudi 可轻松用于任何[云存储平台](https://hudi.apache.org/cn/docs/cloud)。Hudi 的高级性能优化使任何流行的查询引擎（包括 Apache Spark、Flink、Presto、Trino、Hive 等）的分析查询/管道速度更快。

Hudi は、コア データベース機能をデータ レイクに直接提供します - [テーブル](https://hudi.apache.org/cn/docs/next/sql_ddl)、[トランザクション](https://hudi.apache.org/cn/docs/next/timeline)、[効率的な upsert/delete](https://hudi.apache.org/cn/docs/next/write_operations)、[高度なインデックス](https://hudi.apache.org/cn/docs/next/indexes)、[取り込みサービス](https://hudi.apache.org/cn/docs/hoodie_streaming_ingestion)、データ [クラスタリング](https://hudi.apache.org/cn/docs/next/clustering)/[圧縮](https://hudi.apache.org/cn/docs/next/compaction) の最適化、[同時実行] Apache Hudi は、データをオープン ファイル形式のままにして、リアルタイムのストリーミングと並列処理 ([concurrency control](https://hudi.apache.org/cn/docs/next/concurrency_control) を実現します。Apache Hudi はストリーミング ワークロードに最適であるだけでなく、効率的な増分バッチ パイプラインを作成することもできます。Apache Hudi は、あらゆる [クラウド ストレージ プラットフォーム](https://hudi.apache.org/cn/docs/cloud) で簡単に使用できます。Hudi の高度なパフォーマンス最適化により、Apache Spark、Flink、Presto、Trino、Hive などの一般的なクエリ エンジンを使用して、分析クエリ/パイプラインを高速化できます。
Hudi wa, koa dētabēsu kinō o dēta reiku ni chokusetsu teikyō shimasu - [tēburu] (https: / / Hudi. Apache. Orugu/ cn/ docs/ next/ sql _ ddl),[toranzakushon] (https: / / Hudi. Apache. Orugu/ cn/ docs/ next/ timeline),[kōritsu-tekina upsert/ delete] (https: / / Hudi. Apache. Orugu/ cn/ docs/ next/ raito _ operations),[kōdona indekkusu] (https: / / Hudi. Apache. Orugu/ cn/ docs/ next/ indexes),[torikomi sābisu] (https: / / Hudi. Apache. Orugu/ cn/ docs/ hoodie _ streaming _ ingestion), dēta [kurasutaringu] (https: / / Hudi. Apache. Orugu/ cn/ docs/ next/ clustering)/ [asshuku] (https: / / Hudi. Apache. Orugu/ cn/ docs/ next/ compaction) no saiteki-ka,[dōji jikkō] apatchi Hudi wa, dēta o ōpun fairu keishiki no mama ni shite, riarutaimu no sutorīmingu to heiretsu shori ( [concurrency control] (https: / / Hudi. Apache. Orugu/ cn/ docs/ next/ concurrency _ control) o jitsugen shimasu. Apatchi Hudi wa sutorīmingu wākurōdo ni saitekidearu dakedenaku, kōritsu-tekina zōbun batchi paipurain o sakusei suru koto mo dekimasu. Apatchi Hudi wa, arayuru [Kura udo sutorēji purattofōmu] (https: / / Hudi. Apache. Orugu/ cn/ docs/ kuraudo) de kantan ni shiyō dekimasu. Hudi no kōdona pafōmansu saiteki-ka ni yori, apatchi Spark, Flink, puresuto, Trino, Hive nado no ippantekina kueri enjin o shiyō shite, bunseki kueri/ paipurain o kōsoku-ka dekimasu.

Read the docs for more [use case descriptions](https://hudi.apache.org/cn/docs/use_cases) and check out [who's using Hudi](https://hudi.apache.org/cn/powered-by), to see how some of the largest data lakes in the world including [Uber](https://eng.uber.com/uber-big-data-platform/), [Amazon](https://aws.amazon.com/blogs/big-data/how-amazon-transportation-service-enabled-near-real-time-event-analytics-at-petabyte-scale-using-aws-glue-with-apache-hudi/), [ByteDance](http://hudi.apache.org/blog/2021/09/01/building-eb-level-data-lake-using-hudi-at-bytedance), [Robinhood](https://s.apache.org/hudi-robinhood-talk) and more are transforming their production data lakes with Hudi.

阅读文档了解更多 [用例描述](https://hudi.apache.org/cn/docs/use_cases) 并查看 [谁在使用 Hudi](https://hudi.apache.org/cn/powered-by)，了解世界上一些最大的数据湖的使用情况，包括 [Uber](https://eng.uber.com/uber-big-data-platform/)、[Amazon](https://aws.amazon.com/blogs/big-data/how-amazon-transportation-service-enabled-near-real-time-event-analytics-at-petabyte-scale-using-aws-glue-with-apache-hudi/)、[ByteDance](http://hudi.apache.org/blog/2021/09/01/building-eb-level-data-lake-using-hudi-at-bytedance)， [Robinhood](https://s.apache.org/hudi-robinhood-talk) 等公司正在使用 Hudi 改造其生产数据湖。

詳細な[ユースケースの説明](https://hudi.apache.org/cn/docs/use_cases)についてはドキュメントをお読みください。また、[Hudi を使用している企業](https://hudi.apache.org/cn/powered-by) をチェックして、[Uber](https://eng.uber.com/uber-big-data-platform/)、[Amazon](https://aws.amazon.com/blogs/big-data/how-amazon-transportation-service-enabled-near-real-time-event-analytics-at-petabyte-scale-using-aws-glue-with-apache-hudi/)、[ByteDance](http://hudi.apache.org/blog/2021/09/01/building-eb-level-data-lake-using-hudi-at-bytedance)など、世界最大級のデータレイクがどのように Hudi を使用しているかを確認してください。 [Robinhood](https://s.apache.org/hudi-robinhood-talk) など多くの企業が、Hudi を使用して本番環境のデータ レイクを変革しています。

[
Shōsaina [yūsukēsu no setsumei] (https: / / Hudi. Apache. Orugu/ cn/ docs/ use _ cases) ni tsuite wa dokyumento o o yomi kudasai. Mata,[Hudi o shiyō shite iru kigyō] (https: / / Hudi. Apache. Orugu/ cn/ powered - by) o chekku shite,[Uber] (https: / / Eng. Uber. Komu/ uber - big - dēta - purattohōmu/),[amazon] (https: / / Aws. Amazon. Komu/ blogs/ big - dēta/ how - amazon - transportation - service - enabled - near - real - time - ibento - analytics - at - petabyte - sukēru - using - aws - gurū - u~izu - apache - hudi/),[ByteDance] (http: / / Hudi. Apache. Orugu/ blog/ 2021/ 09/ 01/ building - eb - level - dēta - lake - using - hudi - at - bytedance) nado, sekai saidai-kyū no dētareiku ga dono yō ni Hudi o shiyō shite iru ka o kakuninshitekudasai. [Robinhood] (https: / / S. Apache. Orugu/ hudi - robinhood - tōku) nado ōku no kigyō ga, Hudi o shiyō shite honban kankyō no dēta reiku o henkaku shite imasu. [

[Hudi-rs](https://github.com/apache/hudi-rs) is the native Rust implementation for Apache Hudi, which also provides bindings to Python. It expands the use of Apache Hudi for a diverse range of use cases in the non-JVM ecosystems.