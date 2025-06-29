# Module 02 homework

## Q1. Embedding the query

Embed the query: `'I just discovered the course. Can I join now?'`.
Use the `'jinaai/jina-embeddings-v2-small-en'` model. 

You should get a numpy array of size 512.

What's the minimal value in this array?

* -0.51
* -0.11
* 0
* 0.51

### Solution:

```python

from qdrant_client import QdrantClient, models
from fastembed import TextEmbedding

client = QdrantClient("http://localhost:6333")

EMBEDDING_DIMENSIONALITY = 512
model_handle = "jinaai/jina-embeddings-v2-small-en"
query = 'I just discovered the course. Can I join now?'

# Define the collection name
collection_name = "zoomcamp-hw02-q01"

# Create the collection with specified vector parameters
client.create_collection(
    collection_name=collection_name,
    vectors_config=models.VectorParams(
        size=EMBEDDING_DIMENSIONALITY,  # Dimensionality of the vectors
        distance=models.Distance.COSINE  # Distance metric for similarity search
    )
)

points = []

point = models.PointStruct(
    id=0,
    vector=models.Document(text=query, model=model_handle), #embed text locally with "jinaai/jina-embeddings-v2-small-en" from FastEmbed
    payload={
        "text": query,
    } #save all needed metadata fields
)
points.append(point)

client.upsert(
    collection_name=collection_name,
    points=points
)

# Get vectors from the collection
vectors = client.retrieve(
    collection_name=collection_name,
    ids=[0],  # we'll retrieve the vector for point with id=0
    with_vectors=True  # include vectors in the response
)

print(min(vectors[0].vector))
```

Output:
-0.117263734


### Answer 1

-0.11

## Q2. Cosine similarity with another vector

Now let's embed this document:

```python
doc = 'Can I still join the course after the start date?'
```

What's the cosine similarity between the vector for the query
and the vector for the document?

* 0.3
* 0.5
* 0.7
* 0.9


### Solution:

```python
q2 = 'Can I still join the course after the start date?'

point = models.PointStruct(
    id=1,
    vector=models.Document(text=q2, model=model_handle), #embed text locally with "jinaai/jina-embeddings-v2-small-en" from FastEmbed
    payload={
        "text": q2,
    } #save all needed metadata fields
)
points.append(point)

client.upsert(
    collection_name=collection_name,
    points=points
)

# Get vectors from the collection
vectors = client.retrieve(
    collection_name=collection_name,
    ids=[0, 1],  # we'll retrieve the vector for point with id=0
    with_vectors=True  # include vectors in the response
)

q1_vector = vectors[0].vector
q2_vector = vectors[1].vector

# Compute cosine similarity between q1_vector and q2_vector
def cosine_similarity(a, b):
    a = np.array(a)
    b = np.array(b)
    return a.dot(b)

similarity = cosine_similarity(q1_vector, q2_vector)
print(f"Cosine similarity between vector 0 and vector 1: {similarity}")
```

Output: Cosine similarity between vector 0 and vector 1: 0.9008528900649803

### Answer 2

0.9

## Q3. Ranking by cosine

For Q3 and Q4 we will use these documents:

```python
documents = [{'text': "Yes, even if you don't register, you're still eligible to submit the homeworks.\nBe aware, however, that there will be deadlines for turning in the final projects. So don't leave everything for the last minute.",
  'section': 'General course-related questions',
  'question': 'Course - Can I still join the course after the start date?',
  'course': 'data-engineering-zoomcamp'},
 {'text': 'Yes, we will keep all the materials after the course finishes, so you can follow the course at your own pace after it finishes.\nYou can also continue looking at the homeworks and continue preparing for the next cohort. I guess you can also start working on your final capstone project.',
  'section': 'General course-related questions',
  'question': 'Course - Can I follow the course after it finishes?',
  'course': 'data-engineering-zoomcamp'},
 {'text': "The purpose of this document is to capture frequently asked technical questions\nThe exact day and hour of the course will be 15th Jan 2024 at 17h00. The course will start with the first  “Office Hours'' live.1\nSubscribe to course public Google Calendar (it works from Desktop only).\nRegister before the course starts using this link.\nJoin the course Telegram channel with announcements.\nDon’t forget to register in DataTalks.Club's Slack and join the channel.",
  'section': 'General course-related questions',
  'question': 'Course - When will the course start?',
  'course': 'data-engineering-zoomcamp'},
 {'text': 'You can start by installing and setting up all the dependencies and requirements:\nGoogle cloud account\nGoogle Cloud SDK\nPython 3 (installed with Anaconda)\nTerraform\nGit\nLook over the prerequisites and syllabus to see if you are comfortable with these subjects.',
  'section': 'General course-related questions',
  'question': 'Course - What can I do before the course starts?',
  'course': 'data-engineering-zoomcamp'},
 {'text': 'Star the repo! Share it with friends if you find it useful ❣️\nCreate a PR if you see you can improve the text or the structure of the repository.',
  'section': 'General course-related questions',
  'question': 'How can we contribute to the course?',
  'course': 'data-engineering-zoomcamp'}]
```

Compute the embeddings for the text field, and compute the 
cosine between the query vector and all the documents.

What's the document index with the highest similarity? (Indexing starts from 0):

- 0
- 1
- 2
- 3
- 4

Hint: if you put all the embeddings of the text field in one matrix `V` (a single 2-dimensional numpy array), then
computing the cosine becomes a matrix multiplication:

```python
V.dot(q)
```

If this hint is rather confusing you than helping, feel free
to ignore it.

### Solution:

[link](/homework/hw02-03.ipynb)

### Answer 3

1

## Q4. Ranking by cosine, version two

Now let's calculate a new field, which is a concatenation of
`question` and `text`:

```python
full_text = doc['question'] + ' ' + doc['text']
``` 

Embed this field and compute the cosine between it and the
query vector. What's the highest scoring document?

- 0
- 1
- 2
- 3
- 4

Is it different from Q3? If yes, why?

### Solution:

[link](/homework/hw02-04.ipynb)

### Answer 4

0, because the question field from the embedding contains the phrase from the query vector.

## Q5. Selecting the embedding model

Now let's select a smaller embedding model.
What's the smallest dimensionality for models in fastembed?

- 128
- 256
- 384
- 512

One of these models is `BAAI/bge-small-en`. Let's use it.

### Solution:

[link](/homework/hw02-05.ipynb)

### Answer 5

384

## Q6. Indexing with qdrant (2 points)

For the last question, we will use more documents.

We will select only FAQ records from our ml zoomcamp:

```python
import requests 

docs_url = 'https://github.com/alexeygrigorev/llm-rag-workshop/raw/main/notebooks/documents.json'
docs_response = requests.get(docs_url)
documents_raw = docs_response.json()


documents = []

for course in documents_raw:
    course_name = course['course']
    if course_name != 'machine-learning-zoomcamp':
        continue

    for doc in course['documents']:
        doc['course'] = course_name
        documents.append(doc)
```

Add them to qdrant using the model form Q5.

When adding the data, use both question and answer fields:

```python
text = doc['question'] + ' ' + doc['text']
```

After the data is inserted, use the question from Q1 for querying the collection.

What's the highest score in the results?
(The score for the first returned record):

- 0.97
- 0.87
- 0.77
- 0.67

### Solution:

[link](/homework/hw02-06.ipynb)

### Answer 6

0.87
