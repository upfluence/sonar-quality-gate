# Sonar quality gate code

Sonarqube have feature quality code gate, but it's not work for Community Edition. So, this plugin will be intergate to CI/CD, get quality code and push report to merge request when has change.

**quality-gate** is a command line interface for quality code gate.
- Analytics code: Use command `sonar-scanner` to analytic code, report and push issues to sonar servers.
- Push issue to code changes of merge request
- Generate report quality code of new code, and create note for merge request.

**For Github and Gitlab**

Result:

![Gitlab Quality Gate](https://raw.githubusercontent.com/dieuhd/sonar-quality-gate/master/images/gitlab_quality_gate.png)


## Getting Started
```bash
$ npm install -g sonar-quality-gate
# Show help
$ quality-gate --help
```

Result:
```bash
   __ _   _   _    __ _  | | (_) | |_   _   _            __ _    __ _  | |_    ___
  / _` | | | | |  / _` | | | | | | __| | | | |  _____   / _` |  / _` | | __|  / _ \
 | (_| | | |_| | | (_| | | | | | | |_  | |_| | |_____| | (_| | | (_| | | |_  |  __/
  \__, |  \__,_|  \__,_| |_| |_|  \__|  \__, |          \__, |  \__,_|  \__|  \___|
     |_|                                |___/           |___/
Usage: quality-gate [options]

Global Options:
  -h, --help                                                                                                   [boolean]
  -D, --define   Define sonar property

                 Authentication:
                 sonar.login The authentication token or login of a SonarQube user with Execute Analysis permission on
                 the project.
                 More parameters:
                 - https://docs.sonarqube.org/latest/analysis/analysis-parameters/                               [array]
      --git      Config git
                 --git.url Git server URL. Default: $GIT_URL
                 --git.token Git token. Default: $GIT_TOKEN
                 --git.project_id Gitlab project ID or Github repository. Default: $CI_PROJECt_ID or $GITHUB_REPOSITORY
                 --git.merge_id Git merge request IID. Default: $CI_MERGE_REQUEST_IID
                                                                                                           [default: {}]
      --sonar    Config sonar
                 --sonar.url Sonarqube server URL. Default: $SONAR_HOST_URL or sonar.host.url in file
                 sonar-project.properties.
                 --sonar.token The authentication token of a SonarQube user with Execute Analysis permission on the
                 project. Default: $SONAR_TOKEN
                 --sonar.project_key Sonar project key. Default: sonar.projectKey in file sonar-project.properties
                                                                                                           [default: {}]
  -v, --version  Show version                                                                                  [boolean]
  -X, --debug    Produce execution debug output                                               [boolean] [default: false]
  -p, --provide                                                                                      [default: "gitlab"]
```

To run check quality code gate:
```bash
quality-gate -p=github -D sonar.login="<token>" --sonar.url="<sonar url>" --sonar.token="<sonar token>" --sonar.project_key="<sonar token>" --git.url="https://gitlab.com" --git.token="xxx" --git.project_id=123 --git.merge_id=345
```


if set env for bellow parameters:
```bash
GIT_URL=""
GIT_TOKEN=""
CI_PROJECt_ID=""
CI_MERGE_REQUEST_IID=""

SONAR_HOST_URL=""
SONAR_TOKEN=""
```
and has file `sonar-project.properties`:
```
sonar.host.url=
sonar.projectKey=
````
We can use short command:
```bash
quality-gate -Dsonar.login=""
```

## Config CI/CD

### Add sonar-project.properties
Add new file `sonar-project.properties` as below content:

```
# sonar.organization=dieuhd # if use sonarcloud, uncomment this line
sonar.host.url=[SONAR_HOST]
sonar.projectKey=[SONAR_PROJECT_KEY]
sonar.qualitygate.wait=true
```
Ref: [sonar-project.properties](./sonar-project.properties)

### Run with Gitlab-CI
Use `quality-gate` instead of `sonar-scanner`.

Example:

``` bash
quality-gate -Dsonar.login=$SONAR_KEY
```
And config for gitlab-ci:

```yaml
stages:
  - CheckSonar

.CheckSonarqube: &CheckSonarqube |
  quality-gate -Dsonar.login=$SONAR_KEY

Sonar:
  stage: CheckSonar
  image: dieuhd/sonar-quality-gate
  rules:
    - if: '$CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"'
  script:
    - *CheckSonarqube
```

**P/S: Only work for merge request. Becase, the plugin need Merge Request IID.**

### [Run with Github Action](https://github.com/marketplace/actions/sonar-quality-gate)

Example:
```yaml
name: Check sonarqube
on: [pull_request]
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2.3.4
        with:
          fetch-depth: 0  # Shallow clones should be disabled for a better relevancy of analysis
      - name: Set up Sonar Quality Gate
        uses: dieuhd/sonar-quality-gate@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} 
          GIT_URL: "https://api.github.com"
          GIT_TOKEN: ${{ secrets.GIT_TOKEN }} 
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_PROJECT_KEY: ${{ secrets.SONAR_PROJECT_KEY }}
        with:
          login:  ${{ secrets.SONAR_TOKEN }}
          url: ${{ secrets.SONAR_HOST_URL }}
          projectKey: ${{ secrets.SONAR_PROJECT_KEY }}
```

## Contribute

```bash
$ git clone https://github.com/dieuhd/sonar-quality-gate.git
$ cd sonar-quality-gate
$ npm install
$ husky install && chmod ug+x .husky/*
$ npm run start:dev
```

## License
MIT. See LICENSE.txt.
