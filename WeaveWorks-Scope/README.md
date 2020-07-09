# Deploy WeaveWorks Scope on AWS ECS in standalone mode

## Reference:
- https://github.com/weaveworks/scope/blob/master/site/installing.md#ecs

```sh
$ aws cloudformation create-stack \
     --template-body file://WeaveWorks-Scope/ecs-identiorca.yaml \
     --stack-name ecs-weave-scope \
     --parameters ParameterKey=EcsInstanceType,ParameterValue=t2.micro \
     ParameterKey=Scale,ParameterValue=2 \
     ParameterKey=KeyName,ParameterValue=chilcan0_rpi \
     ParameterKey=DeployExampleApp,ParameterValue=No \
     ParameterKey=WeaveCloudServiceToken,ParameterValue="" \
     --capabilities CAPABILITY_IAM 

$ aws cloudformation update-stack \
     --template-body file://WeaveWorks-Scope/ecs-identiorca.yaml \
     --stack-name ecs-weave-scope \
     --parameters ParameterKey=EcsInstanceType,ParameterValue=t2.micro \
     ParameterKey=Scale,ParameterValue=2 \
     ParameterKey=KeyName,ParameterValue=chilcan0_rpi \
     ParameterKey=DeployExampleApp,ParameterValue=Yes \
     ParameterKey=WeaveCloudServiceToken,ParameterValue="" \
     --capabilities CAPABILITY_IAM 
```

## SSH access to EC2 instances

