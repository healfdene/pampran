Pampran
=======

Packer script to create an AMI with MySQL, Papertrails, Redis, Node.js, and NewRelic installed.

The AMI contains scripts that will poll an specified S3 bucket for a node package, in the form of a tar file. When a new version appears in the S3 bucket, it will be installed and started with npm. If a previous version of the server is running, its process will be killed before the new version is started.

Requirements
------------
To use this, you must:

0. Have an interest in using node.js with Redis and MySQL
0. Have an AWS account
0. Have a Papertrail account (see papertrailapp.com)
0. Fork or clone this repo onto your dev machine
0. Have packer installed on your local system (linux or OSX, see packer.io for install instructions)

Procedure
---------
#### Step 0, AWS CLI

Make sure you local system has the AWS CLI installed and working. See http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html for help with this.

#### Step 1, Packer build
In your clone of this repo,

    cd pampran
    packer build packer.json

This will create an AMI that can run your node/redis/mysql app. This is intended for low-cost application testing, as it runs everything in one instance. A future project (pampran-cf) will include AWS CloudFormation templates appropriate for a production deployment of an app using reliable, separate deployment of redis and mysql and deployment of the node servers behind ELB with an autoscaling group.

#### Step 2, Create IAM role
To create an instance for testing, first create an IAM profile for AmazonS3ReadOnlyAccess. (See http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html for help with this.)

#### Step 3, Create S3 bucket

In this example, we create a bucket for the node package in the test directory called 'simple'. Keep in mind that S3 bucket names must be globally unique, even if they are private buckets, as in this example. You need to use your own bucket name, and make it weird enough to be globally unique or the command will fail

`aws s3 mb s3://simple.freak-domain.party`

#### Step 4, Upload your node package to the bucket

We provide a shell script for this. The script uploads into the bucket a file named 'version' which consists of three lines:

1. A unique version id, which must be a valid directory name. The script uses the current date.
2. The name of the tar file for the npm package
3. The name of the package (for npm Starting)

It also uploads the npm package as a tar file. The script runs `npm pack` to create the tar file. Example:

`cd test/simple`
`../../tools/upload.sh simple.freak-domain.party`

#### Step 5, Start an EC2 instance

[]