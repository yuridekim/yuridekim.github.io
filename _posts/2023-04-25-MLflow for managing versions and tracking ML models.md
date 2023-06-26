---
title : MLflow for managing versions and tracking ML models
date : 2023-04-25 00:09:00 +0900
categories : [ML]
tags : [ML, tools, mlops] #소문자만 가능
pinned : 0
---

# What is MLflow?
MLflow is an open source platform used for tracking, logging and managing an end-to-end machine learning lifecycle.

## Log 
To keep track of your model versions and determine which among them are best, log necessary information.

- Parameters: Static values that do not change during the run. E.g. batch size, number of epochs
- Artifact: Resources that you want to store. E.g. The model itself, text files that the run produced
- Metric: Values that change with time. E.g. training loss,

```python
import mlflow
from mlflow import log_param, log_artifact, log_metric
import mlflow.pytorch

mlflow.start_run()
log_param("batch size", batch_size)
log_artifact("path/name/to/artifact")
log_metric("training loss", train_loss)
mlflow.end_run()
```

## Track with MLflow

All of the files needed to run the ui should be stored in the `mlruns` and `mlartifacts` directory.

```bash
mlflow ui
```
Now the entire log of Experiments should be available on localhost:5000.

## MLOps
So what is the essential difference between MLFlow and other services that offer similar functionalities to keep track of the machine learning lifecycle?

MLFlow is deployable on top of Cloud environments which makes it easier to access, regardless of your work context. While you are away from your desk, you'll be able to check in on the progress of the machine learning model, for which the training can be extensively lengthy most times.