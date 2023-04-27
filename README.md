# Project Description
## Introduction
- This project is about creating and improving a QA Pipeline. The two-stage process of first retrieving relevant documents and then extracting answers from them is the basis for many modern QA systems, including semantic search engines, intelligent assistants, and automated information extractors.
- For this particular project, I used a subset of SubjQA, which consists of more than 10,000 customer review in English about products and services in six domains: books, electronics, grocery, movies, restaurants, and tripadvisor. I chose this dataset as it provides the start and span of where the answer to the question is found (required for training the Reader). It is found on Kaggle under this URL: https://www.kaggle.com/datasets/arashnic/subjqa-question-answering-dataset
- It is important to note that most of the questions and answers are subjective, meaning they depend on the personal experience of the users. In some cases, important parts of the question do not appear in the review at all, so shortcuts like keyword search might not be an option. This makes it a realistic dataset to benchmark review-based QA models.

## Haystack and Elasticsearch
- Modern QA systems are typically based on the retriever-reader architecture. The retriever is responsible for retrieving relevant documents for a given query. The reader is responsible for extracting an answer fro mthe documents provided by the retriever. The reader is usually a reading comprehension model, although some models can generate free-form answers.
- Haystack is an important tool for building a QA system. It is based on the retriever-reader architecture, abstracts much of the complexity involved in building these systems, and integrates tightly with Hugging Face Transformers.
- In addition to the retriever and reader, there are two more components involved when building a QA pipeline with Haystack: 
  - The document store, which is a document-oriented database that stores documents and metadata
  - A pipeline that combines all components of a QA system 
### Initializing the Components
- Elasticsearch is a search engine that is capable of handlng a diverse range of data types. its ability to store huge volumes of data and quickly filter it with full-text search makes it well suited for developing QA systems. To initialize the document store, make sure to run Elasticsearch on your computer. The simplest option is using Docker:

  docker run -d -p 9200:9200 -e "discovery.type=single-node" elasticsearch:7.9.2
- The following sections will focus on the Python code in this repository. In the Python environment, make sure everything is up and running by sending a small HTTP request to the localhost. With the server running, instantiate the document store, retriever and reader.
- At first, I will use BM25 as retriever which is an improved version of the classic Term Frequency-Inverse Document Frequency (TF-iDF) algorithm. It represents the question and context as sparse vectors and relevance of a query and context is then determined by computing the inner product of the two vectors.
- As the reader, I am using FARMReader, which is based on deepset's (developed Haystack) FARM framework. Another possibility is the TransformersReader, which is based on the QA pipeline from Hugging Face. Although both readers handle a model's weight the same, Hugging Face's pipeline normalizes the start and end logits with a softmax in each passage.
- As a consequence, it is only meaningful to compare answer scores between answers extracted from the same passage, where the probabilities sum to 1. For example, an answer score of 0.9 from one passage is not necessarily better than a score of 0.8 from another. In FARM, the logits are not normalized, so inter-passage answers can be compared more easily.
- I also made a first prediction. The first answer fits the question, the second does not produce anything and the third is off-topic. In the next section, I will make a baseline evaluation with more qualitative metrics.

## Baseline Evaluation
- It does not matter how good the reader is if the retriever cannot find the relevant documents in the first place. The retriever sets an upper bound on the performance of the whole QA system. There are some common metrics that can be used to evaluate the retriever.
- A common metric for evaluating retrievers is recall, which measures the fraction of all relevant documents that are retrieved. In this context, relevant means whether the answer is present in a passage of text or not.
- Haystack provides a Label object that contains all the information relevant to a document retrieval. These labels can be passed to the document store and are generally used for evaluation. The code is looping over each question in the test set and extracts the matching answers and additional metadata. These labels can now be used to evaluate the pipeline. The retriever actually does not perform too bad with a score of 0.67.
- When it comes to the reader, two main metrics are used:
  - Exact Match (EM): a binary metric	that returns 1 if the characters in the predicted and ground truth answers match exactly, and 0 otherwise. If no answer is expected, the EM is 0 if it predicts any text at all
  - F1-score: Measures the harmonic mean of the precision and recall
- The results of the retriever


