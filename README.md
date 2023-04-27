# Project description
## Introduction
- This project is about creating and improving a QA pipeline. The two-step process of first retrieving relevant documents and then extracting answers from them is the basis for many modern QA systems, including semantic search engines, intelligent wizards, and automated information extractors.
- For this particular project, I used a subset of SubjQA consisting of more than 10,000 customer reviews in English about products and services in six domains: Books, Electronics, Food, Movies, Restaurants, and Tripadvisor. I chose this dataset because it contains the beginning and range of the answer to the question (required for reader training). It can be found on Kaggle at this URL: https://www.kaggle.com/datasets/arashnic/subjqa-question-answering-dataset
- It is important to note that most questions and answers are subjective, i.e., they depend on the personal experiences of the users. In some cases, important parts of the question do not appear in the assessment at all, so shortcuts such as keyword searches may not be possible. This makes it a realistic dataset for benchmarking review-based QA models.

## Haystack and Elasticsearch.
- Modern QA systems are usually based on a retriever-reader architecture. The retriever is responsible for retrieving relevant documents for a given query. The Reader is responsible for extracting a response from the documents provided by the Retriever. The Reader is usually a reading comprehension model, although some models can generate free-form responses.
- Haystack is an important tool for building a QA system. It is based on the retriever-reader architecture, abstracts away much of the complexity associated with building such systems, and is tightly integrated with Hugging Face Transformers.
- In addition to the Retriever and Reader, there are two other components involved in building a QA pipeline with Haystack: 
  - The Document Store, a document-oriented database that stores documents and metadata.
  - A pipeline that combines all the components of a QA system 
### Initialization of the components
- Elasticsearch is a search engine capable of handling a wide range of data types. Its ability to store large amounts of data and filter it quickly with a full-text search makes it well suited for QA system development. To initialize the document store, you need to run Elasticsearch on your computer. The easiest way is to use Docker:

  docker run -d -p 9200:9200 -e "discovery.type=single-node" elasticsearch:7.9.2
- The following sections deal with the Python code in this repository. In the Python environment, make sure everything is running by sending a small HTTP request to localhost. Once the server is running, instantiate the document repository, retriever, and reader.
- First, I'll use BM25 as the retriever, which is an improved version of the classic Term Frequency-Inverse Document Frequency (TF-iDF) algorithm. It represents the question and context as sparse vectors, and the relevance of a question and context is then determined by computing the inner product of the two vectors.
- As a reader, I use FARMReader, which is based on deepset's FARM framework (developed by Haystack). Another option is the TransformersReader, which is based on Hugging Face's QA pipeline. Although both readers handle the weighting of a model in the same way, Hugging Face's pipeline normalizes the start and end logits (non-normalized prediction) with a softmax in each passage.
- Consequently, it is only useful to compare response values between responses extracted from the same passage when the sum of probabilities equals 1. For example, a response value of 0.9 from one passage is not necessarily better than a value of 0.8 from another. In FARM, logits are not normalized, so it is easier to compare responses between passages.
- I also made an initial prediction. The first answer fits the question, the second yields nothing, and the third is off-topic. In the next section, I will do a basic evaluation with more qualitative metrics.

## Baseline Evaluation
- It doesn't matter how good the reader is if the retriever can't find the relevant documents in the first place. The retriever sets an upper bound on the performance of the entire QA system. There are some common metrics that can be used to evaluate the retriever.
- One common metric used to evaluate retrievers is recall, which measures the percentage of all relevant documents that are found. In this context, relevant means whether or not the answer is present in a passage of text.
- Haystack provides a Label object that contains all information relevant to a document retrieval. These labels can be passed to the document store and are generally used for evaluation. The code loops over each question in the test set and extracts the appropriate answers and additional metadata. These labels can now be used for pipeline scoring. With a score of 0.67, the retriever doesn't do too badly.
- For the reader, two metrics are mainly used:
  - Exact Match (EM): a binary metric that returns 1 if the characters in the predicted and actual response match exactly, and 0 otherwise. If no response is expected, the EM is 0 if it predicts any text at all.
  - F1 evaluation: Measures the harmonic mean of Precision and Recall.
- Retriever evaluation results are intermediate, ranging from 0.4 to 0.5 for both scores.

### Dense Passage Retrieval
- A known limitation of sparse retrievers such as B25 is that they cannot capture the relevant documents if the user query contains terms that do not exactly match those in the review. An alternative is to use dense embeddings to represent the query and the document. One popular architecture is known as dense passage retrieval (DPR). The main idea behind DPR is to use two BERT models as encoders for the question and the passage, something like this:

  ![image](https://user-images.githubusercontent.com/127037803/234890772-09b6f10b-70c3-4d1c-8587-e298d8b913b6.png)
- In Haystack, a retriever for DPR is initialized in a similar manner as the process for BM25. In addition to specifying the document repository, the BERT coders for the question and passage are also selected. These coders are trained by giving them questions with relevant (positive) passages and irrelevant (negative) passages, the goal being to learn that relevant question-passage pairs have higher similarity.
- In this case, however, the use of DPR did not improve performance. However, it is good practice to use it as a default choice.

## Fine tuning and inference
- The easiest way to improve the reader is to further fine tune the MiniLM model on the subjQA training set. The FARMReader has a train() method provided for this purpose, which expects the data in SQuAD JSON format, where all QA pairs are grouped. Another function was required for this conversion.
- Once the two JSON files were created, the training could begin. Training the model definitely resulted in an increase in performance, as shown below:

  ![image](https://user-images.githubusercontent.com/127037803/234892314-56fb4e4b-65ce-4f3e-9a6c-ca4cbc03d006.png)
- As a final step, I wanted to see if there was any notable difference in the answers that would be returned by the Reader. The first answer is very appropriate, the second didn't return anything, and the last one also agrees with the question, so I definitely see an improvement over the old Reader.

## Conclusion
- I really enjoyed this project because I learned about many different tools, such as Haystack. I have never worked with Haystack before, but the hours I spent learning the framework really broadened my horizons when it comes to NLP. The biggest challenge was finding the right data set. I didn't realize that fine-tuning the reader would require information about where to find the answer in the text. I started the project with a different data set when I realized it didn't contain that information. Fortunately, it wasn't too much work to adjust the code.
- On Kaggle, I experimented with many different Hugging Face models, comparing and fine-tuning them. I'm really grateful that Kaggle gives everyone 30 hours of GPU, so I was able to try many different things (for other NLP tasks). However, I had to initialize the document store on my computer, so of course I fine-tuned the model with my own GPU. For this reason, I didn't spend too much time on the fine-tuning, which only resulted in a medium performance increase. 
- However, with this project I wanted to go deeper into the QA pipeline, and domain tuning is just one part of it. In the future, I will probably take further steps towards QA pipeline development, as I think information gathering is a very valuable task to master.



