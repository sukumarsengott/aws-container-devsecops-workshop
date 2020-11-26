# Module 2 <small>Add a secrets scanning stage</small>

**Time**: 10 minutes

Next, you need to setup a stage for identifying secrets throughout your code.  For this stage you'll be leveraging <a href="https://github.com/dxa4481/truffleHog" target="_blank">trufflehog</a>, a popular open source project for finding secrets accidentally committed in repositories.  It essentially searches through git repositories for secrets, digging deep into commit history and branches.  It identifies secrets by running entropy checks as well as high signal regex checks. 

## Create the Build Spec file

1.  Click on your Cloud9 IDE tab.

2.  In the left file tree, expand the **configurations** folder and open **buildspec_secrets.yml**.

3.  Review the YAML code below, paste it in the file, and save.

```yaml
version: 0.2

phases:
  pre_build:
    commands:
    - echo "Setting CodeCommit Credentials"
    - git config --global credential.helper '!aws codecommit credential-helper $@'
    - git config --global credential.UseHttpPath true
    - echo "Copying secrets_config.json to the application directory"
    - cp secrets_config.json $CODEBUILD_SRC_DIR_AppSource/secrets_config.json
    - echo "Switching to the application directory"
    - echo "Installing truffleHog"
    - which pip3 && pip3 --version
    - which python3 && python3 --version
    - pip3 install 'truffleHog>=2.1.0,<3.0'
  build:
    commands:
    - echo "Build started on $(date)"
    - echo "Scanning with truffleHog..."
    - trufflehog --regex --rules secrets_config.json --entropy=False "$APP_REPO_URL"
  post_build:
    commands:
    - echo "Build completed on $(date)"
```

## Add the trufflehog regex configuration

When using trufflehog you can optionally specify a configuration file that contains custom regex checks.

1.  In the left file tree, expand the **configurations** folder and open **secrets_config.json**.

3.  Paste the JSON below and save the file.

```json
{
    "Slack Token": "(xox[p|b|o|a]-[0-9]{12}-[0-9]{12}-[0-9]{12}-[a-z0-9]{32})",
    "RSA private key": "-----BEGIN RSA PRIVATE KEY-----",
    "SSH (OPENSSH) private key": "-----BEGIN OPENSSH PRIVATE KEY-----",
    "SSH (DSA) private key": "-----BEGIN DSA PRIVATE KEY-----",
    "SSH (EC) private key": "-----BEGIN EC PRIVATE KEY-----",
    "PGP private key block": "-----BEGIN PGP PRIVATE KEY BLOCK-----",
    "Facebook Oauth": "[f|F][a|A][c|C][e|E][b|B][o|O][o|O][k|K].*['|\"][0-9a-f]{32}['|\"]",
    "Twitter Oauth": "[t|T][w|W][i|I][t|T][t|T][e|E][r|R].*['|\"][0-9a-zA-Z]{35,44}['|\"]",
    "GitHub": "[g|G][i|I][t|T][h|H][u|U][b|B].*['|\"][0-9a-zA-Z]{35,40}['|\"]",
    "Google Oauth": "(\"client_secret\":\"[a-zA-Z0-9-_]{24}\")",
    "AWS API Key": "AKIA[0-9A-Z]{16}",
    "Heroku API Key": "[h|H][e|E][r|R][o|O][k|K][u|U].*[0-9A-F]{8}-[0-9A-F]{4}-[0-9A-F]{4}-[0-9A-F]{4}-[0-9A-F]{12}",
    "Generic Secret": "[s|S][e|E][c|C][r|R][e|E][t|T].*['|\"][0-9a-zA-Z]{32,45}['|\"]",
    "Generic API Key": "[a|A][p|P][i|I][_]?[k|K][e|E][y|Y].*['|\"][0-9a-zA-Z]{32,45}['|\"]",
    "Slack Webhook": "https://hooks.slack.com/services/T[a-zA-Z0-9_]{8}/B[a-zA-Z0-9_]{8}/[a-zA-Z0-9_]{24}",
    "Google (GCP) Service-account": "\"type\": \"service_account\"",
    "Twilio API Key": "SK[a-z0-9]{32}",
    "Password in URL": "[a-zA-Z]{3,10}://[^/\\s:@]{3,20}:[^/\\s:@]{3,20}@.{1,100}[\"'\\s]"
}
```

## Pipeline Architecture

Below is the current architecture of your pipeline.

![Architecture](./images/02-arch.png "Pipeline Architecture")

After you have successfully configured the secrets scanning stage, you can proceed to the next module.