# Operationalizing Machine Learning

In this project, we are working with the Bank Marketing dataset. We use Azure to configure a cloud-based machine learning production model using AutoML, deploy it, and consume it. We are also creating, publishing, and consuming a pipeline. 

This dataset is about a phone call marketing campaign. The original data can be found [@UC Irvine Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/bank+marketing). The dataset can be used (as we are using) to predict if the client will subscribe to a term deposit or not. The target variable is y. 

The lab environment provided by Udacity will not be used for this project. Instead, a local development environment along with a Microsoft Azure account will be used. 

## Architectural Diagram

In this project, We are following the below steps:

1. Authentication
2. Automated ML Experiment
3. Deploy the Best Model
4. Enable Logging
5. Swagger Documentation
6. Consume Model Endpoints
7. Create and Publish a Pipeline
8. Documentation

![Main Steps](images/steps/steps.png)

Image by Udacity

## Key Steps
1. Authentication

   Authentication is crucial for the continuous flow of operations. Continuous Integration and Delivery system (CI/CD) rely on uninterrupted flows. When authentication is not set properly, it requires human interaction and thus, the flow is interrupted. An ideal scenario is that the system doesn't stop waiting for a user to input a password. So whenever possible, it's good to use authentication with automation.

   A “Service Principal” is a user role with controlled permissions to access specific resources. Using a service principal is a great way to allow authentication while reducing the scope of permissions, which enhances security.

   We will use local environment for Authentication.
   
   Main operations in the Authentication step are as follows:
   
      - Use Git Bash to sign in Microsoft account using the `az login` command.
      
      <p style="color:blue;font-size:10px;">Git Bash screen showing the result 'az login' command</p>
      
      ![Authentication_rm_2.png](images/authentication/authentication_2.png)
      
      - Ensure the az command-line tool is installed along with the ml using the `az extension add -n azure-cli-ml` command.

      <p style="color:blue;font-size:10px;">Git Bash screen showing the result 'az extension add -n azure-cli-ml' command</p>
      
      ![Authentication_rm_3.png](images/authentication/authentication_3.png)
      
      - Create the Service Principal with az after login in using the `az ad sp create-for-rbac --sdk-auth --name ml-auth` command.

      <p style="color:blue;font-size:10px;">Git Bash screen showing the result 'az ad sp create-for-rbac --sdk-auth --name ml-auth' command</p>
      
      ![Authentication_rm_4.png](images/authentication/authentication_4.png)
      
      - Capture the "objectId" using the `clientID`. Use the following command:
      
      ```az ad sp show --id xxxxxxxx-3af0-4065-8e14-xxxxxxxxxxxx```

      <p style="color:blue;font-size:10px;">Git Bash screen showing the result 'az ad sp show --id xxxxxxxx-3af0-4065-8e14-xxxxxxxxxxxx' command</p>
      
      ![Authentication_rm_5.png](images/authentication/authentication_5.png)
      
      - Assign the role to the new Service Principal for the given Workspace, Resource Group, and User `objectId`. You will need to match your workspace, subscription, and ID. There should be no error in the output. Use the following command:
      
      ```az ml workspace share -w xxx -g xxx --user xxxxxxxx-cbdb-4cfd-089f-xxxxxxxxxxxx --role owner```

      <p style="color:blue;font-size:10px;">Git Bash screen showing the result 'az ml workspace share -w xxx -g xxx --user xxxxxxxx-cbdb-4cfd-089f-xxxxxxxxxxxx --role owner' command</p>
      
      ![Authentication_rm_6.png](images/authentication/authentication_6.png)
      
