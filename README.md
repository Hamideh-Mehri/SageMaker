Process of training a Scikit-learn model by using the SageMaker Python SDK:

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
3-1 **Creating an Estimator**: We run Scikit-learn training scripts(scripts.py) on SageMaker by creating `SKLearn Estimators`(`SKLearn` imported from `sagemaker.sklearn.estimator`). We specified these arguments: the path of the training script(`entry_point`), the compute resources needed (`instance_type` and `instance_count`), and the passed hyperparameters(n_estimators and random_state). 

3-2 **Calling the fit method**: We start our training script by calling `fit` on the `SKLearn Estimator`. The argument for the fit method is a dict from string channel names to s3 URIs.

4- **Deploy an Endpoint from Model Data**: We deployed the model directly from model data(model artifact) in S3. Training Job model data is saved to .tar.gz files in S3, however if you have local data you want to deploy, you can prepare the data yourself.

sklearn_model = SKLearnModel(name = model_name,
                             model_data= the s3 path that model artifact is saved,
                             role= ,
                             entry_point=path to "script.py" file,
                             framework_version="1.0-1")

predictor = sklearn_model.deploy(instance_type="ml.c4.xlarge", initial_instance_count=1, 
                                endpoint_name = endpoint_name)




