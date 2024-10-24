---
layout: post
title:  "How to Train a PyTorch model using an EC2 instance"
date:   2024-10-17 12:16:43 -0700
categories: aws pytorch ec2 machine-learning training
--- 
# How to Train a PyTorch model using an EC2 instance

## Introduction

Cloud computing is an essential concept to understand for machine learning at a large scale. 
For example, ChatGPT contains about 178 <strong>billion</strong> parameters, which can be estimated to be ~712 terabytes if each parameter is 4 bytes.... 
In this article, I will be going over how to set up an EC2 instance for PyTorch model training.
If you are unfamiliar with cloud computing, I recommend checking out Amazon's definition of [cloud computing](https://aws.amazon.com/what-is-cloud-computing/).

I will be focusing on Amazon Web Services (AWS) since they're currently the biggest cloud computing market shareholder, as well as being the most desired in the job listings I've seen.

## Procedure

### 1. Set up a Github repository

The easiest method to get code on an EC2 instance (that I've seen) is to pull a Github repository.
Therefore, we will want to create a Github repo and populate it with an untrained PyTorch model.
Follow Github's official [guide]() on setting up a repo.

<strong>However, if you are planning on using a model that is already available as a repo, skip to step three.</strong>

### 2. Implement your PyTorch model

My repo is inspired by the official [PyTorch MNIST example](https://github.com/pytorch/examples/tree/main/mnist) model repo.
If you are not familiar with implementing 

### 3. Set up EC2 instance

1. Give non-adminstrator users permission to access the Elastic Container Registry (ECR) by adding the <em>ECS_FullAccess</em> permission to the desired user(s) or group(s).

2. Launch a new EC2 instance.

    a. Select an AMI image of your choice. 
    I recommend using '<em>Amazon Linux 2023 AMI</em>' since it is free and most deep learning dependencies will be handled by the Docker container.

    b. Select an instance type. 
    I recommend using a GPU-based instance, but they cost money. 
    I used the <em>t2.micro</em> instance type since it is free.

    c. Assign a key-pair to the instance. 
    If you are creating a new key-pair, I recommend creating an RSA key-pair type that uses the .pem file type.

3. Connect to the instance via SSH.

    a. Navigate to the <em> Instance Summary </em> for the instance and copy the <em>Public IPv4 DNS</em> address.
    <strong>NOTE: The value is dynamic and will change frequently.</strong>

    b. Use the following command to SSH into the instance:

    ```console
    ssh -L localhost:8888:localhost:8888 -i <.pem filepath> ec2-user@<Public IPv4 DNS of instance>
    ```

4. Install Docker.

    ```console
    sudo yum update -y                  # Use for Linux 2023 AMI.
    sudo pkill -f "apt.systemd.daily"
    sudo yum install -y docker          # Use for Linux 2023 AMI.
    sudo usermod -a -G docker ec2-user
    ```

5. Set up the AWS CLI.

    a. Create an access key-pair for the EC2 instance. 
    This will allow you to access the AWS console via CLI.
    Follow AWS's official [guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html) on creating access keys.

    b. Configure the EC2 instance to use your Access Key ID and Secret Access Key:

    ```console
    aws configure
    ```

    - `Default region name` and `Default output format` can be skipped by pressing Enter.

6. Run AWS deep learning containers.

    ```console
    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-training:2.4.0-cpu-py311-ubuntu22.04-ec2
    ```

    - This command will initialize the PyTorch CPU Training container; modify the command using the table below if you want to initialize the PyTorch GPU Training container.

    - There exists multiple containers for multiple configurations which be found [here](https://github.com/aws/deep-learning-containers).
    The table below contains the different PyTorch training containers that are available.

    |Framework|Job Type|Horovod Options|CPU/GPU|Python Version Options|Example URL|
    |---------|--------|---------------|-------|----------------------|-----------|
    |PyTorch 2.4.0|training|No|CPU|3.11 (py311)|763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-training:2.4.0-cpu-py311-ubuntu22.04-ec2|
    |PyTorch 2.4.0|training|No|GPU|3.11 (py311)|763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-training:2.4.0-gpu-py311-cu124-ubuntu22.04-ec2|

7. Fetch and train the desired model.

    ```console
    git clone https://github.com/chancetran/python-pytorch-cnn-mnist.git
    python3 ~/python-pytorch-cnn-mnist/train.py
    ```

### Optional:

8. Detach the container to allow for background execution.

    - Detach container: Press the shortcut `^P^Q`.
    This will 'detach' the container from your current SSH session.
    You can terminate your SSH session and the container will still be running.

    - Attach container: 
    Enter the bash command `docker attach <container name>`.
    This will 'reattach' the container to your current SSH session.

9. <strong>Stop your instance after training is complete.</strong>

## Conclusion

Congratulations!
You are now ready to train any PyTorch model you want using EC2 instances.
In my next post, I plan to discuss how to set up the EC2 instance to download and upload data to an S3 bucket.


## Citations
- [Train a Deep Learning Model with AWS Deep Learning Containers on Amazon EC2 Tutorial by Amazon](https://aws.amazon.com/tutorials/train-deep-learning-model-aws-ec2-containers/)
- [AWS Deep Learning Containers - Amazon EC2 Tutorials by Amazon](https://docs.aws.amazon.com/deep-learning-containers/latest/devguide/deep-learning-containers-ec2.html)
- [AWS Serverless Application Model - Installing Docker to use with the AWS SAM CLI by Amazon](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-docker.html)
- [Manage access keys for IAM users by Amazon](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html)
- [Deep Learning Containers by AWS](https://github.com/aws/deep-learning-containers)