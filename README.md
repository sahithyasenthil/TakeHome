# TakeHome
###**Brainstorming**:
####**Approach 1**: Fine-tune a large language model for job description classification, I  would train the model on a dataset where each job description is followed by its category label. This training teaches the model to predict the category as the next tokens after a job description. Once fine-tuned, the model can then classify new job descriptions by predicting the category based on the input text. This approach leverages the model's language understanding capabilities for a specific classification task. This approach is best suitable when the dataset does not change much and the categories are constant. It workes well when the test and train dataset are from the same distribution

####**Approach 2**: Utilize a vector database to manage embeddings of job descriptions. It involves generating vector embeddings for each description, storing them in a vector database, and then, for a new job description, creating its embedding and querying the database to retrieve the top N semantically similar records. This method leverages the database's ability to efficiently perform similarity searches in a high-dimensional space, offering relevant matches based on the semantic content of the job descriptions. This approach works well when the categories are not constant and the dataset is dynamic.

####**Assumptions:** 
#####The dataset is dynamic, as the job postings during various parts of the year may vary.
#####There are no fixed set of categories in which all the job postings may fall into.
#####There are currently no ways to estimate the relevant dcouments out of the retrieved ones.

####**Findings:** There are close to 700 categories in train and there are 709 records in test and there are only 602 categories that overlap. Assuming a fixed set of categories might be counterintuitive in our case. Using a fine tuned model may have better understanding of the context and might provide better results but when it comes to handling out of distribution categories in the test data, it is going to overfit to categories in the train. 

####Another issue is that all categories are not equally represented and fine tuning it to a imbalanced dataset will possibly lead to under representation of some classes, therefore skewing the results.

####Retrieval system might be a better approach as we have to handle dynamic data and it is very difficult to fine tune every time we have new data. Retrieval system is a cost efficient approach and easy to implement that fine tuning. Specific to the use case given, it is more reasonable to use a retrieval system to handle dynamic data and the out of distribution test set.

####**Implementation:** The input is the title of the job description appended to the body. The output is the ONET_NAME. All the input from the training data is encoded using sentence transformer embeddings and stored in a FAISS index. At inference time, use the test title and job description, retrieve top N documents that are semantically similar to the one provided and return their labels as the predictions.

####**Evalaution:** 
#####**Top K accuracy:**Accuracy is calculated if the ground truth category is found within the top K retrieved categories. K ranges from 1-5. However this metric does not penalize based on rank. 

#####**Mean Reciprocal Rank:** This metric ranks the prediction based on which position among the retrieved records is the ground truth present. 

#####**Custom Evaluation Metric:** All the above metrics works well, given the assumption that test is not out of distribution. In our case, test being out of distribtuion penalizing the scores giving these data points zero might skew the final results. To handle this case, I propose a custom metric that will penalize the retrieved records only if they are not from the super category(the first two numbers of the ONET).







