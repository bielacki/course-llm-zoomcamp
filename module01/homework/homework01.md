# Module 01 homework

## Q1. Running Elastic

Run Elastic Search 8.17.6, and get the cluster information. If you run it on localhost, this is how you do it:
```bash
curl localhost:9200
```
What's the version.build_hash value?

### Solution:

```bash
docker container run -it \                                          
    --rm \
    --name elasticsearch \
    -m 4GB \
    -p 9200:9200 \
    -p 9300:9300 \
    -e "discovery.type=single-node" \
    -e "xpack.security.enabled=false" \
    docker.elastic.co/elasticsearch/elasticsearch:8.17.6

curl http://localhost:9200
```

Output:
```bash
{
  "name" : "3e48c0bbcdf8",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "nVeofXiHTreywzhtLuQUAA",
  "version" : {
    "number" : "8.17.6",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "dbcbbbd0bc4924cfeb28929dc05d82d662c527b7",
    "build_date" : "2025-04-30T14:07:12.231372970Z",
    "build_snapshot" : false,
    "lucene_version" : "9.12.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

### Answer 1

dbcbbbd0bc4924cfeb28929dc05d82d662c527b7


## Q2. Indexing the data

Index the data in the same way as was shown in the course videos. Make the course field a keyword and the rest should be text.

Don't forget to install the ElasticSearch client for Python:
```
pip install elasticsearch
```
Which function do you use for adding your data to elastic?

- insert
- index
- put
- add

### Answer 2

index

## Q3. Searching

Now let's search in our index.

We will execute a query "How do execute a command on a Kubernetes pod?".

Use only question and text fields and give question a boost of 4, and use "type": "best_fields".

What's the score for the top ranking result?

- 64.50
- 84.50
- 44.50
- 24.50

Look at the _score field.

### Solution 3:

```python
import requests
from elasticsearch import Elasticsearch

docs_url = 'https://github.com/DataTalksClub/llm-zoomcamp/blob/main/01-intro/documents.json?raw=1'
docs_response = requests.get(docs_url)
documents_raw = docs_response.json()

documents = []

for course in documents_raw:
    course_name = course['course']

    for doc in course['documents']:
        doc['course'] = course_name
        documents.append(doc)

es_client = Elasticsearch('http://localhost:9200')
es_client.info()

index_settings = {
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
    },
    "mappings": {
        "properties": {
            "text": {"type": "text"},
            "section": {"type": "text"},
            "question": {"type": "text"},
            "course": {"type": "keyword"} 
        }
    }
}

index_name = "course-questions"

es_client.indices.create(
    index=index_name,
    body=index_settings
)

for document in documents:
    es_client.index(
        index=index_name,
        document=document
    )

query = 'How do execute a command on a Kubernetes pod?'

def elastic_search(query):

    search_query = {
        "size": 5,
        "query": {
            "bool": {
                "must": {
                    "multi_match": {
                        "query": query,
                        "fields": ["question^4", "text"],
                        "type": "best_fields"
                    }
                },
                # "filter": {
                #     "term": {
                #         "course": "data-engineering-zoomcamp"
                #     }
                # }
            }
        }
    }

    response = es_client.search(
        index=index_name,
        body=search_query
    )

    return response

results = elastic_search(query)

print(results['hits']['hits'][0]['_score'])
```

Output:

```
44.50556
```

### Answer 3:

44.50

## Q4. Filtering

Now ask a different question: "How do copy a file to a Docker container?".

This time we are only interested in questions from `machine-learning-zoomcamp`.

Return 3 results. What's the 3rd question returned by the search engine?

- How do I debug a docker container?
- How do I copy files from a different folder into docker container’s working directory?
- How do Lambda container images work?
- How can I annotate a graph?

### Solution 4:

```python
import requests
from elasticsearch import Elasticsearch

docs_url = 'https://github.com/DataTalksClub/llm-zoomcamp/blob/main/01-intro/documents.json?raw=1'
docs_response = requests.get(docs_url)
documents_raw = docs_response.json()

documents = []

for course in documents_raw:
    course_name = course['course']

    for doc in course['documents']:
        doc['course'] = course_name
        documents.append(doc)

es_client = Elasticsearch('http://localhost:9200')
es_client.info()

