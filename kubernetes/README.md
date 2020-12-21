# Kubernetes

## Team Members:
1. Rajashree Joshi
1. Kinnar Kansara

## Managing Kubernetes clusters with Ansible Playbooks and Kops

This project is supported for Linux (Ubuntu 18.04). For other OS like MacOS or Windows, some system commands will need changes.

### Reuirements
- Python >= 2.7 (Generally available with recent linux distros)
- Ansible = 2.9.x

    ```    
    $ sudo apt update
    $ sudo apt install software-properties-common
    $ sudo apt-add-repository --yes --update ppa:ansible/ansible
    $ sudo apt install ansible
    ```

- AWS profile

    We can use `AWS_ACCESS_KEY_ID` and `AWS_SECRET_KEY` but profile is much better way to avoid passing these keys as parameter or setting them in environment variables. (Ref: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html)

- S3 Bucket

    Create an S3 bucket for storing state of kubernetes cluster. Keep versioning and default encryption enabled. Use this bucket name in below script prefixed by `s3://`

- Route53 DNS Zone

    Create a Route53 public DNS zone for subdomain which is going to be used for this exercise. Replace your subdomain in below shell scripts.

*This playbook installs `kops`, `kubectl`, `boto` and `boto3` packages if they're not installed in the system. These packages are required to run all the tasks but they're installed within the playbook so not mentioned above.*

### Create Cluster

Create shell script preferably with file name `setup-k8s-cluster.sh` and keep the below content in the file:
```
#!/bin/bash

export AWS_PROFILE=profile_name

export AWS_REGION=us-east-1
export KOPS_CLUSTER_NAME=k8s.prod.example.com
export KOPS_STATE_STORE=s3://kops-cluster-prod-example-com
export DNS_ZONE=k8s.prod.example.com
export SSH_PRIVATE_KEY=~/.ssh/id_rsa_kops
export METRICS_SERVER_FILE_PATH=/path/to/metrics/server/yaml
export SUB_DOMAIN_NAME=webapp
export EMAIL=me@example.com

ANSIBLE_STDOUT_CALLBACK=debug ansible-playbook setup-k8s-cluster.yml --extra-vars "aws_region=${AWS_REGION} kops_cluster_name=${KOPS_CLUSTER_NAME} kops_state_store=${KOPS_STATE_STORE} dns_zone=${DNS_ZONE} ssh_private_key=${SSH_PRIVATE_KEY} metrics_server_file_path=${METRICS_SERVER_FILE_PATH} email=${EMAIL} sub_domain_name=${SUB_DOMAIN_NAME}"
```

**Important**: Make sure to execute the above script with elevated privileges i.e. with `sudo` because the playbook tries to install `boto` and `boto3` packages with `apt` which requires to be executed as root user. This is executable file so make sure that execute bit is set. Use `chmod +x setup-k8s-cluster.sh` to make it executable.

At the end of execution, this will give ELB URL for Bastion host. Use that to ssh into Bastion which in turn will be able to access Master an Compute nodes. Make sure the ELB URL has `bastion` prefix.

For getting the internal IP addresses of Master and Compute nodes, use this command:
`kops cluster validate` or `kubectl get nodes`

### Access Bastion Host
Use below command to access Bastion. Replace `bastion-elb-url` with the one got from above execution.
```
ssh -A ubuntu@bastion-elb-url
```

After getting into Bastion, use the same command as above with internal IP address of nodes to ssh into the each EC2 instances.

### Delete Cluster
Create shell script preferably with file name `delete-k8s-cluster.sh` and keep the below content in the file:
```
#!/bin/bash

export AWS_PROFILE=profile_name

export AWS_REGION=us-east-1
export KOPS_CLUSTER_NAME=k8s.prod.example.com
export KOPS_STATE_STORE=s3://kops-cluster-prod-example-com

ANSIBLE_STDOUT_CALLBACK=debug ansible-playbook delete-k8s-cluster.yml \
    --extra-vars "kops_cluster_name=$KOPS_CLUSTER_NAME kops_state_store=$KOPS_STATE_STORE"
```

This will delete the previously created kubernetes cluster