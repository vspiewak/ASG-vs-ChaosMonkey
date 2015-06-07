Setup Simian Army
=================

Checkout Simian Army project
----------------------------

    git clone https://github.com/Netflix/SimianArmy.git


Install AWS CLI
---------------

    sudo pip install awscli


Setup AWS CLI
-------------

Edit `~/.aws/config`:

    [default]
    aws_access_key_id = ...
    aws_secret_access_key = ...
    region = eu-west-1

Note: You should also set these environment variables:

    export EC2_REGION=eu-west-1
    export AWS_ACCESS_KEY=...
    export AWS_SECRET_KEY=...


Check AWS CLI configuration
---------------------------

You should see a similar output:

    aws autoscaling describe-auto-scaling-groups --output text

    AUTOSCALINGGROUPS
    arn:aws:autoscaling:eu-west-1:010154155802:autoScalingGroup:246a6c20-924b-40f9-85f6-2a61274e12f0:autoScalingGroupName/poc.asg.elasticsearch.autoscalinggroup	poc.asg.elasticsearch.autoscalinggroup	2015-06-07T17:27:56.973Z	300	3	180	ELB	poc.asg.elasticsearch.launchconfiguration	10	3
    AVAILABILITYZONES	eu-west-1a
    AVAILABILITYZONES	eu-west-1b
    AVAILABILITYZONES	eu-west-1c
    INSTANCES	eu-west-1b	Healthy	i-73195fd9	poc.asg.elasticsearch.launchconfiguration	InService
    INSTANCES	eu-west-1c	Healthy	i-7f05ec86	poc.asg.elasticsearch.launchconfiguration	InService
    INSTANCES	eu-west-1a	Healthy	i-a4a67d0e	poc.asg.elasticsearch.launchconfiguration	InService
    LOADBALANCERNAMES	poc-asg-elasticsearch-elb
    TAGS	Name	True	poc.asg.elasticsearch.autoscalinggroup	auto-scaling-group	poc.asg.elasticsearch_node
    TERMINATIONPOLICIES	Default


Setup a Simian Army IAM role (optional)
---------------------------------------

    {
      "Statement": [
        {
          "Sid": "Stmt1357739573947",
          "Action": [
            "ec2:CreateTags",
            "ec2:DeleteSnapshot",
            "ec2:DescribeImages",
            "ec2:DescribeInstances",
            "ec2:DescribeSnapshots",
            "ec2:DescribeVolumes",
            "ec2:TerminateInstances",
            "ses:SendEmail",
            "elasticloadbalancing:*"  
          ],
          "Effect": "Allow",
          "Resource": "*"
        },
        {
          "Sid": "Stmt1357739649609",
          "Action": [
            "autoscaling:DeleteAutoScalingGroup",
            "autoscaling:DescribeAutoScalingGroups",
            "autoscaling:DescribeAutoScalingInstances",
            "autoscaling:DescribeLaunchConfigurations"
          ],
          "Effect": "Allow",
          "Resource": "*"
        },
        {
          "Sid": "Stmt1357739730279",
          "Action": [
            "sdb:BatchDeleteAttributes",
            "sdb:BatchPutAttributes",
            "sdb:DomainMetadata",
            "sdb:GetAttributes",
            "sdb:PutAttributes",
            "sdb:ListDomains",
            "sdb:CreateDomain",
            "sdb:Select"
          ],
          "Effect": "Allow",
          "Resource": "*"
        }
      ]
    }

    
Setup SimpleDB table
--------------------

    sdb() {
            case $(uname -s) in
                Darwin) timestamp=$(date -uj  -f %s $(($(date +%s) - (5 * 60)))  +"%FT%TZ" | sed s/:/%3A/g);;
                Linux)  timestamp=$(date -u +"%FT%TZ" --date='5 min ago' | sed s/:/%3A/g);;
            esac
            params="AWSAccessKeyId=$AWS_ACCESS_KEY"
            params="$params&Action=$1"
            [ -z "$2" ] || params="$params&DomainName=$2"
            params="$params&SignatureMethod=HmacSHA256"
            params="$params&SignatureVersion=2"
            params="$params&Timestamp=$timestamp"
            params="$params&Version=2009-04-15"
            payload="GET\nsdb.eu-west-1.amazonaws.com\n/\n$params"
            hash=$(echo -ne $payload | openssl dgst -sha256 -hmac "$AWS_SECRET_KEY" -binary | base64)
            curl -H application/x-www-form-urlencoded "https://sdb.eu-west-1.amazonaws.com/?$params&Signature=$hash"
        }


Create SIMIAN_ARMY SimpleDB table
---------------------------------

    sdb CreateDomain SIMIAN_ARMY

    <?xml version="1.0"?>
    <CreateDomainResponse xmlns="http://sdb.amazonaws.com/doc/2009-04-15/"><ResponseMetadata><RequestId>3f5b73d7-a612-5673-62b7-f4d0477bc10e</RequestId><BoxUsage>0.0055590278</BoxUsage></ResponseMetadata></CreateDomainResponse>


    sdb ListDomain
    
    <?xml version="1.0"?>
    <ListDomainsResponse xmlns="http://sdb.amazonaws.com/doc/2009-04-15/"><ListDomainsResult><DomainName>ElasticMapReduce-2013-02</DomainName><DomainName>SIMIAN_ARMY</DomainName></ListDomainsResult><ResponseMetadata><RequestId>5e8fc8c7-bad1-07f9-8c47-4651b5ee164d</RequestId><BoxUsage>0.0000071759</BoxUsage></ResponseMetadata></ListDomainsResponse>


Delete SIMIAN_ARMY SimpleDB table
---------------------------------

    sdb DeleteDomain SIMIAN_ARMY

    <?xml version="1.0"?>
    <DeleteDomainResponse xmlns="http://sdb.amazonaws.com/doc/2009-04-15/"><ResponseMetadata><RequestId>75a41c3d-0468-fe97-1b7a-d405fee72ba0</RequestId><BoxUsage>0.0055590278</BoxUsage></ResponseMetadata></DeleteDomainResponse>


Configure Chaos Monkey
----------------------

Configure AWS credentials in `src/main/resources/client.properties` :

* `simianarmy.client.aws.accountKey`
* `simianarmy.client.aws.secretKey`
* `simianarmy.client.aws.region`


Activate Simian Army (always run) in `src/main/resources/simianarmy.properties` :

`simianarmy.calendar.isMonkeyTime = true`


Activate Chaos monkey on your ASG (at every run) in `src/main/resources/chaos.properties` :

`simianarmy.chaos.ASG.poc.asg.elasticsearch.autoscalinggroup.enabled = true`
`simianarmy.chaos.ASG.poc.asg.elasticsearch.autoscalinggroup.probability = 6.0`


You can also desactivate others monkeys:

In conformity.properties:
`simianarmy.conformity.enabled = false`

In janitor.properties:
`simianarmy.janitor.enabled = false`

In volumeTagging.properties:
`simianarmy.volumeTagging.enabled = false`


Run Chaos monkey (leashed, no kill ;) :
----------------------------------------

    ./gradlew jettyRun


Run Chaos monkey as a true hero
-------------------------------

In `src/main/resources/chaos.properties` :

    `simianarmy.chaos.leashed = false`


Then launch as usual:

    ./gradlew jettyRun