```sh
$ chmod 0400 ~/Downloads/chilcan0_rpi.pem 
$ ll ~/Downloads/chilcan0_rpi.pem
-r-------- 1 pi pi 1672 Jul  9 13:21 /home/pi/Downloads/chilcan0_rpi.pem

$ ssh ec2-user@54.89.161.71 -i ~/Downloads/chilcan0_rpi.pem
Last login: Wed Jun 13 14:38:58 2018 from 37.157.33.76

   __|  __|  __|
   _|  (   \__ \   Amazon ECS-Optimized Amazon Linux AMI 2018.03.f
 ____|\___|____/

For documentation visit, http://aws.amazon.com/documentation/ecs
24 package(s) needed for security, out of 50 available
Run "sudo yum update" to apply all updates.
[ec2-user@ip-172-31-0-184 ~]$ 
[ec2-user@ip-172-31-0-184 ~]$ tail -f /var/log/cloud-init-output.log
  python27-lockfile.noarch 0:0.8-3.5.amzn1
  python27-pystache.noarch 0:0.5.3-2.8.amzn1

Complete!
+ /opt/aws/bin/cfn-init --verbose --stack ecs-weave-scope --region us-east-1 --resource EcsInstanceLc
Error occurred during build: Could not successfully install python packages (return code 1)
Jul 09 17:46:06 cloud-init[2928]: util.py[WARNING]: Failed running /var/lib/cloud/instance/scripts/part-001 [1]
Jul 09 17:46:06 cloud-init[2928]: cc_scripts_user.py[WARNING]: Failed to run module scripts-user (scripts in /var/lib/cloud/instance/scripts)
Jul 09 17:46:06 cloud-init[2928]: util.py[WARNING]: Running module scripts-user (<module 'cloudinit.config.cc_scripts_user' from '/usr/lib/python2.7/dist-packages/cloudinit/config/cc_scripts_user.pyc'>) failed
Cloud-init v. 0.7.6 finished at Thu, 09 Jul 2020 17:46:06 +0000. Datasource DataSourceEc2.  Up 91.47 seconds

[ec2-user@ip-172-31-0-184 ~]$ docker ps
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS               NAMES
21223a4c0215        weaveworks/scope:1.9.0   "/home/weave/entrypo…"   46 minutes ago      Up 46 minutes                           weavescope

[ec2-user@ip-172-31-0-184 ~]$ docker images
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
weaveworks/scope          1.9.0               2174af47dca6        2 years ago         68.3MB
weaveworks/weavedb        latest              b5c2d24a7b71        2 years ago         282B
weaveworks/weaveexec      2.3.0               c2030610fb92        2 years ago         79.1MB
weaveworks/weave          2.3.0               6b9ea60b76e7        2 years ago         61.7MB
amazon/amazon-ecs-agent   latest              8b88a5278f74        2 years ago         9.33MB


$ ssh ec2-user@3.81.53.111 -i ~/Downloads/chilcan0_rpi.pem
The authenticity of host '3.81.53.111 (3.81.53.111)' can't be established.
ECDSA key fingerprint is SHA256:iWLRQb+mYYxknFsisEXLEYlny1Pw6o5IPYSlkRj2VO8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '3.81.53.111' (ECDSA) to the list of known hosts.
Last login: Wed Jun 13 14:38:58 2018 from 37.157.33.76

   __|  __|  __|
   _|  (   \__ \   Amazon ECS-Optimized Amazon Linux AMI 2018.03.f
 ____|\___|____/

For documentation visit, http://aws.amazon.com/documentation/ecs
24 package(s) needed for security, out of 50 available
Run "sudo yum update" to apply all updates.
[ec2-user@ip-172-31-1-20 ~]$
[ec2-user@ip-172-31-1-20 ~]$ tail -f /var/log/cloud-init-output.log
  python27-lockfile.noarch 0:0.8-3.5.amzn1
  python27-pystache.noarch 0:0.5.3-2.8.amzn1

Complete!
+ /opt/aws/bin/cfn-init --verbose --stack ecs-weave-scope --region us-east-1 --resource EcsInstanceLc
Error occurred during build: Could not successfully install python packages (return code 1)
Jul 09 17:46:27 cloud-init[2794]: util.py[WARNING]: Failed running /var/lib/cloud/instance/scripts/part-001 [1]
Jul 09 17:46:27 cloud-init[2794]: cc_scripts_user.py[WARNING]: Failed to run module scripts-user (scripts in /var/lib/cloud/instance/scripts)
Jul 09 17:46:27 cloud-init[2794]: util.py[WARNING]: Running module scripts-user (<module 'cloudinit.config.cc_scripts_user' from '/usr/lib/python2.7/dist-packages/cloudinit/config/cc_scripts_user.pyc'>) failed
Cloud-init v. 0.7.6 finished at Thu, 09 Jul 2020 17:46:27 +0000. Datasource DataSourceEc2.  Up 109.09 seconds

[ec2-user@ip-172-31-1-20 ~]$ docker ps
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS               NAMES
40a8a661d29f        weaveworks/scope:1.9.0   "/home/weave/entrypo…"   50 minutes ago      Up 50 minutes                           weavescope

[ec2-user@ip-172-31-1-20 ~]$ docker images
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
weaveworks/scope          1.9.0               2174af47dca6        2 years ago         68.3MB
weaveworks/weavedb        latest              b5c2d24a7b71        2 years ago         282B
weaveworks/weaveexec      2.3.0               c2030610fb92        2 years ago         79.1MB
weaveworks/weave          2.3.0               6b9ea60b76e7        2 years ago         61.7MB
amazon/amazon-ecs-agent   latest              8b88a5278f74        2 years ago         9.33MB


// Checking the bash scripts created by Cloud-Init
[ec2-user@ip-172-31-1-20 ~]$ ls -la /var/lib/cloud/instance/scripts/

[ec2-user@ip-172-31-1-20 ~]$ sudo cat /var/lib/cloud/instance/scripts/part-001
#!/bin/bash -ex
yum install -y aws-cfn-bootstrap
/opt/aws/bin/cfn-init --verbose --stack ecs-weave-scope --region us-east-1 --resource EcsInstanceLc[ec2-user@ip-172-31-1-20 ~]$
```

If stack is in UPDATE_IN_PROGRESS, then:
```sh
$ aws cloudformation cancel-update-stack --stack-name ecs-weave-scope 
```

```sh
$ aws cloudformation describe-stacks --stack-name ecs-weave-scope | jq -c '.Stacks[].Outputs[] | {k: .OutputKey, v:.OutputValue}'
```

When stack is in UPDATE_ROLLBACK_COMPLETE, then:
```sh
$ aws cloudformation delete-stack --stack-name ecs-weave-scope 
```