Process of training a Scikit-learn model by using the SageMaker Python SDK:

1- Prepare dataset and upload it to Amazon S3.

2- Write Scikit-learn training script(script.py):
training script loads data from the input channels(there are two input channels in this example: `train` and `test`), configures training with hyperparameters, trains a model, and saves a model to model_dir so that it can be hosted later. Hyperparameters are passed to the script as arguments and can be retrieved with an `argparse.ArgumentParser instance`.

3- Configure the training job by specifying the path of the training script(entry_point), S3 location of the data, 
the compute resources needed (instance_type and instance_count), 
and the output path for the model artifacts(args.model_dir in script.py, the default value is specified by the 
environment variable `SM_MODEL_DIR` which is  A string representing the path to the directory to write 
model artifacts to. These artifacts are uploaded to S3 for model hosting.)
