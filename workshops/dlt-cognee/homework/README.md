# Homework: From REST to reasoning: ingest, index, and query with dlt and Cognee

# Homework

## Question 1. dlt Version

In this homework, we will load the data from our FAQ to Qdrant

Let's install dlt with Qdrant support and Qdrant client:

```bash
pip install -q "dlt[qdrant]" "qdrant-client[fastembed]"
```

What's the version of dlt that you installed?


## Solution:

```bash
uv python pin 3.12
uv init --bare
uv add "dlt[qdrant]" "qdrant-client[fastembed]"
source .venv/bin/activate 
dlt --version  
```

Output:

```
dlt 1.12.3
```

## Answer 1:

1.12.3

## Question 2. dlt pipeline

Now let's create a pipeline. 

We need to define a destination for that. Let's use the `qdrant` one:

```python
from dlt.destinations import qdrant

qdrant_destination = qdrant(
  qd_path="db.qdrant", 
)
```

In this case, we tell dlt (and Qdrant) to create a folder with
our data, and the name for it will be `db.qdrant`

Let's run it:

```python
pipeline = dlt.pipeline(
    pipeline_name="zoomcamp_pipeline",
    destination=qdrant_destination,
    dataset_name="zoomcamp_tagged_data"

)
load_info = pipeline.run(zoomcamp_data())
print(pipeline.last_trace)
```

How many rows were inserted into the `zoomcamp_data` collection?

Look for `"Normalized data for the following tables:"` in the trace output.

## Solution:

[link](main.ipynb)

```
Step normalize COMPLETED in 0.04 seconds.
Normalized data for the following tables:
- _dlt_pipeline_state: 1 row(s)
- zoomcamp_data: 948 row(s)
```

## Answer 2:

948

## Question 3. Embeddings

When inserting the data, an embedding model was used. Which one?

You can find this out by inspecting the `meta.json` file created
in the target folder.

## Answer 3:

fast-bge-small-en