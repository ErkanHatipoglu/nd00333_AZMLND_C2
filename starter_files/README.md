# Operationalizing Machine Learning

In this project, we are working with the Bank Marketing dataset. We use Azure to configure a cloud-based machine learning production model, deploy it, and consume it. We are also creating, publishing, and consuming a pipeline. 

This dataset is about a phone call marketing campaign. The original data can be found [@UC Irvine Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/bank+marketing). The dataset can be used (as we are using) to predict if the client will subscribe to a term deposit or not. The target variable is y. 

The lab environment provided by Udacity will not be used for this project. Instead a local development environment along with Microsoft Azure account will be used. 

## Architectural Diagram

In this project, We are following the below steps:

1. Authentication
2. Automated ML Experiment
3. Deploy the best model
4. Enable logging
5. Swagger Documentation
6. Consume model endpoints
7. Create and publish a pipeline
8. Documentation

![Main Steps](images/steps/steps.png)

Image by Udacity

## Key Steps
1. Authentication

   Authentication is crucial for the continuous flow of operations. Continuous Integration and Delivery system (CI/CD) rely on uninterrupted flows. When authentication is not set properly, it requires human interaction and thus, the flow is interrupted. An ideal scenario is that the system doesn't stop waiting for a user to input a password. So whenever possible, it's good to use authentication with automation.

   A “Service Principal” is a user role with controlled permissions to access specific resources. Using a service principal is a great way to allow authentication while reducing the scope of permissions, which enhances security.
   
   Main operations in Authentication step are as follows:
   
      - Use Git Bash to sign in Microsoft account using `az login` command.
      
      ![Authentication_rm_2.png](images/authentication/authentication_2.png)
      
      - Ensure the az command-line tool is installed along with the ml using `az extension add -n azure-cli-ml` command.
      
      ![Authentication_rm_3.png](images/authentication/authentication_3.png)
      
      - Create the Service Principal with az after login in using `az ad sp create-for-rbac --sdk-auth --name ml-auth` command.
      
      ![Authentication_rm_4.png](images/authentication/authentication_4.png)
      
      - Capture the "objectId" using the `clientID`. Use the following command:
      
      ```az ad sp show --id xxxxxxxx-3af0-4065-8e14-xxxxxxxxxxxx```
      
      ![Authentication_rm_5.png](images/authentication/authentication_5.png)
      
      - Assign the role to the new Service Principal for the given Workspace, Resource Group and User `objectId`. You will need to match your workspace, subscription, and ID. There should be no error at the output. Use the following command:
      
      ```az ml workspace share -w xxx -g xxx --user xxxxxxxx-cbdb-4cfd-089f-xxxxxxxxxxxx --role owner```
      
      ![Authentication_rm_6.png](images/authentication/authentication_6.png)
      
2. Automated ML Experiment

   In this step, we will create an experiment using Automated ML, configure a compute cluster, and use that cluster to run the experiment. We will use the Bank Marketing dataset described above.
   
   Main operations in Automated ML Experiment step are as follows:
   
   - Upload the bankmarketing_train.csv to Azure Machine Learning Studio so that it can be used when training the model.
   
   ![automl_rm_dataset_1.png](images/automl/automl_dataset_1.png)
   
   - Create a new AutoML run and select BAnkmarketting dataset
   
   ![automl_1.png](images/automl/automl_1.png)
   
   - Create a new AutoML experiment
   
   ![AutoML_4.png](images/automl/AutoML_4.png)
   
   - Configure a new compute cluster
   
   ![AutoML_3.png](images/automl/AutoML_3.png)
   
   - Run the experiment using classification, ensure 'Explain best model' is checked
   
   ![AutoML_5.png](images/automl/AutoML_5.png)
   
   ![AutoML_8.png](images/automl/AutoML_8.png)
   
   - Wait for the experiment finishes and explore the best model
   
   !AutoML_9.png](images/automl/AutoML_9.png)
   
   ![AutoML_10.png](images/automl/AutoML_10.png)
        
   ![AutoML_11.png](images/automl/AutoML_11.png)
   
   ![AutoML_12.png](images/automl/AutoML_12.png)
   
3. Deploy the best model

   Deploying the Best Model will allow to interact with the HTTP API service and interact with the model by sending data over POST requests.
   
   Main operations in Automated ML Experiment step are as follows:

## Screen Recording
*TODO* Provide a link to a screen recording of the project in action. Remember that the screencast should demonstrate:

## Standout Suggestions
*TODO (Optional):* This is where you can provide information about any standout suggestions that you have attempted.

## References
- [Machine Learning Engineer for Microsoft Azure](https://www.udacity.com/course/machine-learning-engineer-for-microsoft-azure-nanodegree--nd00333)

