---
layout: post
title: "Spark Summit 2020"
date: 2020-06-28
---

## Spark 3.0
- Adaptive Query Engine: no need to manually configure and tune SQL query parameters
- Python type hints for Pandas UDFs
- Faster Apache Arrow-based calls

## Spark Ecosystem projects
- Koalas 1.0
- Scale-out on Spark (scikit-learn, HyperOpt, joblib)
- Rapids
- Project Zen (e.g. avoid to display lengthy error page including java errors, python errors)
- Delta Lake (building the Structured Transactional Layers)
	- **ACID Transactions** to solve
		- Hard to append data
		- Modification of existing data
		- Jobs failing midway
		- Real-time operations hard
		- Costly to keep historical data versions
	- Indexing to deal with "too many files" problem
	- Schema validation and expectations including Delta Expectation (written and defined in SQL)
- Delta Engine

## Demo
```python
import databricks.koalas as ks
from mlflow.sklearn import load_model
from typing import Iterator
import pandas as pd

ks_df = ks.read_parquet("/dir/big_data.parquet")

spark.conf.set("spark.sql.adaptive.enabled", "true")

spark_df = ks_df.to_spark()
# train a ML model
def predict(iterator: Iterator[pd.DataFrame]) -> Iterator[pd.DataFrame]:
	model = load_model("models:/lr-model/1")
	for feature in iterator:
		prediction = pd.Seres(model.predict(features[["n_steps", "n_ingredients"]]), name="predicted_minutes")
		yield pd.concat([features, predictions], axis=1)

spark_df.mapInPandas(predict, "name STRING, description STRING, n_steps INT n_ingredients INT, minutes INT, predicted_minutes DOUBLE")
```

## Disrupting risk managment through emerging technologies
- Execute multiple experiments at the push of a buttom
- What if analysis

### Use case: get insights at your fingertips
A super cool application built within Capital One!
User making request -> slack bot -> AWS SQS -> AWS Lambda -> ML model -> execution result to S3 -> Databricks for DA to analyze -> send back results

## Spark SQL with Pandas UDFs
- Scalar UDFs: one-to-one mapping function
- Grouped Map UDFs -> can return complicated return types
Jupyter notebooks, use **magic commands** such as `timeit`, `time`, `prun`


## Koalas
Exciting tool for Pandas users like myself to seamlessly switch ove to Spark. All starts with the following simple commands.
```python
import databricks.koalas as ks
ks.set_option('compute.default_index_type', 'distributed') # not explicitly require an index that monotonically increases one-by-one
```