2. Automated ML Experiment

   In this step, we will create an experiment using Automated ML, configure a compute cluster, and use that cluster to run the experiment. We will use Azure portal for this purpose.

   We will use the Bank Marketing dataset described above. 

   In the following section we will deploy the best model of this AutoML experiment.

   We need to configure a compute cluster for this AutoML experiment. To do that we can either use an existing cluster or create a new one. We will create a new cluster with the following configuration:

      - Region: eastus2
      - Virtual machine priority: Low priority
      - Virtual machine type: CPU
      - Virtual machine size: Standart_DS12_v2
      - Compute name: auto-ml
      - Minimum number of nodes: 1
      - Maximum number of nodes: 2

   Run configuration for the autoML experiment is as follows:

      - Experiment name (Create new): ml-experiment-1
      - Target column: y
      - Compute cluster name: auto-ml
      - Primary metric: Accuracy
      - Explain best model: Selected
      - Exit criterion: 
         - Training job time (hours): 1
      - Concurrency:
         - Max concurrent iterations: 2
   
   Main operations in the Automated ML Experiment step are as follows:
   
   - Upload the bankmarketing_train.csv to Azure Machine Learning Studio so that it can be used when training the model.

   <p style="color:blue;font-size:10px;">Bank Marketing dataset</p>
   
   ![automl_rm_dataset_1.png](images/automl/automl_dataset_1.png)
   
   - Create a new AutoML run and select Bankmarketting Dataset

   <p style="color:blue;font-size:10px;">New AutoML Run</p>
   
   ![automl_1.png](images/automl/automl_1.png)
   
   - Create a new AutoML experiment

   <p style="color:blue;font-size:10px;">A New Experiment</p>
   
   ![AutoML_4.png](images/automl/AutoML_4.png)
   
   - Configure a new compute cluster

   <p style="color:blue;font-size:10px;">Create Compute Cluster</p>
   
   ![AutoML_3.png](images/automl/AutoML_3.png)
   
   - Run the experiment using classification, ensure 'Explain best model' is checked

   <p style="color:blue;font-size:10px;">Additional Configurations</p>
   
   ![AutoML_5.png](images/automl/AutoML_5.png)

   <p style="color:blue;font-size:10px;">AutoML Run with status 'Running'</p>
   
   ![AutoML_8.png](images/automl/AutoML_8.png)
   
   - Wait for the experiment to finish and explore the best model

   <p style="color:blue;font-size:10px;">AutoML Run with Status 'Completed'</p>
   
   ![AutoML_9.png](images/automl/AutoML_9.png)

   <p style="color:blue;font-size:10px;">Automated ML Tab Showing the Completed Experiment</p>
   
   ![AutoML_10.png](images/automl/AutoML_10.png)

   <p style="color:blue;font-size:10px;">Trained Models for Each Run</p>
        
   ![AutoML_11.png](images/automl/AutoML_11.png)

   <p style="color:blue;font-size:10px;">Best Model: VotingEnsemble</p>
   
   ![AutoML_12.png](images/automl/AutoML_12.png)
   
3. Deploy the Best Model

   The best model in the previous step was a VotingEnsemble. Deploying the Best Model will allow to interact with the HTTP API service and interact with the model by sending data over POST requests.

   We will use Azure portal for deployment.

   Main operations in Deploy the best model step are as follows:
   
   - Select the best model for deployment

   <p style="color:blue;font-size:10px;">Best Model Selected (Deploy status: No deployment yet)</p>
   
   ![Deploy_1.png](images/deploy/Deploy_1.png)
   
   - Deploy the model and enable Authentication

   <p style="color:blue;font-size:10px;">Best Model Selected (Deploy status: Running)</p>
   
   ![Deploy_2.png](images/deploy/Deploy_2.png)
   
   - Deploy the model using Azure Container Instance

   <p style="color:blue;font-size:10px;">Deployed Model (Endpoint) With a Healthy Deployment State</p>
   
   ![Deploy_5.png](images/deploy/Deploy_5.png)
   
4. Enable Logging

   We can now enable Application Insights and retrieve logs. Application Insights is a special Azure service that provides key facts about an application. It is a very useful tool to detect anomalies and visualize performance.

   We will work on local environment in this step. But first, we need to download `config.json` from Azure portal.
   
   Main operations in the Enable Logging step are as follows:
   
   - Download `config.json` from ML Studio

   <p style="color:blue;font-size:10px;">How to download 'config.json'</p>
   
   ![App-In_8.png](images/application_insights/App-In_8.png)
   
   - Write and run the code (logs.py) to enable Application insights

   <p style="color:blue;font-size:10px;">logs.py output in Git Bash</p>

   ![App-In_5.png](images/application_insights/App-In_5.png)

   <p style="color:blue;font-size:10px;">'Application Insights url' in Model Endpoint</p>
   
   ![App-In_6.png](images/application_insights/App-In_6.png)
   
   - Explore Application insights

   <p style="color:blue;font-size:10px;">Application Insights</p>
   
   ![App-In_7.png](images/application_insights/App-In_7.png)
   
