## boto3 library
* It is the `AWS SDK(Software Development Kit)` for Python.
* It provides python developers with the ability to interact with nearly all aspects of AWS.
* It abstracts the complexity of dealing with direct API calls and offers a more intuitive way to manage AWS resources programmatically.
* It allows you to write python scripts to automate tasks across various AWS services. 

  ### Client vs Resource
  `boto3` offers two distinct ways to interact with AWS services:
    
  **Client**: provides low-level AWS service access that is closely aligned with the underlying AWS API.
    
  **Resource**: provides a higher-level, object oriented interface to AWS services, making it easier to use.
   
      S3 = boto3.client('s3')
      
      ddb = boto3.resource('dynamodb')  

### Sessions
* A `session` in `boto3` is a configuration container that stores settings like region, credentials, and other options needed to communicate with AWS services.
* Why use sessions? sessions manage your authentication(credentials) and configuration(like region settings) when talking to AWS services. They are particularly useful when you need different configurations in a single application, such as using multiple regions or credential sets.
* you can create a session if you need specific configurations:

    import boto3
  
    session = boto3.session(region_name='...', profile_name='...')  #this session uses a specific AWS profile and region

    s3 = session.client('s3')   #we use this session to create a client for s3

    ddb = session.resource('dynamodb')




## Process of training a Scikit-learn model by using the SageMaker Python SDK:

1- **Prepare dataset and upload it to Amazon S3**.

2- **Write Scikit-learn training script**(script.py in this example):
training script loads data from the input channels(there are two input channels in this example: `train` and `test`), configures training with hyperparameters, trains a model, and saves a model to model_dir so that it can be hosted later. Hyperparameters are passed to the script as arguments and can be retrieved with an `argparse.ArgumentParser instance`. 

we can access useful properties about the training environment through various **environment variables**. For example:

`SM_MODEL_DIR`: A string representing the path to the directory to write model artifacts to. These artifacts are uploaded to S3 for model hosting.

`SM_OUTPUT_DATA_DIR`: A string representing the filesystem path to write output artifacts to. Output artifacts may include checkpoints, graphs, and other files to save, not including model artifacts. These artifacts are compressed and uploaded to S3 to the same S3 prefix as the model artifacts.

`SM_CHANNEL_TRAIN`: A string representing the path to the directory containing data in the ‘train’ channel

`SM_CHANNEL_TEST`: Same as above, but for the ‘test’ channel.

In the training script we **saved the trained model for deployment on SageMaker**. We saved our model to a certain filesystem path called `model_dir`. This value is accessible through the environment variable `SM_MODEL_DIR`. we saved it as .joblib file at the end of training.  

3- **Configure the training job**:

3-1 **Initiating a training job by Creating an Estimator**: We run Scikit-learn training scripts(scripts.py) on SageMaker by creating `SKLearn Estimators`(`SKLearn` imported from `sagemaker.sklearn.estimator`). We specified these arguments: the path of the training script(`entry_point`), the compute resources needed (`instance_type` and `instance_count`), and the passed hyperparameters(n_estimators and random_state). sagemaker passes these arguments to script.py as command-line arguments. 

3-2 **launching the training job by Calling the fit method**: We start our training script by calling `fit` on the `SKLearn Estimator`. The argument for the fit method is a dict from string channel names to s3 URIs. SageMaker sets `SM_CHANNEL_TRAIN` and `SM_CHANNEL_TEST` to the trainpath and testpath specified in the fit method. 

It seems that SageMaker handle cases where users specify the full path, including the filename. It could automatically extract the directory part from the provided path and set the environment variables `SM_CHANNEL_TRAIN` and `SM_CHANNEL_TEST` accordingly. This means it would ignore the filename in the path and just use the directory for setting up the environment variables. When I sent the path without the filename it still works. If we want to specify the filenames something other than default values specified in `train-file` and `test-file` arguments , we can specify them in the hyperparameter in SKLearn: 

 hyperparameters={
                          "n_estimators": 100,
                          "random_state": 0,
                          "train-file": "train-V-1.csv", 
                          "test-file": "test-V-1.csv"  
                      },

4- **Deploy an Endpoint from Model Data**: We deployed the model directly from model data(model artifact) in S3. Training Job model data is saved to .tar.gz files in S3, however if you have local data you want to deploy, you can prepare the data yourself.

sklearn_model = SKLearnModel(name = model_name,
                             model_data= the s3 path that model artifact is saved,
                             role= ,
                             entry_point=path to "script.py" file,
                             framework_version="1.0-1")

predictor = sklearn_model.deploy(instance_type="ml.c4.xlarge", initial_instance_count=1, 
                                endpoint_name = endpoint_name)


The Scikit-learn Endpoint you create with `deploy` runs a SageMaker Scikit-learn model server. The SageMaker Scikit-learn model server loads your model by invoking a `model_fn function` that you must provide in your script. 

more information: https://sagemaker.readthedocs.io/en/stable/frameworks/sklearn/using_sklearn.html
