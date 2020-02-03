# Comparison of Python pipeline packages: Airflow, Luigi, Gokart, Metaflow, Kedro, PipelineX


This article compares open-source Python packages for pipeline/workflow development: Airflow, Luigi, Gokart, Metaflow, Kedro, PipelineX.

Please kindly let me know if you find anything inaccurate. 

Disclaimer: I'm the developer of PipelineX.


## Airflow 

https://github.com/apache/airflow

Released in 2015 by Airbnb.
Airflow enables you to define your DAG (workflow) of tasks in Python code (an independent Python module).

(Optionally, unofficial plugins such as [dag-factory](https://github.com/ajbosco/dag-factory) enables you to define DAG in YAML.)

### Pros:

- Provides rich GUI with features including DAG visualization, execution progress monitoring, scheduling, and triggering.
- Provides distributed computing option (using Celery).
- DAG definition is modular; independent from processing functions.
- Workflow can be nested using `SubDagOperator`.

### Cons:

- Not designed to pass data between dependent tasks without using a database. 
There is no good way to pass unstructured data (e.g. image, video, pickle, etc.) between dependent tasks in Airflow.
- You need to write file access (read/write) code. 
- Does not support automatic pipeline resuming option using the intermediate data files or databases.


## Luigi

https://github.com/spotify/luigi

Released in 2012 by Spotify.
Luigi enables you to define your pipeline by child classes of `Task` with 3 class methods (`requires`, `output`, `run`) in Python code.

### Pros:

- Support automatic pipeline resuming option using the intermediate data files in local or cloud (AWS, GCP, Azure) or databases as defined in `Task.output` method using `Target` class.
- You can write code so any data can be passed between dependent tasks.
- Provides GUI with features including DAG visualization, execution progress monitoring.
- Supports Hadoop.

### Cons:

- You need to write file/database access (read/write) code.
- Pipeline definition is not modular. You need to modify the task classes to reuse in future projects.
- Each Task class will be the project-specific and it will be difficult to reuse the Luigi tasks in future projects without modifying.
- You may feel it is not intuitive to pass multiple data objects between tasks.


## Gokart

https://github.com/m3dev/gokart

Released in Dec 2018 by M3.
Gokart works on top of Luigi. 

### Pros: 

- Provides built-in file access (read/write) wrappers as `FileProcessor` classes for pickle, npz, gz, txt, csv, tsv, json, xml.
- Save parameters for each experiment to assure reproducibility. Viewer called `thunderbolt` (https://github.com/m3dev/thunderbolt) can be used.
- Rerun tasks upon parameter change based on hash string unique to the parameter set in each intermediate file name.
- Syntactic sugar for Luigi's `requires` class method using class decorator. 


### Cons:

- Supported data formats for file access wrappers are limited.


## Metaflow

https://github.com/Netflix/metaflow

Released in Dec 2019 by Netflix.
Metaflow enables you to define your pipeline as a child class of `FlowSpec` that includes class methods with `step` decorators in Python code.

### Pros:

- Intuitive to define pipeline dependencies.

### Cons:

- You need to write file/database access (read/write) code.
- Pipeline definition is not modular. You need to modify the task classes to reuse in future projects.
- Each Task class will be the project-specific and it will be difficult to reuse the Luigi tasks in future projects without modifying.
- Does not support GUI. 
- Not much support for GCP & Azure.
- Does not support automatic pipeline resuming option using the intermediate data files or databases.


## Kedro

https://github.com/quantumblacklabs/kedro

Released in May 2019 by QuantumBlack/McKinsey & Company.
Kedro enables you to define pipelines in Python code (an independent Python module).

### Pros:

- Provides built-in file access (read/write) wrappers as `DataSet` classes for CSV, Pickle, YAML, JSON, Parquet, Excel, and text in local or cloud (S3 in AWS, GCS in GCP), as well as SQL, spark, feather, and more in contrib. 
- Any data format support can be added by users and is easily reusable in future projects. 
- File access (read/write) functionality is modular; independent from processing functions. Can be configured in YAML (`DataCatalog`).
- Pipeline definition is modular; independent from processing functions.
- Pipelines can be nested. (A pipeline can be used as sub-pipeline. )
- GUI (`kedro-viz`) provides DAG visualization feature.

### Cons:
- Does not support automatic pipeline resuming option using the intermediate data files or databases.
- GUI (`kedro-viz`) does not provide execution progress monitoring feature.
- Kedro installs package dependencies which are not used in many cases (e.g. pyarrow) as defined in the requirements.txt.


## PipelineX:

https://github.com/Minyus/pipelinex

Released in Nov 2019 by a Kedro user (me).
PipelineX works on top of Kedro. 

PipelineX enables you to define your pipeline in YAML (an independent YAML file).

### Pros:
- Supports automatic pipeline resuming option using the intermediate data files or databases.
- Optional syntactic sugar for Kedro Pipeline. (e.g. Sequential API similar to PyTorch (`torch.nn.Sequential`) and Keras (`tf.keras.Sequential`))
- Optional syntactic sugar for Kedro `DataSet` catalog. (e.g. Use file name in the file path as the dataset instance name)
- Backward-compatible to pure Kedro.
- Integration with MLflow to save parameters, metrics, and other output artifacts such as models for each experiment to assure reproducibility.
- Integration with common packages for Data Science: PyTorch, Ignite, pandas, OpenCV.
- Additional `DataSet` including image set (a folder including images) useful for computer vision applications.
- Lean project template compared with pure Kedro.

### Cons:
- PipelineX is developed and maintained by an individual (me) at this moment.


## Summary

- 👍: good
- 👍👍: better

| Package                                                                 | Airflow | Luigi&nbsp;&nbsp;&nbsp; | Gokart | Metaflow | Kedro&nbsp;&nbsp;&nbsp; | PipelineX       |
|-------------------------------------------------------------------------|---------|-------|--------|----------|-------|-----------------|
| Wrapped packages                                                        |         |       | Luigi  |          |       | Kedro, MLflow   |
| Easiness/flexibility to define DAG                                      |         |       | 👍      | 👍        | 👍     | 👍👍             |
| Modularity of DAG definition                                            | 👍👍       |       |        |          | 👍👍     | 👍👍               |
| Unstructured data can be passed between tasks                           |         | 👍👍     | 👍👍      | 👍👍        | 👍👍     | 👍👍               |
| Built\-in file/database availability check wrappers                     |         | 👍👍   | 👍👍    |          | 👍👍   | 👍👍             |
| Built\-in file/database operation (read/write) wrappers                 |         |       | 👍      |          | 👍👍   | 👍👍             |
| Modularity, reusability, testability of file/database operation         |         |       | 👍      |          | 👍👍   | 👍👍             |
| Automatic pipeline resuming option using the intermediate data files    |         | 👍👍     | 👍👍      |          |       | 👍👍               |
| Force rerun of tasks upon parameter change                              |         |       | 👍👍      |          |       |                 |
| Save parameters for experiments                                         |         |       | 👍👍      |          |       | 👍👍               |
| Parallel execution                                                      | 👍       | 👍     | 👍      | 👍        | 👍     | 👍               |
| Distributed parallel execution with Celery                              | 👍👍       |       |        |          |       |                 |
| Visualization of DAG                                                    | 👍👍       | 👍     | 👍      |          | 👍     | 👍               |
| Monitoring in GUI                                                       | 👍👍     | 👍     | 👍      |          |       |                 |
| Scheduling, Triggering in GUI                                           | 👍       |       |        |          |       |                 |
| Notification to Slack                                                   | 👍       |       | 👍      |          |       |                 |


## Platform-specific packages

### Argo

https://github.com/argoproj/argo

Uses Kubernetes/Docker for each task in the pipeline

### Kubeflow Pipelines

https://github.com/kubeflow/pipelines

Works on top of Argo

### Oozie

https://github.com/apache/oozie

Manages Hadoop jobs


### Azkaban

https://github.com/azkaban/azkaban

Manages Hadoop jobs


## References:

- Airflow
    - https://github.com/apache/airflow
    - https://airflow.apache.org/docs/stable/howto/initialize-database.html
- Luigi
    - https://github.com/spotify/luigi
    - https://luigi.readthedocs.io/en/stable/api/luigi.contrib.html
    - https://www.m3tech.blog/entry/2018/11/12/110000
- Gokart
    - https://github.com/m3dev/gokart
    - https://www.m3tech.blog/entry/2019/09/30/120229
    - https://qiita.com/Hase8388/items/8cf0e5c77f00b555748f
- Metaflow
    - https://github.com/Netflix/metaflow
    - https://docs.metaflow.org/metaflow/basics
    - https://medium.com/bigdatarepublic/a-review-of-netflixs-metaflow-65c6956e168d
- Kedro
    - https://github.com/quantumblacklabs/kedro
    - https://kedro.readthedocs.io/en/latest/kedro.io.html#data-sets
    - https://medium.com/mhiro2/building-pipeline-with-kedro-for-ml-competition-63e1db42d179
- PipelineX
    - https://github.com/Minyus/pipelinex
- Airflow vs Luigi
    - https://towardsdatascience.com/data-pipelines-luigi-airflow-everything-you-need-to-know-18dc741449b7
    - https://medium.com/better-programming/airbnbs-airflow-versus-spotify-s-luigi-bd4c7c2c0791
    - https://www.quora.com/Which-is-a-better-data-pipeline-scheduling-platform-Airflow-or-Luigi
 