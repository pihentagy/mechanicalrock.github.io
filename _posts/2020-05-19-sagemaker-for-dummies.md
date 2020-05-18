---
layout: post
title: Sagemaker for Dummies
date: 2020-05-19
tags: aws sagemaker ml machinelearning
author: Zainab Maleki
image: img/blog/sagemaker-for-dummies/header.png
---


With todays machine learning enhancements, there are countless opportunities where you could utilise its benifit.

## Everything to know about Sagemaker

**SageMaker Notebook Instance:** 
A fully managed ML compute instance running the Jupyter Notebook App. You can use Jupyter notebooks to prepare and process data, write code to train models, deploy models to Amazon SageMaker hosting, and test or validate your models. It also provides a lot of sample ML algorithms that could be reused as well as having access to terminal on the instance to install additional packages.

**Instance Lifecycle policies:**
Custom bash scripts that can run during the instance creation or start. Maximum timeout is 15 minutes. if require more than that you have to specify & before the command so it can run in the background when the instance starts.


**SageMaker Elastic Inference:**
GPUs used to train a model or run an existing model.


**Different ways to build training models algorithms on SageMaker:**

Built-in algorithms

Script Mode
Amazon managed container supports one of below. You write the code for your model on the container then it runs on the SageMaker training jobs. Specify entry-point in the Sagemaker estimator. Include any extra libraries in the requirement.txt. Uses managed webserver with inference 
1. MXnet
2. TensorFlow 
3. Pytorch
4. Scikit Learn
5. Spark MLlib
6. Chainer

Docker Container 
Bringing your own container. You have to manage creation of the container and pushing it to ECR

AWS ML MarketPlace
200 + example algorithms hosted on aws ECR that are mostly free

Notebook Instance

 


**Training a Modal:**

Every model running on Sagemaker training job has its own ephemeral cluster. Each training job should have its own dedicated EC2 instance that is running for the number of seconds that the model needs to train. The cluster automatically stops after the training job is finished.

You will specify the training job configurations on the SageMaker Estimator.


Container is where the model algorithm is. Either bring your own container hosted on ECR or script mode container or marketplace containers 



**Splitting the data and passing them into training jobs:**

We have to split the data to 3 different groups (training data, validation data and test data) and each group should be in a different s3 location. Then we should create data channel object and pass it to model.fit function which basically starts the cluster to start the training job.


 



**Hyper Parameter Tuning:**

In the previous example you only run a single training job on your data, however in real life you would like to run multiple training jobs using different hyper parameters to identify which parameter enhances the model performance the best way. SageMaker provides hyper parameter tuning where you specify a range for your hyper parameters and other configurations such as parameter tuning method or number of training job in parallel. Then you will have to attach the hyper parameter configs to the sagemaker and run the training jobs using tuner.fit


xgb is the Estimator.




**Deploying the model:**

Sagemaker uses endpoints which is in front of EC2 instance or instances where the model is running. All you will have to do is to run the model.deploy command and it will give you a managed endpoint with ec2 instances in multiple AZs with a autoscaling group


 


**SageMaker Ground Truth:**
It’s an amazon managed ML program to help data scientists to label their dataset before feeding it into their own ML model. 

Is Machine Learning a new and scary world to step into? We can help!

[Contact Mechanical Rock to Get Started!](https://www.mechanicalrock.io/lets-get-started)
