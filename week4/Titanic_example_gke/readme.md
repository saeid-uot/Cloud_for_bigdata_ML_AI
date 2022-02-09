## Introduction

This sample code illustrates how to start a *Google Cloud Engine (GCE)* instance and train a model using scikit-learn on the instance. We will be using the Titanic dataset. The goal is to train a classifier to predict whether a given passenger on Titanic survived or not.

## Prerequisites

Before we start, we need to:

* Use the data in train.csv 
 
* have a [*GCP*](https://cloud.google.com/) account, and [download](https://cloud.google.com/sdk/), install, and [configure](https://cloud.google.com/sdk/gcloud/reference/config/) the *Google Cloud SDK* on our local computer.

* [create a project](https://cloud.google.com/sdk/gcloud/reference/projects/create) on *GCP* and set the project, region, and zone properties as instructed [here](https://cloud.google.com/sdk/gcloud/reference/config/set).

* set these environment variables on the local computer:
	* $MY_INSTANCE: the name of the new instance (e.g. ```export MY_INSTANCE=titanic_trainer```).
	* $MY_BUCKET: the bucket name to be created on *Google Cloud Storage (GCS)* (e.g. ```export MY_BUCKET=titanic_bucket```).
	* $MY_FOLDER: the folder name to be used in the bucket (e.g. ``` export MY_FOLDER=titanic_folder```).

## Steps:
We will create a [VM instance](https://cloud.google.com/compute/docs/instances/), configure it, and use it to train a model. While some of the steps can be done using the web portal [here](https://pantheon.corp.google.com/compute/instances), we will try to accomplish this using the *SDK* and mainly the [*gcloud*](https://cloud.google.com/sdk/gcloud/) command. Unless otherwise specified as a comment, all the commands are to be run locally.

##### 1. Create a Google Compute Engine Instance
First, we should create a *GCE* instance that we can use to train our model on. In this case, we can rely on many of the default arguments:

```bash
gcloud compute instances create $MY_INSTANCE
```
which creates an instance of type *n1-standard-1* (1 CPU, 3.75GB of RAM), with a *Debian* disk image and no scopes. This is fine as long as we access the dataset locally and not through *GCS*. Otherwise, we will have to provide the right scope:

```bash
gcloud compute instances create $MY_INSTANCE --scopes storage-rw
```

The *scopes* argument grants the instance the right permission to access *GCS*.

We can also specify a different machine type or image. For example, the following command creates an *Ubuntu* instance of type *n1-standard-2*:

```bash
gcloud compute instances create $MY_INSTANCE --machine-type=n1-standard-2 --scopes storage-rw
```
You may want to learn more about the [machine types](https://cloud.google.com/compute/docs/machine-types) and  [images](https://cloud.google.com/compute/docs/images), or see a list of available machine types and images with the following two commands:

```bash
gcloud compute machine-types list
gcloud compute images list
```

A more detailed instruction about *gcloud compute instance create* can be found [here](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create).

##### 2. Accessing and Configuring the Instance

Accessing the newly created instance is quite simple:
```bash
gcloud compute ssh $MY_INSTANCE
```
This command may prompt us to create a *ssh* key, if we have not done it before on our local machine.

Once we *ssh* to the instance, we will need to use *pip* to install *scikit-learn* and some other packages:
```bash
# Run on the new instance
sudo apt-get install -y python-pip
pip install numpy pandas sklearn scipy tensorflow
```
*Note:* If you choose a different image for the instance, you may need to do something different to install *pip* and the required python packages.

Also note that we are not using *tensorflow* to train the model in this sample. We will however use *gfile*, a python class implemented in *tensorflow* which unifies how we access the local and remote files. An alternative to using *gfile*  for accessing the files stored in *GCS* is [Cloud Storage Client Libraries](https://cloud.google.com/storage/docs/reference/libraries#client-libraries-install-python).

##### 3. Running the Code
We need to make the dataset available to the training code. Here are two options for the code to access the dataset.

###### Option 1: Copying it to the instance
We can copy the dataset into the new instance and use it directly inside the instance and read it as a local file. Fortunately, *gcloud* makes this an easy step:
```bash
gcloud compute scp ./train.csv $MY_INSTANCE:~/
```

The same method can be used to copy any other file (e.g. the python code):
```bash
gcloud compute scp ./titanic.py $MY_INSTANCE:~/
```

We can then run our python code on the instance to create a model:
```bash
# Run on the new instance
python titanic.py --titanic-data-path ~/train.csv --model-output-path ~/model.pkl
```

which will generate the model and save it on the instance.


##### 4. Stopping or Deleting the Instance
The *GCE* instance that we created will be running indefinitely and incurring charges, unless we either [stop or delete it](https://cloud.google.com/compute/docs/instances/stopping-or-deleting-an-instance). We are not charged for a stopped instance, but we may be charged for the resources attached to it. Stopping the instance will allow us to start it again later on, should we require to use it again, which may be the better choice:
```bash
gcloud compute instances stop $MY_INSTANCE
```
We can also stop the instance while we are logged into it:
```bash
# Run on the new instance
sudo shutdown -h now
```
Since the instance has only stopped, and not deleted, we can start it again:
```bash
gcloud compute instances start $MY_INSTANCE
```
However, if we want to completely delete the instance (and all the resources attached to it), we can use the following command:
```bash
gcloud compute instances delete $MY_INSTANCE
```
That is it!! We have successfully trained a classifier using scikit-learn on a *GCE* instance.
