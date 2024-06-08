Process of training a Scikit-learn model by using the SageMaker Python SDK:

1- Prepare dataset and upload it to Amazon S3.

2- Write Scikit-learn training script(script.py):
training script loads data from the input channels(there are two input channels in this example: `train` and `test`), configures training with hyperparameters, trains a model, and saves a model to model_dir so that it can be hosted later. Hyperparameters are passed to the script as arguments and can be retrieved with an `argparse.ArgumentParser instance`.

we can access useful properties about the training environment through various environment variables. For example:

`SM_MODEL_DIR`: A string representing the path to the directory to write model artifacts to. These artifacts are uploaded to S3 for model hosting.

`SM_OUTPUT_DATA_DIR`: A string representing the filesystem path to write output artifacts to. Output artifacts may include checkpoints, graphs, and other files to save, not including model artifacts. These artifacts are compressed and uploaded to S3 to the same S3 prefix as the model artifacts.

`SM_CHANNEL_TRAIN`: A string representing the path to the directory containing data in the ‘train’ channel

`SM_CHANNEL_TEST`: Same as above, but for the ‘test’ channel.

3- Configure the training job by specifying the path of the training script(entry_point), S3 location of the data, 
the compute resources needed (instance_type and instance_count), 
and the output path for the model artifacts(args.model_dir in script.py, the default value is specified by the 
environment variable `SM_MODEL_DIR` which is  A string representing the path to the directory to write 
model artifacts to. These artifacts are uploaded to S3 for model hosting.)
