# Module 1 <small>Add a Dockerfile linting stage</small>

**Time**: 10 minutes

Now that you have your initial pipeline setup, it is time to start integrating security testing.  The first stage you'll add is for doing linting of Dockerfiles to help you build best practice Docker images.  For linting you'll be leveraging <a href="https://github.com/hadolint/hadolint" target="_blank">Hadolint</a>, which is a popular open source project for linting Dockerfiles and validating inline bash. The linter parses the Dockerfile into an AST and performs rules on top of the AST.  The rules aren't all security specific but they have good coverage across best practices.

## View your CodeBuild Project

For each AWS CodePipeline stage you'll be using <a href="https://aws.amazon.com/codebuild/" target="_blank">AWS CodeBuild</a>, which is a continuous integration service that compiles source code, runs tests, and produces software packages that are ready to deploy.  The CodeBuild project for Dockerfile linting has already been created but hasn't been properly configured.  

1.  Click <a href="https://us-east-2.console.aws.amazon.com/codesuite/codebuild/projects/container-devsecops-wksp-build-dockerfile/details?region=us-east-2" target="_blank">here</a> to view your CodeBuild project

## Create the Build Spec file

Each CodeBuild project contains a build specification (build spec) file, which is a collection of build commands and related settings, in YAML format, that CodeBuild uses to run a build.   This is the file where you define the commands for doing Dockerfile linting using Hadolint. 

1.  Click on your Cloud9 IDE tab.

2.  In the left file tree, expand the **configurations** folder and open **buildspec_dockerfile.yml**.

3.  Review the YAML code below, paste it in the file, and save it.

```yaml
version: 0.2

phases:
    pre_build:
        commands:
        - echo "Copying hadolint.yml to the application directory"
        - cp hadolint.yml $CODEBUILD_SRC_DIR_AppSource/hadolint.yml
        - echo "Switching to the application directory"
        - cd $CODEBUILD_SRC_DIR_AppSource
        - echo "Pulling the hadolint docker image"
        - docker pull hadolint/hadolint:v1.16.2
    build:
        commands:
        - echo "Build started on $(date)"
        - echo "Scanning with Hadolint..."        
        - result=`docker run --rm -i -v ${PWD}/hadolint.yml:/.hadolint.yaml hadolint/hadolint:v1.16.2 hadolint -f json - < Dockerfile`
    post_build:
        commands:
        - echo "Lint Results:"
        - echo $result | jq . 
        - aws ssm put-parameter --name "codebuild-dockerfile-results" --type "String" --value "$result" --overwrite
        - echo Build completed on `date`
```

## Add the Hadolint configuration

When using Hadolint you can optionally specify a configuration file to ignore certain rules you might not necessary care about as well as specify trusted registries. 

!!! info "You can view all the current rules by scrolling down on the <a href="https://github.com/hadolint/hadolint" target="_blank">Hadolint</a> github project "

1.  In the left file tree, expand the **configurations** folder and open **hadolint.yml**.

3.  Paste the YAML below and save the file.

```yaml
ignored: 
  - DL3000 
  - DL3025 
 
trustedRegistries: 
  - examplecorp.com 
```
---

## Pipeline Architecture

Below is the current architecture of your pipeline.

![Architecture](./images/01-arch.png "Pipeline Architecture")

After you have successfully configured the Dockerfile linting stage, you can proceed to the next module.