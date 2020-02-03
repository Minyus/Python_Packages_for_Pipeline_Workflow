# PythonのPipelineパッケージ比較：Airflow, Luigi, Gokart, Metaflow, Kedro, PipelineX

この記事では、Open-sourceのPipeline/Workflow開発用PythonパッケージのAirflow, Luigi, Gokart, Metaflow, Kedro, PipelineXを比較します。

この記事では、"Pipeline"、"Workflow"、"DAG"の単語はほぼ同じ意味で使用しています。

## Airflow 

https://github.com/apache/airflow

2015年にAirbnb社からリリースされました。

Airflowは、Pythonコード（独立したPythonモジュール）でDAGを定義します。
（オプションとして、非公式の [dag-factory](https://github.com/ajbosco/dag-factory) 等を使用して、YAMLでDAGを定義できます。）

### 良い点:

- DAGの可視化、実行進捗監視、スケジューリング、トリガリング機能をGUI上で使えます。
- Celeryを使用した分散コンピューティングを使えます。
- DAGの定義はモジュラーで、処理関数とは独立してます。
- Workflowは `SubDagOperator` を使用してネストできます。
- Slackに通知できます。

### 良いとは言えない点:

- 依存タスク間でデータベースを介さずにデータを渡せるように設計されていません。
Airflow内の依存タスク間で非構造化データ（画像、動画、pickle等）を渡す良い方法がありません。
- ファイルアクセス（読み書き）のためのコードが別途必要になります。
- 中間生成データファイルやデータベースを使用して、自動的にPipelineの途中から実行できません。

## Luigi

https://github.com/spotify/luigi

2012年にSpotify社からリリースされました。

Luigiは、Pythonコードで、3つのクラスメソッド(`requires`, `output`, `run`)を持つ`Task`の子クラス達によりPipelineを定義します。

### 良い点:

- `Target`クラスを使用した`Task.output`メソッドで定義されたとおり、ローカルかクラウド(AWS, GCP, Azure)上の中間データファイルやデータベースを使用して、
自動的にPipelineの途中から実行できます。
- 依存タスク間で任意のデータを渡すようにコーディングできます。
- DAGの可視化、実行進捗監視機能をGUI上で使えます。


### 良いとは言えない点:

- ファイルやデータベースアクセス（読み書き）するためのコードを書く必要があります。
- Pipeline定義、タスク処理（ETLのうちのTransform）、データアクセス（ETLのうちのExtract&Load）が密に結合していて、モジュラーではありません。
将来のプロジェクトで再利用するためにタスククラスを修正しないといけません。


## Gokart

https://github.com/m3dev/gokart

2018年12月にエムスリー社からリリースされました。

Gokartは内部でLuigiを使用します。

### 良い点: 

Luigiの良い点に追加として：
- pickle, npz, gz, txt, csv, tsv, json, xml 形式のファイルアクセス（読み書き）ラッパーが`FileProcessor`クラスとして提供されます。
- 各実験パラメータを保存する仕組みが備わっています。[thunderbolt](https://github.com/m3dev/thunderbolt)というビューワーが提供されます。
- 中間ファイル名に含まれた、パラメータセットに固有のハッシュ文字列を参照して、パラメータが変更された際にはタスクを再実行します。
様々なパラメータセットで実験するのに有用です。
- クラスデコレータを使用してLuigiの `requires` クラスメソッドを簡潔に書くためのシンタクティックシュガーが提供されます。
- Slackに通知できます。

### 良いとは言えない点:

- サポートされているデータファイル形式が限られています。サポートされていない形式を使用するためには、ファイルやデータベースアクセス（読み書き）するためのコードを書く必要があります。
- Pipeline定義とタスク処理（ETLのうちのTransform）が密に結合していて、モジュラーではありません。将来のプロジェクトで再利用するためにタスククラスを修正しないといけません。

## Metaflow

https://github.com/Netflix/metaflow

2019年12月にNetflix社からリリースされました。

Metaflowは、Pythonコードで、`step` デコレータ付きのクラスメソッドを含む`FlowSpec`の子クラスによりPipelineを定義します。

### 良い点:

- AWSサービス（特にAWS Batch）とのインテグレーション。

### 良いとは言えない点:

- ファイルやデータベースアクセス（読み書き）するためのコードを書く必要があります。
- Pipeline定義、タスク処理（ETLのうちのTransform）、データアクセス（ETLのうちのExtract&Load）が密に結合していて、モジュラーではありません。
将来のプロジェクトで再利用するためにタスククラスを修正しないといけません。
- GUIはありません。
- GCP、Azureにあまり対応していません。
- 中間生成データファイルやデータベースを使用して、自動的にPipelineの途中から実行できません。


## Kedro

https://github.com/quantumblacklabs/kedro

2019年5月にMcKinseyの子会社のQuantumBlack社からリリースされました。

Kedroは、Pythonコード（独立したPythonモジュール）で、3つの引数
(`func`: タスク処理関数, `inputs`: 入力データ名（複数の場合はlist or dict）, `outputs`: 出力データ名（複数の場合はlist or dict）) 
を持つ`node` 関数のリストによりPipelineを定義します。

### 良い点:

- ローカル又はクラウド（AWSのS3 GCPのGCS）上のCSV, Pickle, YAML, JSON, Parquet, Excel, textファイル、SQL、Spark等のリソースへのアクセス（読み書き）用の
ラッパーを`DataSet`クラスとして備えています。
- 対応していないデータ形式へのサポートは、ユーザーにより追加できます。
- Pipeline定義、タスク処理（ETLのうちのTransform）、データアクセス（ETLのうちのExtract&Load）は独立していて、モジュラーです。
将来のプロジェクトで簡単に再利用できます。
- Pipelineはネストできます。（Pipelineは他のPipelineの一部として使用できます。）
- GUI ([kedro-viz](https://github.com/quantumblacklabs/kedro-viz))でDAGを可視化できます。


### 良いとは言えない点:
- 中間生成データファイルやデータベースを使用して、自動的にPipelineの途中から実行できません。
- GUI ([kedro-viz](https://github.com/quantumblacklabs/kedro-viz))で実行進捗監視する機能はありません。
- 大抵の場合は使用しないパッケージ（pyarrow 等）が`requirements.txt`に含まれています。


## PipelineX:

https://github.com/Minyus/pipelinex

2019年11月にKedroユーザー（私）によりリリースされました。
PipelineXはKedroとMLflowを内部で使用します。
PiplineXは、YAML（独立したYAMLファイル）で、KedroよりもPipelineを定義します。

### 良い点:

Kedroの長所に追加として：
- 中間データファイルやデータベースを使用して、自動的にPipelineの途中から実行できます。
- Kedro Pipelineを簡潔に書くためのシンタクティックシュガーを使用できます。
（例：PyTorch (`torch.nn.Sequential`) や Keras (`tf.keras.Sequential`) に似たSequential API）
- Kedro `DataSet` catalog を簡潔に書くためのシンタクティックシュガーを使用できます。
（例：ファイルパス内のファイル名をデータセットインスタンス名として使用）
- Kedroと後方互換性があります。
- 各実験パラメータ、メトリック、モデルその他の生成データファイルを保存するするためのMLflowとのインテグレーションを使用できます。
- PyTorch, Ignite, pandas, OpenCVといったデータサイエンスで一般的なパッケージとのインテグレーションを使用できます。
- コンピュータビジョンアプリケーションで有用な画像セット（画像を含むフォルダ）を扱うための`DataSet`クラスを使用できます。
- 元のKedroで提供されているものよりもリーンなプロジェクトテンプレートが提供されます。 

### 良いとは言えない点:

- GUI ([kedro-viz](https://github.com/quantumblacklabs/kedro-viz))で実行進捗監視する機能はありません。
- 大抵の場合は使用しないパッケージ（pyarrow 等）がKedroの`requirements.txt`に含まれています。
- PipelineXは、現時点では一個人（私）により開発、メンテナンスされています。


## 要約

- 👍: 良い
- 👍👍: より良い

| パッケージ                                                               | Airflow | Luigi&nbsp;&nbsp;&nbsp; | Gokart | Metaflow | Kedro&nbsp;&nbsp;&nbsp; | PipelineX       |
|-------------------------------------------------------------------------|---------|-------|--------|----------|-------|-----------------|
| ラップされたパッケージ                                                    |         |       | Luigi  |          |       | Kedro, MLflow   |
| DAG定義のしやすさ・柔軟性                                                |         |       | 👍      | 👍        | 👍     | 👍👍             |
| DAG定義のモジュール性                                                      | 👍👍       |       |        |          | 👍👍     | 👍👍               |
| 非構造化データをタスク間で渡せるか                                           |         | 👍👍     | 👍👍      | 👍👍        | 👍👍     | 👍👍               |
| 各種データ（ファイル、データベース）の存在チェックが備わっている                         |         | 👍👍   | 👍👍    |          | 👍👍   | 👍👍             |
| 各種データ（ファイル、データベース）のオペレーション（読み書き）ラッパーが備わっている                     |         |       | 👍      |          | 👍👍   | 👍👍             |
| データ（ファイル、データベース）オペレーションのモジュール性、再利用性、テストのしやすさ |         |       | 👍      |          | 👍👍   | 👍👍             |
| 中間データ（ファイル、データベース）を検知して自動的にPipelineを途中から実行できる  |         | 👍👍     | 👍👍      |          |       | 👍👍               |
| パラメータ変更を検知して強制的にタスクを再実行                               |         |       | 👍👍      |          |       |                 |
| 実験パラメータを保存する                                                   |         |       | 👍👍      |          |       | 👍👍               |
| 並列実行                                                                 | 👍       | 👍     | 👍      | 👍        | 👍     | 👍               |
| Celeryによる分散並列実行                                                  | 👍👍       |       |        |          |       |                 |
| DAGの可視化                                                              | 👍👍       | 👍     | 👍      |          | 👍     | 👍               |
| GUIでの実行状況監視                                                              | 👍👍     | 👍     | 👍      |          |       |                 |
| GUIでのスケジューリング、トリガリング                                       | 👍       |       |        |          |       |                 |
| Slackへの通知                                                            | 👍       |       | 👍      |          |       |                 |


## プラットフォームに特化したパッケージ

### Argo

https://github.com/argoproj/argo

ArgoはKubernetesを使用してPipelineを実行します。

### Kubeflow Pipelines

https://github.com/kubeflow/pipelines

Kubeflow Pipelinesは内部でArgoを使用します。

### Oozie

https://github.com/apache/oozie

Hadoopジョブを管理できます。


### Azkaban

https://github.com/azkaban/azkaban

Hadoopジョブを管理できます。


## リファレンス

Airflow
- https://github.com/apache/airflow
- https://airflow.apache.org/docs/stable/howto/initialize-database.html
- https://medium.com/datareply/integrating-slack-alerts-in-airflow-c9dcd155105

Luigi
- https://github.com/spotify/luigi
- https://luigi.readthedocs.io/en/stable/api/luigi.contrib.html
- https://www.m3tech.blog/entry/2018/11/12/110000

Gokart
- https://github.com/m3dev/gokart
- https://www.m3tech.blog/entry/2019/09/30/120229
- https://qiita.com/Hase8388/items/8cf0e5c77f00b555748f

Metaflow
- https://github.com/Netflix/metaflow
- https://docs.metaflow.org/metaflow/basics
- https://docs.metaflow.org/metaflow/scaling
- https://medium.com/bigdatarepublic/a-review-of-netflixs-metaflow-65c6956e168d

Kedro
- https://github.com/quantumblacklabs/kedro
- https://kedro.readthedocs.io/en/latest/03_tutorial/04_create_pipelines.html
- https://kedro.readthedocs.io/en/latest/kedro.io.html#data-sets
- https://medium.com/mhiro2/building-pipeline-with-kedro-for-ml-competition-63e1db42d179
    
PipelineX
- https://github.com/Minyus/pipelinex
    
Airflow vs Luigi
- https://towardsdatascience.com/data-pipelines-luigi-airflow-everything-you-need-to-know-18dc741449b7
- https://medium.com/better-programming/airbnbs-airflow-versus-spotify-s-luigi-bd4c7c2c0791
- https://www.quora.com/Which-is-a-better-data-pipeline-scheduling-platform-Airflow-or-Luigi
 
## 不正確な点

不正確な点がありましたら、お知らせください。

https://github.com/Minyus/Python_Packages_for_Pipeline_Workflow/blob/master/ja/README.md へのプルリクエストを歓迎します。
