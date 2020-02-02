# Python packages for pipeline/workflow development: Airflow, Luigi, Gokart, Kedro, PipelineX


This article compares the following open-source Python packages for pipeline/workflow development:

- Airflow: https://github.com/apache/airflow
- Luigi: https://github.com/spotify/luigi
- Gokart: https://github.com/m3dev/gokart
- Kedro: https://github.com/quantumblacklabs/kedro
- PipelineX: https://github.com/Minyus/pipelinex

All of these packages support:
- Parallel execution
- Log output
- DAG visualization

Disclaimer: I'm the developer of PipelineX.

Note: This article is based on my understanding. Please let me know if you find anything inaccurate. 

## Airflow 

https://github.com/apache/airflow

DAG (workflow) of tasks is defined by Python code (or optionally YAML using unofficial plugins such as dag-factory (https://github.com/ajbosco/dag-factory)).

### Pros:

- Airflow provides strong scheduling feature.
- Airflow provides GUI in which users can monitor workflow execution progress and trigger execution.
- Airflow provides distributed computing option (using Celery).
- DAG definition is modular; independent from processing functions.
- Workflow can be nested using `SubDagOperator`.

### Cons:

- Airflow stores metadata in a database (MySQL or Postgres recommended by Airflow) and data can be shared among dependent tasks only via the database (using features called "XCom" or "Variable").
- You need to manage passing/sharing of unstructured data (e.g. image, video, pickle, etc.), if any, outside of Airflow.
- Airflow does not support automatic pipeline resuming option using the intermediate data files.
- You need to write file/database access (read/write) code. 

Airflow might be good for production, but apparently not for rapid experimentation. 


## Luigi

https://github.com/spotify/luigi

Pipeline is defined by task class methods (`requires`, `output`, `run`) in Python code.

### Pros:

- Support automatic pipeline resuming option using the intermediate data files in local or cloud (AWS, GCP, Azure) as well as databases as defined in `Task.output` method using `Target` class.
- You can write code so any data can be passed between dependent tasks.
- Luigi provides GUI in which users can monitor the pipeline.
- Luigi supports Hadoop.

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

- Gokart provides built-in file access (read/write) wrappers as `FileProcessor` classes for pickle, npz, gz, txt, csv, tsv, json, xml.
- Save parameters for each experiment to assure reproducibility. Viewer called `thunderbolt` (https://github.com/m3dev/thunderbolt) can be used.
- Syntactic sugar for Luigi's `requires` class method using class decorator. 


### Cons:

- Supported data formats for file access wrappers are limited.


## Kedro

https://github.com/quantumblacklabs/kedro

Pipeline is defined in Python code (an independent Python module).

### Pros:

- Kedro provides built-in file access (read/write) wrappers as `DataSet` classes for CSV, Pickle, YAML, JSON, Parquet, Excel, and text in local or cloud (S3 in AWS, GCS in GCP), as well as SQL, spark, feather, and more in contrib. 
- Any data format support can be added by users and is easily reusable in future projects. 
- File access functionality is modular; independent from processing functions. Can be configured in YAML (`DataCatalog`).
- Pipeline definition is modular; independent from processing functions.
- Pipelines can be nested.

### Cons:
- Kedro does not support automatic pipeline resuming option using the intermediate data files.
- Kedro's GUI (`kedro-viz`) does not include execution monitoring.
- Kedro installs package dependencies which are not used in many cases (e.g. pyarrow) as defined in the requirements.txt.


## PipelineX:

https://github.com/Minyus/pipelinex

PipelineX works on top of Kedro. 

Pipeline is defined in YAML or Python code.

### Pros:
- PipelineX supports automatic pipeline resuming option using the intermediate data files.
- Optional syntactic sugar for Kedro Pipeline. (e.g. Sequential API similar to Keras & PyTorch)
- Optional syntactic sugar for Kedro DataSet catalog. (e.g. Use file name in file path as the dataset instance name)
- Backward-compatible to pure Kedro.
- Integration with MLflow to save parameters, metrics, and other output artifacts such as models for each experiment to assure reproducibility.
- Integration with common packages for Data Science: PyTorch, Ignite, pandas, OpenCV.
- Additional `DataSet` including (a folder including) images useful for computer vision applications.
- Lean project template compared with pure Kedro.

### Cons:
- PipelineX is developed and maintained by an individual (me) at this moment.


## Platform-specific packages

### Argo

https://github.com/argoproj/argo

Use Kubernetes/Docker for each task in the pipeline

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
- Kedro
    - https://github.com/quantumblacklabs/kedro
    - https://kedro.readthedocs.io/en/latest/kedro.io.html#data-sets
    - https://medium.com/mhiro2/building-pipeline-with-kedro-for-ml-competition-63e1db42d179
- PipelineX
    - https://github.com/Minyus/pipelinex
- Airflow vs Luigi
    - https://towardsdatascience.com/data-pipelines-luigi-airflow-everything-you-need-to-know-18dc741449b7
    - https://medium.com/better-programming/airbnbs-airflow-versus-spotify-s-luigi-bd4c7c2c0791
 