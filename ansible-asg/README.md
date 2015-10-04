Ansible EC2 Autoscaling demo
============================

Install an Elasticsearch cluster on EC2 with Autoscaling features


Install Ansible
---------------
(On OSX for instance...)

    sudo easy_install pip
    sudo pip install --upgrade pip
    sudo -H pip install --upgrade boto
    sudo -H pip install --upgrade ansible


Configure your AWS Credentials
------------------------------

    export ANSIBLE_HOST_KEY_CHECKING=False
    export EC2_REGION=eu-west-1

    export AWS_ACCESS_KEY=...
    export AWS_SECRET_KEY=...
    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY
    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_KEY

Configure your EC2 pem file
---------------------------

    export PEM_NAME=<aws_pem_name>
    export PEM_PATH=~/.ssh/<aws_pem_name>.pem


Configure EC2 tags
------------------

    export CLIENT=poc
    export ENV=asg


Configure your VPC
------------------

Use the default VPC or create one with a public subnet

    export VPC_ID=<vpc_id>
    export SUBNET_ID=<public_subnet_id>


Configure IAM Role for EC2 discovery
------------------------------------

Create an IAM account with the following permissions:

    {
        "Statement": [
            {
                "Action": [
                    "ec2:DescribeInstances"
                ],
                "Effect": "Allow",
                "Resource": [
                    "*"
                ]
            }
        ],
        "Version": "2012-10-17"
    }


Configure EC2 discovery with this IAM account:

    export ES_DISCOVERY_AWS_REGION=eu-west
    export ES_DISCOVERY_AWS_ACCESS_KEY=...
    export ES_DISCOVERY_AWS_SECRET_KEY=...


See: <https://github.com/elastic/elasticsearch-cloud-aws>


Create EC2 instance
-------------------

    ansible-playbook -i inventory/local playbooks/create-ec2.yml


Configure EC2 instance
----------------------

    ansible-playbook -i inventory/ec2.py playbooks/elasticsearch.yml


Create EC2 AMI
--------------

    ansible-playbook -i inventory/ec2.py playbooks/create-ami.yml


Configure EC2 ELB + LC + ASG
----------------------------

    export AMI_IMAGE_ID=ami-...

    ansible-playbook -i inventory/ec2.py playbooks/create-asg.yml

For demo, configure ASG with the following values :
* Default Cooldown: 5
* Health Check Grace Period: 0


You can check your cluster
--------------------------

`http://<lb_url>9200/_plugin/head`


Destroy EC2 infrastructure
--------------------------

    ansible-playbook -i inventory/ec2.py playbooks/terminate.yml
