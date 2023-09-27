# WAFER FAULT DETECTION


## PROBLEM STATEMENT
The inputs of various sensors for different wafers have been provided. In electronics, a wafer (also called a slice or substrate) is a thin slice of semiconductor used for the fabrication of integrated circuits. The goal is to build a machine learning model that predicts whether a wafer needs to be replaced or not(i.e., whether it is working or not) based on the inputs from various sensors. There are two classes: +1 and -1. 
- +1 means that the wafer is in working condition and it doesn’t need to be replaced.
- -1 means that the wafer is faulty and it needs to be replaced.


## FUNCTIONAL ARCHITECTURE
![image](https://github.com/Ria2810/wafer-fault-detection/assets/67699993/681ab298-2942-4138-88c7-e648c7195b0c)


## MODEL ARCHITECTURE
![image](https://github.com/Ria2810/wafer-fault-detection/assets/67699993/d8b9e982-8de7-4452-975e-0ba389b764d0)


### DATA DESCRIPTION
The client will send data in multiple sets of files in batches at a given location. Data will contain Wafer names and 590 columns of different sensor values for each wafer. The last column will have the "Good/Bad" value for each wafer.
"Good/Bad" column will have two unique values +1 and -1.  
- "+1" represents Bad wafer.
- "-1" represents Good Wafer.

Apart from training files, we also require a "schema" file from the client, which contains all the relevant information about the training files such as:
Name of the files, Length of Date value in FileName, Length of Time value in FileName, Number of Columns, Name of the Columns, and their datatype.


### DATA VALIDATION
In this step, we perform different sets of validation on the given set of training files.  
1.	 <ins>Name Validation</ins>- We validate the name of the files based on the given name in the schema file. We have created a regex pattern as per the name given in the schema file to use for validation. After validating the pattern in the name, we check for the length of date in the file name as well as the length of time in the file name. If all the values are as per requirement, we move such files to "Good_Data_Folder" else we move such files to "Bad_Data_Folder."

2.	 <ins>Number of Columns</ins> - We validate the number of columns present in the files, and if it doesn't match with the value given in the schema file, then the file is moved to "Bad_Data_Folder."


3.	 <ins>Name of Columns</ins> - The name of the columns is validated and should be the same as given in the schema file. If not, then the file is moved to "Bad_Data_Folder".

4.	 <ins>The datatype of columns</ins> - The datatype of columns is given in the schema file. This is validated when we insert the files into Database. If the datatype is wrong, then the file is moved to "Bad_Data_Folder".


5.	<ins>Null values in columns</ins> - If any of the columns in a file have all the values as NULL or missing, we discard such a file and move it to "Bad_Data_Folder".


### DATA INSERTION IN DATABASE
1. <ins>Database Creation and connection</ins> - Create a database with the given name passed. If the database is already created, open the connection to the database.
2. <ins>Table creation in the database</ins> - Table with name - "Good_Data", is created in the database for inserting the files in the "Good_Data_Folder" based on given column names and datatype in the schema file. If the table is already present, then the new table is not created and new files are inserted in the already present table as we want training to be done on new as well as old training files.
3. <ins>Insertion of files in the table</ins> - All the files in the "Good_Data_Folder" are inserted in the above-created table. If any file has invalid data type in any of the columns, the file is not loaded in the table and is moved to "Bad_Data_Folder".


### MODEL TRAINING
1. <ins>Data Export from Db</ins> - The data in a stored database is exported as a CSV file to be used for model training.
2. <ins>Data Preprocessing</ins>  
   a. Check for null values in the columns. If present, impute the null values using the KNN imputer.
   
   b. Check if any column has zero standard deviation, remove such columns as they don't give any information during model training.
4. <ins>Clustering</ins> - KMeans algorithm is used to create clusters in the preprocessed data. The optimum number of clusters is selected by plotting the elbow plot, and for the dynamic selection of the number of clusters, we are using "KneeLocator" function. The idea behind clustering is to implement different algorithms to train data in different clusters. The Kmeans model is trained over preprocessed data and the model is saved for further use in prediction.
5. <ins>Model Selection</ins> - After clusters are created, we find the best model for each cluster. We are using two algorithms, "Random Forest" and "XGBoost". For each cluster, both the algorithms are passed with the best parameters derived from GridSearch. We calculate the AUC scores for both models and select the model with the best score. Similarly, the model is selected for each cluster. All the models for every cluster are saved for use in prediction.


### PREDICTION DATA DESCRIPTION
Client will send the data in multiple set of files in batches at a given location. Data will contain Wafer names and 590 columns of different sensor values for each wafer. 

Apart from prediction files, we also require a "schema" file from client which contains all the relevant information about the training files such as:

Name of the files, Length of Date value in FileName, Length of Time value in FileName, Number of Columns, Name of the Columns and their datatype.


### DATA VALIDATION
In this step, we perform different sets of validation on the given set of prediction files.  
1.	 <ins>Name Validation</ins>- We validate the name of the files based on the given name in the schema file. We have created a regex pattern as per the name given in the schema file to use for validation. After validating the pattern in the name, we check for the length of date in the file name as well as the length of time in the file name. If all the values are as per requirement, we move such files to "Good_Data_Folder" else we move such files to "Bad_Data_Folder."

2.	 <ins>Number of Columns</ins> - We validate the number of columns present in the files, and if it doesn't match with the value given in the schema file, then the file is moved to "Bad_Data_Folder."


3.	 <ins>Name of Columns</ins> - The name of the columns is validated and should be the same as given in the schema file. If not, then the file is moved to "Bad_Data_Folder".

4.	 <ins>The datatype of columns</ins> - The datatype of columns is given in the schema file. This is validated when we insert the files into Database. If the datatype is wrong, then the file is moved to "Bad_Data_Folder".


5.	<ins>Null values in columns</ins> - If any of the columns in a file have all the values as NULL or missing, we discard such a file and move it to "Bad_Data_Folder".


### DATA INSERTION IN DATABASE
1. <ins>Database Creation and connection</ins> - Create a database with the given name passed. If the database is already created, open the connection to the database.
2. <ins>Table creation in the database</ins> - Table with name - "Good_Data", is created in the database for inserting the files in the "Good_Data_Folder" based on given column names and datatype in the schema file. If the table is already present, then the new table is not created and new files are inserted in the already present table as we want training to be done on new as well as old training files.
3. <ins>Insertion of files in the table</ins> - All the files in the "Good_Data_Folder" are inserted in the above-created table. If any file has invalid data type in any of the columns, the file is not loaded in the table and is moved to "Bad_Data_Folder".


### PREDICTION
1. <ins>Data Export from Db</ins> - The data in the stored database is exported as a CSV file to be used for prediction.
2. <ins>Data Preprocessing</ins>    
   a) Check for null values in the columns. If present, impute the null values using the KNN imputer.
   
   b) Check if any column has zero standard deviation, remove such columns as we did in training.
4. <ins>Clustering</ins> - KMeans model created during training is loaded, and clusters for the preprocessed prediction data is predicted.
5. <ins>Prediction</ins> - Based on the cluster number, the respective model is loaded and is used to predict the data for that cluster.
6. Once the prediction is made for all the clusters, the predictions along with the Wafer names are saved in a CSV file at a given location and the location is returned to the client.


### DEPLOYMENT
We have deployed our model as a web application using flask.
