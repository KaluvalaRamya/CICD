<!-- Headings -->
# CI/CD workshop for Amazon ECS using Terraform 
## Prerequisites
* *Aws Account*
## **Launch Cloud9 in your region**
* *Navigate to the Cloud9 console*
* *Select Create environment*
Name it cicd-workshop, and select Next Step
* *Select default settings and select Next*
* *select Create Environment*
## **Install Tools & Resources**
* *Click on this [link](https://catalog.us-east-1.prod.workshops.aws/v2/workshops/869f7eee-d3a2-490b-bf9a-ac90a8fb2d36/en-US/3-setup/01-tools-resources) and follow the steps.*
##  **Install Terraform**
<!-- Blockquote -->
<!-- italics -->
* *Download and unzip*
```
cd ~/environment
wget https://releases.hashicorp.com/terraform/0.14.6/terraform_0.14.6_linux_amd64.zip
unzip terraform_0.14.6_linux_amd64.zip

```
* *Install in /usr/local/bin*
```
sudo mv terraform /usr/local/bin
```
* *Verify that terraform*
```
which terraform
```
##  **Set up Cloud9 IAM credentials**
* Create an IAM role for your Workspace

    1. Click this [link](https://console.aws.amazon.com/iam/home?#/roles$new?step=type&commonUseCase=EC2%2BEC2&selectedUseCase=EC2&policies=arn:aws:iam::aws:policy%2FAdministratorAccess)
    2. Confirm that AWS service and EC2 are selected, then click Next to view permissions.
    3. Confirm that AdministratorAccess is checked, then click Next: Tags to assign tags.
    4. Take the defaults, and click Next: Review to review.
    5. Enter workshop-admin for the Name, and click Create role.
* Attach the IAM role to your Workspace
    1. Click this [link](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Instances:tag:Name=aws-cloud9-;sort=desc:launchTime) 
    2. Select the instance, then choose Actions / Security / Modify IAM Role
    3. Choose workshop-admin from the IAM Role drop down, and select Apply
* Update IAM settings for your Workspace
    1. Go to workspace and click on settings symbol at top right corner
    2. Select AWS SETTINGS
    3. Turn off AWS managed temporary credentials
    4. Close the Preferences tab
##  **Create application pipeline**
### Deploy the application pipeline
```
cd ~/environment/cicd-for-ecs-workshop-code/lab-terraform/tf/app-pipeline
terraform init
terraform apply \
  -var 'source_repo_name=tf-simple-server' \
  -var 'source_repo_branch=master' \
  -var 'image_repo_name=lab-terraform/simple-server'
```
1. *Type yes at the prompt*
2. *Use the AWS CodePipeline console to monitor the pipeline. At this point it should be in the Failed state, as there is no source to pull*
### *Populate the source repo*
```
export tf_source_repo_clone_url_http=$(terraform output -raw source_repo_clone_url_http)
```
### *Create a local git repo and copy in the application files*
```
cd ~/environment
mkdir simple-server
cd simple-server
cp -r ~/environment/cicd-for-ecs-workshop-code/app/simple-server/* .
git init
git add .
git commit -m "Initial commit"
```
### *Set the CodeCommit repo as remote*
```
git remote add origin $tf_source_repo_clone_url_http
git remote -v
```
### *Push master branch to remote*
```
git push -u origin master
```
* *This triggers the pipeline to pull the code, build the docker image, and push it to ECR.*
* *You can monitor the pipeline in the AWS CodePipeline console*
##  **Create namespace**
* *Create a namespace called lab-tf in the production VPC as follows*
```
export PROD_VPC=$(aws cloudformation describe-stacks \
  --stack-name prod-cluster \
  --query "Stacks[0].Outputs[?OutputKey == 'VpcId'].OutputValue" \
  --output text \
)
aws servicediscovery create-private-dns-namespace --name lab-tf --vpc $PROD_VPC
```
* *Verify that the namespace has been created*
```
aws servicediscovery list-namespaces
```
* *If the response is empty, wait a few seconds and retry*
```
export LAB_TF_NAMESPACE_ID=$( \
    aws servicediscovery list-namespaces \
    --query "Namespaces[?Name == 'lab-tf'].Id" \
    --output text \
)
echo $LAB_TF_NAMESPACE_ID
```
* *Note the namespace id for the lab-tf namespace.You will need it when configuring your service pipelines.*
##  **Create service pipelines**
```
cd ~/environment/cicd-for-ecs-workshop-code/lab-terraform/tf/service-pipelines
terraform init
terraform apply
```
* *Once complete, use the CodePipeline console to verify that all three pipelines have been created.*
* *Note the http URLs for each of the service source repos. Set environment variables to these values*
```
export tf_front_source_clone_url_http=$( \
  terraform output -raw front_source_repo_clone_url_http \
)
export tf_middle_source_clone_url_http=$( \
  terraform output -raw middle_source_repo_clone_url_http \
)
export tf_back_source_clone_url_http=$( \
  terraform output -raw back_source_repo_clone_url_http \
)
```
* *When the stack creation has completed, create a local repo for the tf-front service*
```
cd ~/environment
mkdir tf-front-service
cd tf-front-service
cp -r ~/environment/cicd-for-ecs-workshop-code/lab-terraform/config/front-service/* .
git init
```
* *Edit the tf-front-service terraform.tfvars file.Fill in the correct value for namespace_id, which we have otained above*
* *Add all files and commit*
```
git add .
git commit -m "Initial version"
```
* *set up the pipeline CodeCommit repo as remote*
```
git remote add origin $tf_front_source_clone_url_http
git remote -v
```
* *Push the source files*
```
git push -u origin master
```
* *This triggers the pipeline to pull the service configuration, and to deploy the service.*
## **Check the front service deployment**
* *You can monitor the deployment using the [CodePipeline console](https://console.aws.amazon.com/codesuite/home?&service=codepipeline&returnUrl=https%3A%2F%2Fconsole.aws.amazon.com%2Fcodepipeline)*
* *You can also use the [ECS Console](https://console.aws.amazon.com/ecs) to observe the tasks being created and deployed.*
* *Once the service is deployed you can use the [Route 53 console](https://console.aws.amazon.com/route53) to verify that you can see a private hosted zone for lab-tf, with entries for the tf-front service.*