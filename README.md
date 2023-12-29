# TakeHome
**Brainstorming**:
**Approach 1**: Fine-tune a large language model for job description classification, I  would train the model on a dataset where each job description is followed by its category label. This training teaches the model to predict the category as the next token after a job description. Once fine-tuned, the model can then classify new job descriptions by predicting the category based on the input text. This approach leverages the model's language understanding capabilities for a specific classification task. This approach is best suitable when the dataset does not change much and the categories are constant. It works well when the test and train datasets are from the same distribution

**Approach 2**: Utilize a vector database to manage embeddings of job descriptions. It involves generating vector embeddings for each description, storing them in a vector database, and then, for a new job description, creating its embedding and querying the database to retrieve the top N semantically similar records. This method leverages the database's ability to efficiently perform similarity searches in a high-dimensional space, offering relevant matches based on the semantic content of the job descriptions. This approach works well when the categories are not constant and the dataset is dynamic.

**Assumptions:** 
The dataset is dynamic, as the job postings during various parts of the year may vary.
There is no fixed set of categories into which all the job postings may fall into.
There are currently no ways to estimate the relevant documents out of the retrieved ones.

**Findings:** There are close to 700 categories in train there are 709 records in the test and there are only 602 categories that overlap. Assuming a fixed set of categories might be counterintuitive in our case. Using a fine-tuned model may give a better understanding of the context and might provide better results. Still, when it comes to handling out-of-distribution categories in the test data, it is going to overfit categories in the train. 

Another issue is that all categories are not equally represented and fine-tuning it to an imbalanced dataset will possibly lead to the under-representation of some classes, therefore skewing the results.

Retrieval system might be a better approach as we have to handle dynamic data and it isn't easy to fine-tune every time we have new data. A retrieval system is a cost-efficient approach and easier to implement than fine-tuning. Specific to the use case, it is more reasonable to use a retrieval system to handle dynamic data and the out-of-distribution test set.

**Implementation:** The input is the title of the job description appended to the body. The output is the ONET_NAME. All the input from the training data is encoded using sentence transformer embeddings and stored in an FAISS index. At inference time, use the test title and job description, retrieve top N documents that are semantically similar to the one provided, and return their labels as the predictions.

**Evaluation:** 
**Top K accuracy:** Accuracy is calculated if the ground truth category is found within the top K retrieved categories. K ranges from 1-5. However, this metric does not penalize based on rank. 

**Mean Reciprocal Rank:** This metric ranks the prediction based on which position among the retrieved records is the ground truth present. 

**Custom Evaluation Metric:** All the above metrics work well, given the assumption that the test is not out of distribution. In our case, the test being out of distribution penalizing the scores, and giving these data points zero might skew the final results. To handle this case, I propose a custom metric that will penalize the retrieved records only if they are not from the super category(the first two numbers of the ONET).

**Results:** The retrieval system has a mean reciprocal rank of 0.726 and a mean of 0.733 on the custom metric proposed. The Top K accuracy of k=5 is 0.648 and k =1 is 0.4. The retrieval system is able to predict the right category in 73% of the cases where the category is in the training  set and the super category right when the category is not present in the training. 

**Final Conclusion:** The next steps would be to estimate how dynamic the dataset is over a period of time. Based on the findings, evaluate if the retrieval system is better than fine-tuning.  Given more time, I would investigate on the categories more and see if the test set is always going to be out of distribution and come up with a more robust metric that considers the out-of-distribution test set. In terms of scaling this model, is not as difficult as scaling a fine-tuned model. If a new data point has to be added, adding the embedding of the new data point to the index is all that has to be done. In terms of scaling, a retrieval system is the easiest way to go. Finding better evaluation metrics based on a larger dataset could help improve the performance better.








