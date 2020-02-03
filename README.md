# Comparison of Python pipeline packages: Airflow, Luigi, Gokart, Metaflow, Kedro, PipelineX


This article compares open-source Python packages for pipeline/workflow development: Airflow, Luigi, Gokart, Metaflow, Kedro, PipelineX.

Disclaimer: I'm the developer of PipelineX.

In this article, terms of "pipeline", "workflow", and "DAG" are used almost interchangeably. 

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
- Supports Slack notification.

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

### Cons:

- You need to write file/database access (read/write) code.
- Pipeline definition, task processing (Transform of ETL), and data access (Extract&Load of ETL) are tightly coupled and not modular. You need to modify the task classes to reuse in future projects.


## Gokart

https://github.com/m3dev/gokart

Released in Dec 2018 by M3.
Gokart works on top of Luigi. 

### Pros: 

In addition to Luigi's advantages:
- Provides built-in file access (read/write) wrappers as `FileProcessor` classes for pickle, npz, gz, txt, csv, tsv, json, xml.
- Save parameters for each experiment to assure reproducibility. Viewer called [thunderbolt](https://github.com/m3dev/thunderbolt) can be used.
- Rerun tasks upon parameter change based on hash string unique to the parameter set in each intermediate file name. 
This feature is useful for experimentation with various parameter sets.
- Syntactic sugar for Luigi's `requires` class method using class decorator. 
- Supports Slack notification.

### Cons:

- Supported data formats for file access wrappers are limited. You need to write file/database access (read/write) code to use unsupported formats.
- Pipeline definition and task processing (Transform of ETL) are tightly coupled and not modular. You need to modify the task classes to reuse in future projects.


## Metaflow

https://github.com/Netflix/metaflow

Released in Dec 2019 by Netflix.
Metaflow enables you to define your pipeline as a child class of `FlowSpec` that includes class methods with `step` decorators in Python code.

### Pros:

- Integration with AWS services (Especially AWS Batch).

### Cons:

- You need to write file/database access (read/write) code.
- Pipeline definition, task processing (Transform of ETL), and data access (Extract&Load of ETL) are tightly coupled and not modular. You need to modify the task classes to reuse in future projects.
- Does not support GUI. 
- Not much support for GCP & Azure.
- Does not support automatic pipeline resuming option using the intermediate data files or databases.


## Kedro

https://github.com/quantumblacklabs/kedro

Released in May 2019 by QuantumBlack, part of McKinsey & Company.

Kedro enables you to define pipelines using list of `node` functions with 3 arguments 
(`func`: task processing function, `inputs`: input data name (list or dict if multiple), `outputs`: output data name (list or dict if multiple)) 
in Python code (an independent Python module).

### Pros:

- Provides built-in file/database access (read/write) wrappers as `DataSet` classes for 
CSV, Pickle, YAML, JSON, Parquet, Excel, and text in local or cloud (S3 in AWS, GCS in GCP), as well as SQL, Spark, etc. 
- Any data format support can be added by users. 
- Pipeline definition, task processing (Transform of ETL), and data access (Extract&Load of ETL) are independent and modular. 
You can easily reuse in future projects.
- Pipelines can be nested. (A pipeline can be used as a sub-pipeline of another pipeline. )
- GUI ([kedro-viz](https://github.com/quantumblacklabs/kedro-viz)) provides DAG visualization feature.

### Cons:
- Does not support automatic pipeline resuming option using the intermediate data files or databases.
- GUI ([kedro-viz](https://github.com/quantumblacklabs/kedro-viz)) does not provide execution progress monitoring feature.
- Package dependencies which are not used in many cases (e.g. pyarrow) are included in the `requirements.txt`.


## PipelineX:

https://github.com/Minyus/pipelinex

Released in Nov 2019 by a Kedro user (me).
PipelineX works on top of Kedro and MLflow.

PipelineX enables you to define your pipeline in YAML (an independent YAML file).

### Pros:

In addition to Kedro's advantages:
- Supports automatic pipeline resuming option using the intermediate data files or databases.
- Optional syntactic sugar for Kedro Pipeline. (e.g. Sequential API similar to PyTorch (`torch.nn.Sequential`) and Keras (`tf.keras.Sequential`))
- Optional syntactic sugar for Kedro `DataSet` catalog. (e.g. Use file name in the file path as the dataset instance name)
- Backward-compatible to pure Kedro.
- Integration with MLflow to save parameters, metrics, and other output artifacts such as models for each experiment.
- Integration with common packages for Data Science: PyTorch, Ignite, pandas, OpenCV.
- Additional `DataSet` including image set (a folder including images) useful for computer vision applications.
- Lean project template compared with pure Kedro.

### Cons:

- GUI ([kedro-viz](https://github.com/quantumblacklabs/kedro-viz)) does not provide execution progress monitoring feature.
- Package dependencies which are not used in many cases (e.g. pyarrow) are included in the `requirements.txt` of Kedro.
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

Uses Kubernetes to run pipelines

### Kubeflow Pipelines

https://github.com/kubeflow/pipelines

Works on top of Argo.

### Oozie

https://github.com/apache/oozie

Manages Hadoop jobs.


### Azkaban

https://github.com/azkaban/azkaban

Manages Hadoop jobs.


## References:

- Airflow
    - https://github.com/apache/airflow
    - https://airflow.apache.org/docs/stable/howto/initialize-database.html
    - https://medium.com/datareply/integrating-slack-alerts-in-airflow-c9dcd155105
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
    - https://docs.metaflow.org/metaflow/scaling
    - https://medium.com/bigdatarepublic/a-review-of-netflixs-metaflow-65c6956e168d
- Kedro
    - https://github.com/quantumblacklabs/kedro
    - https://kedro.readthedocs.io/en/latest/03_tutorial/04_create_pipelines.html
    - https://kedro.readthedocs.io/en/latest/kedro.io.html#data-sets
    - https://medium.com/mhiro2/building-pipeline-with-kedro-for-ml-competition-63e1db42d179
- PipelineX
    - https://github.com/Minyus/pipelinex
- Airflow vs Luigi
    - https://towardsdatascience.com/data-pipelines-luigi-airflow-everything-you-need-to-know-18dc741449b7
    - https://medium.com/better-programming/airbnbs-airflow-versus-spotify-s-luigi-bd4c7c2c0791
    - https://www.quora.com/Which-is-a-better-data-pipeline-scheduling-platform-Airflow-or-Luigi
 
## Inaccuracies

Please kindly let me know if you find anything inaccurate. 

Pull requests for https://github.com/Minyus/Python_Packages_for_Pipeline_Workflow/blob/master/README.md are welcome.
