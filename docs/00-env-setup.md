# Module 0 <small>Environment Setup</small>

**Time**: 15 minutes

In the first module you will be configuring the initial pipeline and setting up the Anchore service which you will be integrating into the pipeline later on in this workshop.  This module requires you to run an <a href="https://aws.amazon.com/cloudformation/" target="_blank">AWS CloudFormation</a> template, which will automate the creation of the pipeline and Anchore service.  You will then walk through each stage and manually configure the security testing.

## Setup your environment

To setup your environment please expand one of the following dropdown sections (depending on how you're doing this workshop) and follow the instructions: 

??? info  "Click here if you're at an *AWS event* where the *Event Engine* is being used"
	
	1. Navigate to the <a href="https://dashboard.eventengine.run" target="_blank">Event Engine dashboard</a>
	2. Enter the **team hash** code that was distributed to you by the instructors.
	3. Click **AWS Console**.  The CloudFormation template for this round has already been prerun.

??? info  "Click here if you're running this individually in your own AWS Account"

    The CloudFormation template you run will create the Anchore vulnerability scanning service as well as the initial pipeline.  

    Before you deploy the CloudFormation template feel free to view it <a href="https://github.com/aws-samples/aws-container-devsecops-workshop/blob/master/container-devsecops-workshop.yml" target="_blank">**here**</a href>.

    Region| Deploy
    ------|-----
    US East 2 (Ohio) | <a href="https://console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks/new?stackName=container-dso-wksp&templateURL=https://s3.us-east-2.amazonaws.com/sa-security-specialist-workshops-us-east-2/devsecops/containers/container-devsecops-workshop.yml" target="_blank">![Deploy Module 1 in us-east-2](./images/deploy-to-aws.png)</a>

    1. Click the **Deploy to AWS** button above.  This will automatically take you to the console to run the template.

    2. The **Specify an Amazon S3 template URL** is already selected and the template URL is automatically added.  Click **Next**.

    3. On the **Specify Details** click **Next**. 
	
    4. On the **Options** click **Next** \(leave everything on this page as the default\).

    5. Finally, on the **Review** section acknowledge that the template will create IAM roles and CAPABILITY_AUTO_EXPAND and click **Create**.

    ??? question "What is CAPABILITY_AUTO_EXPAND?"
        Some templates contain macros. Macros perform custom processing on templates; this can include simple actions like find-and-replace operations, all the way to extensive transformations of entire templates. Because of this, users typically create a change set from the processed template, so that they can review the changes resulting from the macros before actually creating the stack. If your stack template contains one or more macros, and you choose to create a stack directly from the processed template, without first reviewing the resulting changes in a change set, you must acknowledge this capability.


    This will bring you back to the CloudFormation console. You can refresh the page to see the stack starting to create.

    !!! warning "Before moving on, make sure the stack is in a **CREATE_COMPLETE** status.  This stack takes ~9 minutes."

## Browse to your Cloud9 IDE

You will be doing the majority of the workshop using the <a href="https://aws.amazon.com/cli/" target="_blank">AWS Command Line Interface (CLI)</a> within <a href="https://aws.amazon.com/cloud9/" target="_blank">AWS Cloud9</a>, a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just a browser.

1.	Open the <a href="https://us-east-2.console.aws.amazon.com/cloud9/home?region=us-east-2" target="_blank">AWS Cloud9 console</a> (us-east-2)
2.	Click **Open IDE** in the **container-devsecops-wksp-ide** environment.  This will take you to your IDE in a new tab.  Always keep this tab open 

??? question "Where is my terminal window?"
    If a new terminal session was not opened automatically on creation, go to ```'Window' -> 'New Terminal'```
    Your main working directory will be ```~/environment```

## Clone repositories

1.  Go back to your Cloud9 IDE

2.  Setup your git credentials and clone the repo that contains all the configurations for your pipeline:

```
git config --global credential.helper '!aws codecommit credential-helper $@'
git config --global credential.UseHttpPath true
git clone https://git-codecommit.us-east-2.amazonaws.com/v1/repos/container-devsecops-wksp-config configurations
git clone https://git-codecommit.us-east-2.amazonaws.com/v1/repos/container-devsecops-wksp-app sample-application
echo 'Cloned repositories.'
```

??? question "What is the aws codecommit credential-helper?"
    The credential-helper utility is not designed to be called directly from the AWS CLI. Instead it is intended to be used as a parameter with the git config command to set up your local computer. It enables Git to use HTTPS and a cryptographically signed version of your IAM user credentials or Amazon EC2 instance role whenever Git needs to authenticate with AWS to interact with CodeCommit repositories.

## Enable AWS Security Hub

You will be using AWS Security Hub to manage your container image vulnerabilities.

1.  Enable Security Hub

```
aws securityhub enable-security-hub
```

---

## Pipeline Architecture

You can browse to <a href="https://us-east-2.console.aws.amazon.com/codesuite/codepipeline/pipelines/container-devsecops-wksp-pipeline/view" target="_blank">AWS CodePipeline</a> to view your current pipeline.  All the stages are there but they have not been properly configured.  Below is the current architecture of your pipeline.

![Architecture](./images/00-arch.png "Pipeline Architecture")

After you have successfully setup your environment, you can proceed to the next module.