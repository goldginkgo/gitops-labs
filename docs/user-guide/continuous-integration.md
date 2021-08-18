# Continuous Integration

## Jenkins Jobs

Use [Job DSL](https://plugins.jenkins.io/job-dsl/) to define Jenkins jobs in a programmatic way.
Refer to [DSL API reference](https://jenkins.gitops.local/plugin/job-dsl/api-viewer/index.html).
Jobs are defined in `gitops-gitops/jenkins-jobs`.

### Example

```groovy
pipelineJob('gitops/docker-jenkins') {
  definition {
    cpsScm {
      scm {
        git {
          remote {
            url('https://github.com/goldginkgo/jenkins-docker.git')
          }
          branch('*/master')
        }
      }
      lightweight()
    }
  }

  triggers {
    gitlabPush {
        buildOnMergeRequestEvents(false)
        buildOnPushEvents(true)
        enableCiSkip(false)
        setBuildDescription(false)
        rebuildOpenMergeRequest('never')
    }
  }
}
```

## Jenkinsfile

### Agents

Currently we recommend to use the `maven38` agent. It has severl containers: kaniko, maven.

### Build Images

We should use [kaniko](https://github.com/GoogleContainerTools/kaniko) to build container images from Dockerfile, inside a container.

### Security Scanning

An example pipeline.

```Groovy
pipeline {
    agent { label "maven38" }

    options {
        // timestamps()    // 日志会有时间
        skipDefaultCheckout()   // 删除隐式checkout scm语句
        disableConcurrentBuilds()   //禁止并行
        timeout(time:10, unit:'MINUTES') //设置流水线超时时间
    }

    // environment {
    //     ACR_REGISTRY_CREDS = credentials('acepi001cr01_username_pass')
    // }

    stages {
        stage('VulnerabilityScan') {
            steps {
                container("trivy"){
                    script{
                        sh '''
                            # export TRIVY_USERNAME=${ACR_REGISTRY_CREDS_USR}
                            # export TRIVY_PASSWORD=${ACR_REGISTRY_CREDS_PSW}
                            trivy client --remote http://trivy.gitops-system:4954 -f json -o trivy-report.json goldginkgo/jenkins:0.1
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            recordIssues(tools: [trivy(pattern: 'trivy-report.json')])
        }
    }
}
```

### Remote Command Execution to VMs

Under development.

### Full Jenkinsfile Example

```Groovy
@Library("jenkins-library")

def tools = new org.devops.v1.tools()
def sendEmail = new org.devops.v1.sendEmail()

pipeline {
    agent { label "maven38"}

    options {
        //timestamps()    // 日志会有时间
        skipDefaultCheckout()   // 删除隐式checkout scm语句
        disableConcurrentBuilds()   //禁止并行
        timeout(time:10, unit:'MINUTES') //设置流水线超时时间
    }

    environment {
        GIT_URL = "560GHD11/gitops-gitops/jenkins-demo.git"
        GIT_BRANCH = "master"
        BUILD_SHELL = "mvn clean package -Dmaven.tet.skip=true"
        IMAGE = "acepi001cr01.azurecr.cn/library/jenkins-demo"
        IMAGE_TAG = "0.1"
        TO_EMAIL_USER = "frank.dai@gitops.cn"

        GITLAB_CREDS = credentials('gitlab_username_pass')
        ACR_REGISTRY_CREDS = credentials('acepi001cr01_username_pass')
    }

    stages {
        stage('GetCode') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: "${GIT_BRANCH}"]],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    submoduleCfg: [],
                    userRemoteConfigs: [[credentialsId: 'gitlab_username_pass', url: "http://${GIT_URL}"]]])
            }
        }

        stage('Build&Test') {
            steps {
                container('maven') {
                    script{
                        tools.PrintMes("编译打包", "blue")
                        sh """
                            ${BUILD_SHELL}
                        """
                    }
                }
            }
        }

        stage('CodeScanner') {
            steps{
                container('maven') {
                    withSonarQubeEnv(installationName: 'SonarQube') {
                        script{
                            tools.PrintMes("静态扫描", "blue")
                            sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                        }

                    }
                }

            }
        }

        stage("QualityGate") {
            steps {
                script{
                    tools.PrintMes("质量门", "blue")
                }
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('BuildImage') {
            steps {
                container("kaniko") {
                    script{
                        tools.PrintMes("创建镜像", "blue")
                        sh "/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --destination=${IMAGE}:${IMAGE_TAG} --cache=true --cache-dir=/kanikocache --skip-tls-verify"
                    }
                }
            }
        }

        stage('VulnerabilityScan') {
            steps {
                container("trivy"){
                    script{
                        tools.PrintMes("漏洞扫描", "blue")
                        sh '''
                            export TRIVY_USERNAME=${ACR_REGISTRY_CREDS_USR}
                            export TRIVY_PASSWORD=${ACR_REGISTRY_CREDS_PSW}
                            trivy client --remote http://trivy.gitops-system:4954 -f json -o trivy-report.json ${IMAGE}:${IMAGE_TAG}
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            when { branch 'master' }
            input {
                message 'Should we continue?'
                ok 'Yes, we should.'
                parameters {
                    string(name: 'input', defaultValue: 'yes', description: 'continue deployment?')
                }
            }

            steps {
                script{
                    tools.PrintMes("是否部署？", "blue")
                    isDeploy = "${input}"

                    if ("${isDeploy}" == 'yes' && "${GIT_BRANCH}" == 'master') {
                        tools.PrintMes("开始部署", "blue")
                        sh """
                            git remote set-url origin http://${GITLAB_CREDS_USR}:${GITLAB_CREDS_PSW}@${GIT_URL}
                            git checkout ${GIT_BRANCH}
                            git config --global user.email "jenkins2@gitops.cn"
                            git config --global user.name "Jenkins Bot"
                            git pull origin master
                            # sed -i 's/{IMAGE}:[0-9]*\\.[0-9]*/{IMAGE}:${IMAGE_TAG}/g' application-demo.yaml
                            # git status
                            # git commit -am 'Bump up image version'
                            # git tag -a ${IMAGE_TAG} -m "version ${IMAGE_TAG}"
                            # git push origin master --tags
                        """
                    } else {
                        tools.PrintMes("不部署", "blue")
                    }
                }
            }
        }
    }

    post {
        success {
            script{
                tools.PrintMes("Success:构建成功", "green")
                currentBuild.description += "\n构建成功!"
                sendEmail.SendEmail("构建成功", TO_EMAIL_USER)
            }
        }
        failure {
            script{
                tools.PrintMes("Failure:构建失败", "red")
                currentBuild.description += "\n构建失败!"
                sendEmail.SendEmail("构建失败", TO_EMAIL_USER)
            }
        }
        aborted {
            script{
                tools.PrintMes("Aborted:构建取消", "red")
                currentBuild.description += "\n构建取消!"
                sendEmail.SendEmail("构建取消", TO_EMAIL_USER)
            }
        }
        always {
            recordIssues(tools: [trivy(pattern: 'trivy-report.json')])
        }
    }
}
```
