### **Introduction**

The dataset contains records describing various companies along with multiple business-related attributes. Although most records appear distinct, several of them may refer to the same **real-world company**. The objective of this project is to identify and group together records that belong to the same entity.

The **entity resolution pipeline** is structured into the following stages:

1. **Spark Setup** – configuration of a **PySpark environment** to ensure scalability.  
2. **Dataset Inspection and Column Coverage** – exploration of dataset **schema** and evaluation of **data completeness**.  
3. **Selecting Relevant Columns** – retention of only the **relevant columns** for blocking, matching, and validation.  
4. **Data Preprocessing and Normalization** – cleaning and normalization of attributes to enable **consistent comparisons**.  
5. **Candidate Pair Generation** – grouping records into **candidate pairs** based on a common **blocking key** to reduce the number of comparisons.  
6. **Pairwise Matching Score Computation** – calculation of a **similarity score** for each candidate pair.  
7. **Graph-Based Clustering** – filtering high-scoring pairs and **grouping connected records** using a graph-based approach.  
8. **Cluster Analysis and Visual Validation** – inspection of **resulting clusters** and presentation of **statistical distributions**.

For a complete view of the code and results obtained, please access [entity_resolution.ipynb](entity_resolution.ipynb).