index_settings = {
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
    },
    "mappings": {
        "properties": {
            "text": {"type": "text"},
            "section": {"type": "text"},
            "question": {"type": "text"},
            "course": {"type": "keyword"} 
        }
    }
}

index_name = "course-questions"

es_client.indices.create(
    index=index_name,
    body=index_settings
)

for document in documents:
    es_client.index(
        index=index_name,
        document=document
    )

query = 'How do copy a file to a Docker container?'

def elastic_search(query):

    search_query = {
        "size": 3,
        "query": {
            "bool": {
                "must": {
                    "multi_match": {
                        "query": query,
                        "fields": ["question^4", "text"],
                        "type": "best_fields"
                    }
                },
                "filter": {
                    "term": {
                        "course": "machine-learning-zoomcamp"
                    }
                }
            }
        }
    }

    response = es_client.search(
        index=index_name,
        body=search_query
    )

    return response

results = elastic_search(query)

results['hits']['hits'][2]['_source']['question']
```

Output:

```
'How do I copy files from a different folder into docker container’s working directory?'
```

### Answer 4:

How do I copy files from a different folder into docker container’s working directory?

## Q5. Building a prompt

Now we're ready to build a prompt to send to an LLM.

Take the records returned from Elasticsearch in Q4 and use this template to build the context. Separate context entries by two linebreaks (\n\n)

```
context_template = """
Q: {question}
A: {text}
""".strip()
```

Now use the context you just created along with the "How do copy a file to a Docker container?" question to construct a prompt using the template below:

```
prompt_template = """
You're a course teaching assistant. Answer the QUESTION based on the CONTEXT from the FAQ database.
Use only the facts from the CONTEXT when answering the QUESTION.

QUESTION: {question}

CONTEXT:
{context}
""".strip()
```

What's the length of the resulting prompt? (use the len function)

- 946
- 1446
- 1946
- 2446


### Solution 5:

```python

...
results = elastic_search(query)

def build_prompt(query, search_results):

    prompt_template = """
You're a course teaching assistant. Answer the QUESTION based on the CONTEXT from the FAQ database.
Use only the facts from the CONTEXT when answering the QUESTION.

QUESTION: {question}

CONTEXT:
{context}
""".strip()

    context_template = "".strip()

    for doc in search_results:
        context_template = context_template + f"Q: {doc['question']}\nA: {doc['text']}\n\n"

    prompt = prompt_template.format(
        question=query,
        context=context_template
    ).strip()

    return prompt

prompt = build_prompt(query, results)

print(prompt)
print((prompt))
```

Output:

```
You're a course teaching assistant. Answer the QUESTION based on the CONTEXT from the FAQ database.
Use only the facts from the CONTEXT when answering the QUESTION.

QUESTION: How do copy a file to a Docker container?

CONTEXT:
Q: How do I debug a docker container?
A: Launch the container image in interactive mode and overriding the entrypoint, so that it starts a bash command.
docker run -it --entrypoint bash <image>
If the container is already running, execute a command in the specific container:
docker ps (find the container-id)
docker exec -it <container-id> bash
(Marcos MJD)

Q: How do I copy files from my local machine to docker container?
A: You can copy files from your local machine into a Docker container using the docker cp command. Here's how to do it:
To copy a file or directory from your local machine into a running Docker container, you can use the `docker cp command`. The basic syntax is as follows:
docker cp /path/to/local/file_or_directory container_id:/path/in/container
Hrithik Kumar Advani

Q: How do I copy files from a different folder into docker container’s working directory?
A: You can copy files from your local machine into a Docker container using the docker cp command. Here's how to do it:
In the Dockerfile, you can provide the folder containing the files that you want to copy over. The basic syntax is as follows:
COPY ["src/predict.py", "models/xgb_model.bin", "./"]											Gopakumar Gopinathan
1446
```

### Answer 5:

1446

## Q6. Tokens

When we use the OpenAI Platform, we're charged by the number of tokens we send in our prompt and receive in the response.

The OpenAI python package uses tiktoken for tokenization:

```bash
pip install tiktoken
```

Let's calculate the number of tokens in our query:

```bash
encoding = tiktoken.encoding_for_model("gpt-4o")
```

Use the encode function. How many tokens does our prompt have?

- 120
- 220
- 320
- 420

Note: to decode back a token into a word, you can use the decode_single_token_bytes function:

```bash
encoding.decode_single_token_bytes(63842)
```

### Solution 6:

```bash
...
import tiktoken

