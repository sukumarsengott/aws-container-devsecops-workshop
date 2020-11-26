# Module 3 <small>Add a vulnerability scanning stage</small>

**Time**: 10 minutes

The last stage you will add will be for identifying vulnerabilities in your container image.  For this stage you'll be using <a href="https://anchore.com/opensource/" target="_blank">Anchore</a>, a popular open source container compliance platform.  This service can do a number of different validations but you will be primarily using it for checking your image for any Common Vulnerabilities and Exposures (CVE).

## Create the Build Spec file

1.  Click on your Cloud9 IDE tab.

2.  In the left file tree, expand the **configurations** folder and open **buildspec_vuln.yml**.

3.  Review the YAML code below, paste it in the file, and save.

```yaml
version: 0.2

phases: 
  pre_build: 
    commands:
      - apt-get update && apt-get install -y python-dev jq
      - docker pull anchore/engine-cli:v0.8.2
      - curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py  
      - python get-pip.py
      - pip install awscli
      - $(aws ecr get-login --no-include-email)
      - ANCHORE_CMD="docker run -e ANCHORE_CLI_URL=$ANCHORE_CLI_URL -e ANCHORE_CLI_USER=$ANCHORE_CLI_USER -e ANCHORE_CLI_PASS=$ANCHORE_CLI_PASS anchore/engine-cli:v0.8.2 anchore-cli"
      - $ANCHORE_CMD registry add $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com awsauto awsauto --registry-type=awsecr || return 0
  build: 
    commands:
      - IMAGE=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME
      - docker build $CODEBUILD_SRC_DIR_AppSource -t $IMAGE
      - docker push $IMAGE
  post_build:
    commands:
      - $ANCHORE_CMD image add $IMAGE
      - while [ $($ANCHORE_CMD --json image get $IMAGE | jq -r '.[0].analysis_status') != "analyzed" ]; do sleep 1; done
      - $ANCHORE_CMD --json image vuln $IMAGE all > scan_results.json
      - jq -c --arg image $IMAGE --arg arn $IMAGE_ARN '. + {image_id:$image, image_arn:$arn}' scan_results.json >> tmp.json
      - mv tmp.json scan_results.json
      - aws lambda invoke --function-name $FUNCTION_ARN --invocation-type RequestResponse --payload file://scan_results.json outfile
      - cat scan_results.json |  jq -r --arg threshold $FAIL_WHEN '.vulnerabilities[] | select(.severity==$threshold)'
      - if cat scan_results.json |  jq -r --arg threshold $FAIL_WHEN '.vulnerabilities[] | (.severity==$threshold)' | grep -q true; then echo "Vulnerabilties Found" && exit 1; fi
```

## Commit all configuration changes

Since you've made changes to a number of files in the configuration repo, you need to commit those changes to ensure your pipeline is pulling in the right files.

```
cd /home/ec2-user/environment/configurations
git add .
git commit -m "Updated Build Spec files and configurations."
git push -u origin master
```

---

## Pipeline Architecture

Below is the current architecture of your pipeline.

![Architecture](./images/03-arch.png "Pipeline Architecture")

After you have successfully configured the secrets scanning stage, you can proceed to the next module.