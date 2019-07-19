The traditional way of building AWS environments
When DevOps engineers need to build an infrastructure on AWS cloud, they tend to use CloudFormation for this. CloudFormation is a graphical tool that allows you to draw how your infrastructure should look and behave. CloudFormation can use JSON or YAML files to automate the process.

But there is a number of advantages of using Ansible over CloudFormation:

You already know the tool so, why waste your time learning another one?
While CloudFormation is going to automate building the infrastructure, it will not deploy your application, create users, and so on. Ansible will do that.

Prerequisites
I am assuming that you are using a modern version of Linux like Ubuntu or Centos. You need to have the latest version of Ansible installed.

The following prcedure was not tested on Microsoft Windows or macOS.

We’re going to deploy Apache
You can build almost any sort of environment of AWS no matter how simple or complex it can get. So, in order not to overwhelm you with so much information, we’ll create one EC2 instance from scratch and use it deploy the Apache web server. We’re going to do the following:

Make an AWS account
Create an IAM role and obtain your access and secret keys

Generate a public/private key pair.

Then, using Ansible, we’ll create a playbook that will:

Create a security group for the environment and add the appropriate rules
Launch an EC2 instance based on the type and region

In the second part of the tutorial, we’ll modify the playbook to deploy Apache.

Another playbook will be used to shut down or destroy the environment.
[the_ad id=”369″]

Installing boto3
Firt things first. Ansible depends on the Python module boto3 to communiate with AWS API. So, boto3 needs to be installed on your machine. Issue the following command on your terminal:

pip install boto boto3
Both boto and boto3 packages are needed for this lab.

Storing your keys in Ansible vault
After creating the IAM account, we’ll need to store the AWS keys. Since they are sensitive data, we should use Ansible vault for this:

ansible-vault create aws_keys.yml
Once open, add the following to it:

aws_access_key: AKIAJLHNMCBOITV643UA
aws_secret_key: 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

Setting up the hosts file
Next, we need to create/update the hosts file to handle our new EC2 instance that yet to be created. Adding the following to ./hosts file:

[local]
localhost

Now, we’re ready to run the playbook by issuing the following command:
ansible-playbook -i hosts --ask-vault-pass aws_provisioning.yml -vvv

After the playbook finishes running successfully, you can check your AWS console for a new EC2 instance created and assigned the correct security group.

Further, you can fire up your browser and naviagate to http://ec2-ip, you should see the default Ubuntu page, where ec2-ip is the public IP address that got assigned to your instance by AWS.

Terminating the instance
Unless you are still in the free-tier period offered by Amazon, which lasts for 1 year, you are going to be charged for running the instance on a time basis. So, if you don’t need the instance for the time being or at all, you should stop or terminate it.

The difference between stopping and terminating the instance
Stopping the instance is like issuing the shutdown command. You can start it up again without losing any data. You will not be charged for a stopped instance. You may be charged – however – for other resources related to the intance like storage.

The following playbook is very simple: it will grab all the instances by a specific tag and terminate them. Create a new file called ec2_down.yml.

The playbook starts with declaring the required variables that will be used throughout the file, then it defines two tasks;

ec2_instance_facts: This task is responsible for collecting the instance facts. Don’t confuse this with the traditional fact-gathering that Ansible performs by default when it executes any playbook. Here, Ansible is collecting facts that are related to the presence of this instance on the AWS platform. Facts like the tags that were assigned to the instance are collected, which is what interests us.

ec2: Again, we use the ec2 module, but this time to terminate the instance. The state parameter can take other values than absent depending on your requirements. For example, stopped will just shut down the instance, restarted will reboot it, and running will ensure that it is running (it will start the machine if stopped).


