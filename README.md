# OCP AWS Miser

Small set of ansible scripts that will do the following:

1) use your aws credentials to query all running EC2 instances
2) create three seperate lists of machines, "masters", "workers" and "windows nodes"
3) If starting the cluster
    a) Start the master nodes
    b) wait 10 minutes for the control plane to settle
    c) Start the worker nodes
    d) Start the windows worker nodes
4) If Stopping the cluster
    a) stop the Windows worker nodes
    b) stop the Worker nodes
    c) stop the control plane nodes

## Configuring aws credentials

If you are managing multiple accounts, you need to create **aws_profile**s for each set of credentials you have. Your `~/.aws/credentials` will need to contain sections for each profile:

```
[boldwright]
aws_access_key_id     = <access_key_here>
aws_secret_access_key = <secret_key_here>

[summit]
aws_access_key_id     = <access_key_here>
aws_secret_access_key = <secret_key_here>

[default]
aws_access_key_id     = <access_key_here>
aws_secret_access_key = <secret_key_here>
```

## Using the scripts

The scripts rely on two Ansible variables:

* _ec2_region_ - defines the region your cluster is in
* _aws_profile_ - select the proper authentication information from the aws credentials file (use "default" if only one cred is defined)

```
$ /bin/ansible-playbook -e "ec2_region=us-east-2" -e "aws_profile=boldwright" ansible/ocp_ec2_stop.yml
```

### Running from crontab

If you want to automate the power off/power on an easy way to do this is with cron. The following would stop "boldwright" at 9pm, and start it at 9am Monday through Friday. It would stop "summit" at 6pm and start it at 7 am Monday through Friday.

```crontab
0 21 * * 1-5 /bin/ansible-playbook -e "ec2_region=us-east-2" -e "aws_profile=boldwright" ./ansible/ocp_ec2_stop.yml >> ~/boldwright.log 2>&1 
0 18 * * 1-5 /bin/ansible-playbook -e "ec2_region=ca-central-1" -e "aws_profile=summit" ./ansible/ocp_ec2_stop.yml >> ~/summit.log 2>&1 
0 09 * * 1-5 /bin/ansible-playbook -e "ec2_region=us-east-2" -e "aws_profile=boldwright" ./ansible/ocp_ec2_start.yml >> ~/boldwright.log 2>&1
0 07 * * 1-5 /bin/ansible-playbook -e "ec2_region=ca-central-1" -e "aws_profile=summit" ./ansible/ocp_ec2_start.yml >> ~/summit.log 2>&1
```
