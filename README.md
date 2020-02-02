# Python packages for pipeline/workflow development: Airflow, Luigi, Gokart, Metaflow, Kedro, PipelineX


This article compares the following open-source Python packages for pipeline/workflow development:

- Airflow: https://github.com/apache/airflow
- Luigi: https://github.com/spotify/luigi
- Gokart: https://github.com/m3dev/gokart
- Metaflow: https://github.com/Netflix/metaflow
- Kedro: https://github.com/quantumblacklabs/kedro
- PipelineX: https://github.com/Minyus/pipelinex



All of these packages support:
- Parallel execution
- Log output

Disclaimer: I'm the developer of PipelineX.

Note: This article is based on my understanding. Please let me know if you find anything inaccurate. 

## Airflow 

https://github.com/apache/airflow

DAG (workflow) of tasks is defined by Python code (or optionally YAML using unofficial plugins such as dag-factory (https://github.com/ajbosco/dag-factory)).

### Pros:

- Provides strong scheduling feature.
- Provides rich GUI with features including DAG visualization, execution progress monitoring, scheduling, and triggering.
- Provides distributed computing option (using Celery).
- DAG definition is modular; independent from processing functions.
- Workflow can be nested using `SubDagOperator`.

### Cons:

- Data can be shared among dependent tasks only via a database (using features called "XCom", "Variable", or "Hook").
- You need to manage passing/sharing of unstructured data (e.g. image, video, pickle, etc.), if any, outside of Airflow.
- Does not support automatic pipeline resuming option using the intermediate data files.
- You need to write file/database access (read/write) code. 
- Does not support automatic pipeline resuming option using the intermediate data files or databases.

Airflow might be good for production, but apparently not for rapid experimentation. 


## Luigi

https://github.com/spotify/luigi

Pipeline is defined by child classes of `Task` with 3 defined class methods (`requires`, `output`, `run`) in Python code.

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

Gokart works on top of Luigi. 

Pipeline is defined by task class methods (`requires`, `output`, `run`) in Python code.

### Pros: 

- Provides built-in file access (read/write) wrappers as `FileProcessor` classes for pickle, npz, gz, txt, csv, tsv, json, xml.
- Save parameters for each experiment to assure reproducibility. Viewer called `thunderbolt` (https://github.com/m3dev/thunderbolt) can be used.
- Rerun tasks upon parameter change based on hash string unique to the parameter set in each intermediate file name.
- Syntactic sugar for Luigi's `requires` class method using class decorator. 


### Cons:

- Supported data formats for file access wrappers are limited.


## Metaflow

https://github.com/Netflix/metaflow

Pipeline is defined as a child class `FlowSpec` with class methods with `step` decorators and call `next` method in the end to specify the next task in Python code.

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

Pipeline is defined in Python code (an independent Python module).

### Pros:

- Kedro provides built-in file access (read/write) wrappers as `DataSet` classes for CSV, Pickle, YAML, JSON, Parquet, Excel, and text in local or cloud (S3 in AWS, GCS in GCP), as well as SQL, spark, feather, and more in contrib. 
- Any data format support can be added by users and is easily reusable in future projects. 
- File access functionality is modular; independent from processing functions. Can be configured in YAML (`DataCatalog`).
- Pipeline definition is modular; independent from processing functions.
- Pipelines can be nested.
- GUI (`kedro-viz`) provides DAG visualization feature.

### Cons:
- Does not support automatic pipeline resuming option using the intermediate data files or databases.
- GUI (`kedro-viz`) does not provide execution progress monitoring feature.
- Kedro installs package dependencies which are not used in many cases (e.g. pyarrow) as defined in the requirements.txt.


## PipelineX:

https://github.com/Minyus/pipelinex

PipelineX works on top of Kedro. 

Pipeline is defined in YAML or Python code.

### Pros:
- Supports automatic pipeline resuming option using the intermediate data files or databases.
- Optional syntactic sugar for Kedro Pipeline. (e.g. Sequential API similar to Keras & PyTorch)
- Optional syntactic sugar for Kedro DataSet catalog. (e.g. Use file name in file path as the dataset instance name)
- Backward-compatible to pure Kedro.
- Integration with MLflow to save parameters, metrics, and other output artifacts such as models for each experiment to assure reproducibility.
- Integration with common packages for Data Science: PyTorch, Ignite, pandas, OpenCV.
- Additional `DataSet` including (a folder including) images useful for computer vision applications.
- Lean project template compared with pure Kedro.

### Cons:
- PipelineX is developed and maintained by an individual (me) at this moment.


## Summary

- 👍: good
- 👍👍: better

| Package                                                                 | Airflow | Luigi | Gokart | Metaflow | Kedro | PipelineX       |
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
| "Scheduling, Triggering in GUI"                                         | 👍       |       |        |          |       |                 |
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
 