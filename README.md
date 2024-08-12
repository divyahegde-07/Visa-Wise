
# USCIS chatbot using RAG system 

## Motivation
Given the importance of understanding visa regulations for maintaining legal status, avoiding penalties, and ensuring a smooth academic journey, it is imperative to develop a solution that simplifies these complexities. 

## Architecture 

The core idea behind RAG is to leverage the vast amount of information available in external knowledge bases to supplement the generative capabilities of models like Llama 3-instruct, LLM used in this project.

RAG has 2 main parts - one part that retrieves the context by searching a large corpus of doucments to find relevant information and the other part that generates response using the fetched context from the retriever.

![Screenshot 1](https://raw.github.com/divyahegde-07/Visa-Wise/main/Screenshots/Screenshot%202024-08-12%20at%2000.52.05.png)

![Screenshot 1](https://raw.github.com/divyahegde-07/Visa-Wise/main/Screenshots/Screenshot%202024-08-12%20at%2000.52.18.png)

Retriever - The documents (could be PDF, PPT, even images and videos) are chunked using chunking strategy. Chunking is important since it decides how many tokens are consdiered as a single sentence and how much overlap should be there between 2 sentences. For this project, 1024 and 2048 chunking sizes were tried and 1024 was found to be performing the best with overlap of 50. Smaller chunk sizes (than 1024) increased retrieval time which wasn't pleasant.
Each chunk is converted into dense vector embeddings using models like BAAI/bge-base-en-v1.5. These embeddings capture the semantic meaning of the text, enabling effective similarity searches during retrieval.

To store these high-dimension vectors, vector databases like Pinecone, Chroma can be used. Here, Pinecone is used with a vector dimention of 768. These vector DBs are really easy to integrate with frameworks like LlamaIndex.

For the generation part, Llama 3-instruct-8B LLM is used along with a propmt template and the context from retriever.
 
## Experimentations and analysis
First experiment performed was on chunking strategy. It was found that 1024 chunk size with 100 overlap performed the best in terms of retreival time.

For a RAG system to be effective, it should get all the relevant information. Now this is where retrievers are important. So I was interested in improving the accuracy of context. Initially, only vector search retrieval was used where it would search based on the semantic similarity. This did perform well when a query was asked but missed some key pieces in some responses. 
So hybrid retrieval was tried - combining vector seach and keyword search using BM25. It was then observed that in some responses, the keyword search messed up the context as more or equal weightage was given to both retreievers. This further led me to explore weight tuning and using COHERE re-ranker model after retreival. The context provided to the generator was much better. 

The RAG system was evaluated on both these retrieval techniques using the RAGAS framework. This was a qunatitative evaluation done by syntethically generating questions using generator from RAGAS.
Qualitative analysis was done by providing some context based questions and manually going over the responses for different weights of retrievers. 


## Results

- RAGAS evaluation score for vector search only retrieval

![Screenshot 1](https://raw.github.com/divyahegde-07/Visa-Wise/main/Screenshots/Screenshot%202024-08-12%20at%2000.29.07.png)

- RAGAS evaluation score for hybrid search retrieval
![Screenshot 1](https://raw.github.com/divyahegde-07/Visa-Wise/main/Screenshots/Screenshot%202024-08-12%20at%2000.29.18.png)

- RAGAS evaluation score for different weights in hybrid search retrieval

![Screenshot 2](https://raw.github.com/divyahegde-07/Visa-Wise/main/Screenshots/Screenshot%202024-08-11%20at%2016.04.05.png)

- Qualitative evaluation of responses for different weights in hybrid search retrieval
![Screenshot 3](https://raw.github.com/divyahegde-07/Visa-Wise/main/Screenshots/Screenshot%202024-08-11%20at%2015.54.56.png)

## Conclusion

Hybrid search retrieval is preferred in RAG because it blends the best of both worlds: dense retrieval, which captures the overall meaning of a query, and sparse retrieval, which focuses on exact keyword matches. By combining these approaches, hybrid search can find a wider range of relevant information and provide more accurate results. This makes it better at handling different types of queries, whether they need a broad understanding or specific details, leading to a more reliable and user-friendly search experience as seen in the examples in this project.

Weight tuning in hybrid search retrieval is key to getting the best results. It helps balance the influence of dense and sparse retrieval methods, so the system can deliver more relevant and accurate information. By adjusting the weights, you ensure the search results fit the specific needs of different queries, improving overall performance and making sure you get the information you're really looking for.