encoding = tiktoken.encoding_for_model("gpt-4o")

print(encoding.encode(prompt))
print(f"\ntokens: {len(encoding.encode(prompt))}")
```

Output:

```
[63842, 261, 4165, 14029, 29186, 13, 30985, 290, 150339, 4122, 402, 290, 31810, 8099, 591, 290, 40251, 7862, 558, 8470, 1606, 290, 19719, 591, 290, 31810, 8099, 1261, 55959, 290, 150339, 364, 107036, 25, 3253, 621, 5150, 261, 1974, 316, 261, 91238, 9282, 1715, 10637, 50738, 734, 48, 25, 3253, 621, 357, 15199, 261, 62275, 9282, 3901, 32, 25, 41281, 290, 9282, 3621, 306, 25383, 6766, 326, 151187, 290, 7251, 4859, 11, 813, 484, 480, 13217, 261, 38615, 6348, 558, 68923, 2461, 533, 278, 2230, 7962, 4859, 38615, 464, 3365, 523, 3335, 290, 9282, 382, 4279, 6788, 11, 15792, 261, 6348, 306, 290, 4857, 9282, 734, 68923, 10942, 350, 6555, 290, 9282, 26240, 446, 68923, 25398, 533, 278, 464, 6896, 26240, 29, 38615, 198, 6103, 277, 10732, 391, 79771, 1029, 48, 25, 3253, 621, 357, 5150, 6291, 591, 922, 2698, 7342, 316, 62275, 9282, 3901, 32, 25, 1608, 665, 5150, 6291, 591, 634, 2698, 7342, 1511, 261, 91238, 9282, 2360, 290, 62275, 27776, 6348, 13, 44257, 1495, 316, 621, 480, 734, 1385, 5150, 261, 1974, 503, 12552, 591, 634, 2698, 7342, 1511, 261, 6788, 91238, 9282, 11, 481, 665, 1199, 290, 2700, 68923, 27776, 6348, 62102, 623, 9439, 45440, 382, 472, 18183, 734, 68923, 27776, 820, 4189, 72231, 52214, 51766, 15400, 35850, 9282, 1537, 27975, 4189, 26985, 190543, 198, 106096, 437, 507, 70737, 15241, 3048, 279, 48, 25, 3253, 621, 357, 5150, 6291, 591, 261, 2647, 15610, 1511, 62275, 9282, 802, 4113, 12552, 3901, 32, 25, 1608, 665, 5150, 6291, 591, 634, 2698, 7342, 1511, 261, 91238, 9282, 2360, 290, 62275, 27776, 6348, 13, 44257, 1495, 316, 621, 480, 734, 637, 290, 91238, 2318, 11, 481, 665, 3587, 290, 15610, 15683, 290, 6291, 484, 481, 1682, 316, 5150, 1072, 13, 623, 9439, 45440, 382, 472, 18183, 734, 128701, 9129, 7205, 8138, 21369, 17311, 672, 392, 13123, 22739, 9320, 10928, 69422, 672, 9633, 2601, 14973, 22713, 167296, 30463, 499, 137058, 22064]

tokens: 320
```

### Answer 6:

320

## Bonus: generating the answer (ungraded)

Let's send the prompt to OpenAI. What's the response?

### Answer:


You can copy a file from your local machine into a Docker container using the `docker cp` command. The basic syntax for copying a file or directory into a running Docker container is as follows:

```bash
docker cp /path/to/local/file_or_directory container_id:/path/in/container
```

## Bonus: calculating the costs (ungraded)

Suppose that on average per request we send 150 tokens and receive back 250 tokens.

How much will it cost to run 1000 requests?

You can see the prices [here](https://openai.com/api/pricing/)

On June 17, the prices for gpt4o are:

- Input: $0.005 / 1K tokens
- Output: $0.015 / 1K tokens

You can redo the calculations with the values you got in Q6 and Q7.

### Answer:

(150 * 0.005) + (250 * 0.015) = $4.5

