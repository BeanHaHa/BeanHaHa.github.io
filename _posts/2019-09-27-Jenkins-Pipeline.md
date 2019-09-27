---
layout: post
title: Jenkins Pipeline实例
date: 2019-09-27
tags: [Pipeline]
---

##Jenkins 多分支流水线

Jenkins 流水线是一套插件，它支持实现和集成持续交付流水线到 Jenkins。流水线提供了一组可扩展的工具，用于通过流水线 DSL 将简单到复杂的交付流水线建模为“代码”。

####1、什么是 pipeline
pipeline 是 jenkins 的一套插件，用于定义 CD 流程，弹性，可管理。

####2、什么是 Jenkinsfile
通过代码的方式来管理 pipeline
这样一来，可以通过版本控制系统来管理。


####3、Jenkinsfile 的内容示例
```json
#!/usr/bin/env groovy
pipeline {
    agent any //在任何可用的代理上，执行流水线或它的任何阶段

    options {
        timeout(time: 30, unit: 'MINUTES') // 设置超时时间 30分钟
        timestamps()
    }

    tools {
        maven 'mavne_3_3_1'
        jdk 'jdk1.8'
        gradle 'Gradle 4.10.2'
    }

    stages {
        // 定义build阶段
        stage('Build') {
            steps {
                script {
                    echo "current branch: $BRANCH_NAME"
                    try {
                        // 区别不同分支，此处是gradle项目
                        // 若是maven项目，则执行'mvn -Ponline_env clean install -Dmaven.test.skip=true'
                        if (BRANCH_NAME.startsWith("master")) {
                            sh 'gradle clean build -P env=prod  -x test  --stacktrace'
                        } else {
                            sh 'gradle clean build -P env=dev  -x test  --stacktrace'
                        }
                    } catch (err) {
                        throw err
                    }
                }
            }
        }
        // 使用Jacoco检测代码覆盖率
        stage('Jacoco Report') {
            steps {
                script {
                    echo 'Jacoco Running..'
                    try {
                        sh 'gradle jacocoTestReport'
                    } catch (err) {
                        throw err
                    }
                }
            }
        }
        // 上传Jacoco检测结果
        stage('JacocoPublisher') {
            steps {
                jacoco()
            }
        }
        // 使用SonarQube 分析代码
        stage('SonarQube analysis') {
            steps {
                script {
                    try {
                        def scannerHome = tool 'sonar1';
                        withSonarQubeEnv('sonar1') {
                            sh "${scannerHome}/bin/sonar-scanner " +
                                    "-Dsonar.projectKey=insurance-tech-admin-backend " +
                                    "-Dsonar.projectName=insurance-tech-admin-backend " +
                                    "-Dsonar.sources=./src/main/java " +
                                    "-Dsonar.tests=./src/test/java " +
                                    "-Dsonar.language=java " +
                                    "-Dsonar.java.coveragePlugin=jacoco " +
                                    "-Dsonar.jacoco.reportPath=./build/jacoco/test.exec " +
                                    "-Dsonar.java.source=1.8 " +
                                    "-Dsonar.java.binaries=./build/classes " +
                                    "-Dsonar.sourceEncoding=UTF-8 " +
                                    "-Dsonar.branch.name=${BRANCH_NAME}"
                        }
                    } catch (err) {
                        throw err
                    }
                }
            }
        }
    }
    post {
        failure {
            emailext(
                    subject: "Jenkins build is ${currentBuild.result}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    mimeType: "text/html",
                    body: """<p>Jenkins build is ${currentBuild.result}: ${env.JOB_NAME} #${env.BUILD_NUMBER}:</p>
                         <p>Check console output at <a href="${env.BUILD_URL}console">${env.JOB_NAME} #${
                        env.BUILD_NUMBER
                    }</a></p>""",
                    recipientProviders: [[$class: 'CulpritsRecipientProvider'],
                                         [$class: 'DevelopersRecipientProvider'],
                                         [$class: 'RequesterRecipientProvider']]
            )
        }
    }
}

```

####4、Gradle项目 build.gradle添加Jacoco配置
```gradle
apply plugin: 'jacoco'

jacoco {
    toolVersion = "0.8.3"
    reportsDir = file("${buildDir}/reports")
}

jacocoTestReport {
    reports {
        xml.enabled false
        csv.enabled false
        html.enabled true
        html.destination file("${reportsDir}/jacocoHtml")
    }
    dependsOn test
}

```

## 创建多分支流水项目
![pipeline1](https://raw.githubusercontent.com/BeanHaHa/BeanHaHa.github.io/master/assets/images/2019/pipeline1.png)

![pipeline2](https://raw.githubusercontent.com/BeanHaHa/BeanHaHa.github.io/master/assets/images/2019/pipeline2.png)

![pipeline3](https://raw.githubusercontent.com/BeanHaHa/BeanHaHa.github.io/master/assets/images/2019/pipeline3.png)

## gitlab-ci.yml添加配置
```yml
trigger_build_develop:
  stage: deploy
  only:
    - develop
  tags:
    - curl
  script:
    - curl http://192.168.180.192/jenkins/git/notifyCommit?url=git@git.winbaoxian.com:wy-serverside/insurance-tech-admin-backend.git&branches=develop
trigger_build_master:
  stage: deploy
  only:
    - master
  tags:
    - curl
  script:
    - curl http://192.168.180.192/jenkins/git/notifyCommit?url=git@git.winbaoxian.com:wy-serverside/insurance-tech-admin-backend.git&branches=master
```


## 代码覆盖率的意义
1. 分析未覆盖部分的代码，从而反推在前期测试设计是否充分，没有覆盖到的代码是否是测试设计的盲点，为什么没有考虑到？需求/设计不够清晰，测试设计的理解有误，工程方法应用后的造成的策略性放弃等等，之后进行补充测试用例设计。
2. 检测出程序中的废代码，可以逆向反推在代码设计中思维混乱点，提醒设计/开发人员理清代码逻辑关系，提升代码质量。
3. 代码覆盖率高不能说明代码质量高，但是反过来看，代码覆盖率低，代码质量不会高到哪里去，可以作为测试自我审视的重要工具之一。

##代码覆盖率工具比较
> 目前Java常用覆盖率工具Jacoco、Emma和Cobertura

![pipeline4](https://raw.githubusercontent.com/BeanHaHa/BeanHaHa.github.io/master/assets/images/2019/pipeline4.png)

![pipeline5](https://raw.githubusercontent.com/BeanHaHa/BeanHaHa.github.io/master/assets/images/2019/pipeline5.png)


##自动化集成流程（理论上）
1. 业务开发完成之后，开发人员做单元测试，单元测试完成之后，保证单元测试全部通过同时单元测试代码覆盖率达到一定程度（这个需要开发和测试约定，理论上越高越好），开发提测。
2. 测试人员根据测试用例进行测试（包括手工测试和自动化测试），结合git获取本次变动代码的覆盖率信息。行覆盖率需达到100%，分支达到50%以上，这个需要具体场景具体分析。
3. 测试通过之后，代码合并至主干，进行自动化回归。
4. 回归测试通过之后，代码可以上线。