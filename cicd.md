<!-- Headings -->
# CI/CD workshop for Amazon ECS using Terraform 
## Prerequisites
* *Aws Account*
## Launch Cloud9 in your closest region
* *Navigate to the Cloud9 console*
* *Select Create environment*
Name it cicd-workshop, and select Next Step
* *Select default settings and select Next*
* *select Create Environment*

## Download workshop resources
```
cd ~/environment
git clone https://github.com/aws-samples/cicd-for-ecs-workshop-code.git
```
##  Install Terraform
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
##  Set up Cloud9 IAM credentials
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
##  Create application pipeline
* *Deploy the app pipeline template using*






* *Login to Jenkins*
## Step-3 : Install the Jenkins Plugins
### *Install the plugins in jenkins by navigating to Manage Jenkins -> Manage Plugins -> Available*
* *Search for the plugins*
* *slack Notification Plugin*
* *nexus artifact uploader*
* *nexus platform*
* *git parameter plugin*
* *pipeline utility steps plugin*
* *Artifactory*
* *Maven Release Plug-in*
* *Pipeline Maven Integration Plugin*
* *disk-usage plugin*
* *Dashboard View*
* *Click* * Install without restart