5. Swagger Documentation

   In this step, we will consume the deployed model using Swagger. Swagger is a tool that helps build, document, and consume RESTful web services like the ones we are deploying in Azure ML Studio.

   We will work on local environment in this step. But first, we need to download `swagger.json` from Azure portal. We can download `swagger.json` using endpoints tab in Azure ML Studio.
   
   Main operations in the Swagger Documentation step are as follows:
    
    - Download the swagger.json file.
    
     Azure provides a swagger.json that is used to create a web site that documents the HTTP endpoint for a deployed model. We can find swagger.json URI in the Endpoints section.

     <p style="color:blue;font-size:10px;">Swagger URI</p>
     
     ![Swagger_8.png](images/swagger/Swagger_8.png)
     
     We need to download the swagger.json to the swagger folder. There should be 3 files in the swagger folder.

     <p style="color:blue;font-size:10px;">Local Directory</p>
     
     ![Swagger_2.png](images/swagger/Swagger_2.png)
     
     - Run the `swagger.sh` in git bash.

     <p style="color:blue;font-size:10px;">'swagger.sh' Output</p>
     
     ![Swagger_3.png](images/swagger/Swagger_3.png)
     
     - Run the `serve.py` in another git bash.

     <p style="color:blue;font-size:10px;">'serve.py' Output</p>
     
     ![Swagger_4.png](images/swagger/Swagger_4.png)
     
     - Interact with the swagger instance running with the documentation for the HTTP API for the model.
     
     To do that we will first use the `http://localhost/` in our browser to interact with the swagger instance running with the documentation for the HTTP API of the model then we will use `http://localhost:8000/swagger.json` to display the contents of the API for the model.

     <p style="color:blue;font-size:10px;">Swagger Page</p>
     
     ![Swagger_5.png](images/swagger/Swagger_5.png)

     <p style="color:blue;font-size:10px;">More on Swagger Page</p>
     
     ![Swagger_6.png](images/swagger/Swagger_6.png)

     <p style="color:blue;font-size:10px;">More on Swagger Page</p>
     
     ![Swagger_7.png](images/swagger/Swagger_7.png)
     
6. Consume Model Endpoints
   
   We can consume a deployed service via an HTTP API. An HTTP API is an URL that is exposed over the network so that interaction with a trained model can happen via HTTP requests.

   Users can initiate an input request, usually via an HTTP POST request. HTTP POST is a request method that is used to submit data. The HTTP GET is another commonly used request method. HTTP GET is used to retrieve information from a URL. The allowed request methods and the different URLs exposed by Azure create a bi-directional flow of information.

   The APIs exposed by Azure ML will use JSON (JavaScript Object Notation) to accept data and submit responses. It served as a bridge language among different environments.

   We will work on local environment in this step. But first, we need to get `scoring_uri` and the `key`from Azure portal.
   
   Main operations in the Consume Model Endpoints step are as follows:
   
   - Modify both the `scoring_uri` and the `key` (in `endpoint.py`) to match the key for our service and the URI that was generated after deployment.
   
   `scoring_uri` and the `key` can be found on the 'consume' tab of the model endpoint.

   <p style="color:blue;font-size:10px;">Model Endpoint (Consume Tab)</p>
   
   ![CME_2.png](images/consume_model_endpoints/CME_2.png)
   
   - Run `endpoint.py`

   <p style="color:blue;font-size:10px;">'endpoint.py' Output</p>
   
   ![CME_3.png](images/consume_model_endpoints/CME_3.png)
 
   **Benchmarking**

   A benchmark is used to create a baseline or acceptable performance measure. Benchmarking HTTP APIs is used to find the average response time for a deployed model.

   One of the most significant metrics is the response time since Azure will timeout if the response times are longer than sixty seconds.

   Apache Benchmark is an easy and popular tool for benchmarking HTTP services. 

   We will work on local environment in this step.
   
   Main operations in the Benchmarking step are as follows:
   
   - Make sure the Apache Benchmark command-line tool is [installed](https://www.cedric-dumont.com/2017/02/01/install-apache-benchmarking-tool-ab-on-windows/) and available in your path.
   
   - In the `endpoint.py`, replace the key and URI again
   
   - Run `endpoint.py`. A `data.json` file should appear
   
   - Run the `benchmark.sh` file.

   <p style="color:blue;font-size:10px;">'benchmark.sh' Output</p>
   
    ![bm_0.png](images/benchmark/bm_0.png)

    <p style="color:blue;font-size:10px;">More on 'benchmark.sh' Output</p>
    
    ![bm_1.png](images/benchmark/bm_1.png)
    
7. Create and Publish a Pipeline

   For this part of the project, we will use the Jupyter Notebook provided in the starter files folder and the Azure portal.
   
   Main operations in the Create and Publish a Pipeline step are as follows:
   
   - Upload the Jupyter Notebook

   <p style="color:blue;font-size:10px;">Jupyter Notebook on Azure Machine Learning Studio</p>
   
   ![pipeline_1.png](images/pipeline/pipeline_1.png)
   
   - Update all the variables that are noted to match the environment and make sure that a `config.json` has been downloaded and is available in the current working directory.

   <p style="color:blue;font-size:10px;">Directory Structure</p>
   
   ![pipeline_2.png](images/pipeline/pipeline_2.png)
   
   - Run through the cells

   <p style="color:blue;font-size:10px;">Jupyter Notebook Output - Dataset</p>
   
   ![pipeline_3.png](images/pipeline/pipeline_3.png)
   
   Below you can find some important screenshots for this stage:

   <p style="color:blue;font-size:10px;">Running Pipeline on Azure Machine Learning Studio</p>
   
   ![pipeline_6.png](images/pipeline/pipeline_6.png)

   <p style="color:blue;font-size:10px;">Running and Completed Pipelines on Azure Machine Learning Studio</p>
   
   ![pipeline_s_1.png](images/pipeline/pipeline_s_1.png)

   <p style="color:blue;font-size:10px;">Pipeline Endpoints on Azure Machine Learning Studio</p>
   
   ![pipeline_s_2.png](images/pipeline/pipeline_s_2.png)

   <p style="color:blue;font-size:10px;">'Pipeline run overview' for Run 61 on Azure Machine Learning Studio</p>

   ![pipeline_14.png](images/pipeline/pipeline_14.png)

   <p style="color:blue;font-size:10px;">'Published pipeline overview' for Bankmarketting Dataset on Azure Machine Learning Studio</p>
   
   ![pipeline_s_3.png](images/pipeline/pipeline_s_3.png)

   <p style="color:blue;font-size:10px;">Run Details Output on Jupyter Notebook</p>

   ![pipeline_8.png](images/pipeline/pipeline_8.png)

   <p style="color:blue;font-size:10px;">More Run Details Output on Jupyter Notebook</p>
   
   ![pipeline_9.png](images/pipeline/pipeline_9.png)

   <p style="color:blue;font-size:10px;">More Run Details Output on Jupyter Notebook</p>
   
   ![pipeline_12.png](images/pipeline/pipeline_12.png)

   <p style="color:blue;font-size:10px;">More Run Details Output on Jupyter Notebook</p>
   
   ![pipeline_13.png](images/pipeline/pipeline_13.png)

   <p style="color:blue;font-size:10px;">Published Pipeline on Jupyter Notebook</p>
   
   ![pipeline_19.png](images/pipeline/pipeline_19.png)

   <p style="color:blue;font-size:10px;">More Run Details Output on Jupyter Notebook</p>
   
   ![pipeline_s_5.png](images/pipeline/pipeline_s_5.png)

   <p style="color:blue;font-size:10px;">'Experiment Run status' on Azure Machine Learning Studio</p>
   
   ![pipeline_10.png](images/pipeline/pipeline_10.png)

   <p style="color:blue;font-size:10px;">'Azure Machine Learning Studio Home Page with Runs and Computes</p>
   
   ![pipeline_22.png](images/pipeline/pipeline_22.png)

   <p style="color:blue;font-size:10px;">'Experiment Run status' on Azure Machine Learning Studio</p>
   
   ![pipeline_s_9.png](images/pipeline/pipeline_s_9.png)   
   
## Screen Recording
[Project 2 Operationalizing Machine Learning Screencast](https://youtu.be/6nQsHSIBeCA)

Note: You can refer to `Screencast_text.txt` if you have difficulty understanding screencast audio.

## Future Work

### Improvement for future experiments

The dataset is imbalanced. Although AutoML seems to handle imbalanced data we can try to handle it manually.

TensorFlow LinearClassifier and TensorFlow DNN seem to be blocked since they are not supported by AutML. We can try some HyperDrive runs with these estimators and see their performance.

Deep learning is disabled. We can include Deep learning and run a new AutoML pipeline.

We can select a smaller size of predictors using the feature importance visualizations generated by the model explainability feature of Azure AutoML and run a new AutoML pipeline. By doing this we may get better performance for our model.

We can try new HyperDrive runs with the best performing estimator (VotingEnsemble) to get a better score.

We can monitor the endpoint using Application Insights and detect performance anomalies and visualize performance. We can easily evaluate performance and identify pitfalls since we have already started benchmarking.

## References
- [Machine Learning Engineer for Microsoft Azure](https://www.udacity.com/course/machine-learning-engineer-for-microsoft-azure-nanodegree--nd00333)
