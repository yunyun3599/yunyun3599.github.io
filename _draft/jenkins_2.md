# Jenkins2
작업된 코드가 deploy 되기까지의 Jenkins를 이용해 CI/CD가 이루어지는 과정은 다음과 같습니다.  
1. 코드 작업 내용을 github에 커밋한다.
2. Jenkins의 CI job을 돌린다. 
3. Jenkins에서 Github의 코드를 Pull 받아와 gradle을 이용해 빌드한다. 
4. Jib을 이용해 빌드된 코드를 포함한 이미지를 생성한다.
5. 생성된 이미지를 dockerhub에 push한다.
6. Jenkins에서 CD Job을 수행한다.
7. 배포 서버에서 dockehub에 push된 이미지를 pull 받아 온다.
8. 받아온 이미지를 통해 container를 구동해 web application을 띄운다.

따라서 앞으로 배포 서버와 docker hub에 대한 credential 설정을 추가로 진행하겠습니다.  

## 도커 허브 credential 설정
앞서 github deploy key를 설정했던 것과 동일하게 Jenkins 관리 > Manage credentials > System > Global credentials로 들어가 dockerhub 인증 정보도 등록을 해줍니다.  
![](/assets/img/2022-12/2022-12-21-jenkins2/dockerhub-credential.png)

## 배포 서버 credential 설정
배포 서버에도 ssh key를 생성해주어야 합니다.  
아래 명령어를 차례대로 수행해 ssh 키를 생성해 줍니다.  
```sh
$ sudo apt install -y openssh-server
$ ssh-keygen -t ed25519 -a 100 -f deploy-key
```
key 생성이 완료될 때까지 엔터를 눌러줍니다.  
결과로 deploy-key와 deploy-key.pub 파일이 생성될 것입니다.  

키가 다 생성 되었으면 Jenkins 관리 > Manage credentials > System > Global credentials로 다시 들어가 키를 등록해줍니다.  

이 때 enter directly를 체크한 후 Add를 눌러 앞에서 생성한 id_rsa 파일의 내용을 전부 붙여넣기 해 줍니다.  
![](/assets/img/2022-12/2022-12-21-jenkins2/add_deploy_key.png)

그리고 배포 서버의 ~/.ssh 디렉터리로 가 deploy-key.pub의 내용을 authorized_keys 파일에 붙여넣기 해 줍니다.  
```sh
$ cd ~/.ssh
$ vi authorized_keys    # deploy-key.pub 내용 붙여넣기
```

## build.gradle 파일 설정 및 Jenkinsfile 설정
gradle을 이용해 spring프로젝트를 생성 후에 push할 예정입니다.  
다음 명령어로 spring boot 프로젝트를 생성합니다.  
```sh
$ gradle init --dsl=groovy --type=java-application --test-framework=junit --package=com.jenkins --project-name=jenkins_docker
```

프로젝트의 내용을 본인이 원하는 대로 구성하시면 됩니다.  
제가 사용한 코드는 [깃허브](https://github.com/yunyun3599/DevOps-Docker_Kubernetes/tree/master/Docker/jenkins_docker)에 올라가 있습니다.  

프로젝트가 생성되면 build.gradle에 들어가 내용을 다음과 같이 작성합니다.  
```gradle
/*
 * This file was generated by the Gradle 'init' task.
 *
 * This generated file contains a sample Java application project to get you started.
 * For more details take a look at the 'Building Java & JVM projects' chapter in the Gradle
 * User Manual available at https://docs.gradle.org/7.4.2/userguide/building_java_projects.html
 */

plugins {
    // Apply the application plugin to add support for building a CLI application in Java.
    id 'java'
    id 'org.springframework.boot' version '2.7.1'
    id 'io.spring.dependency-management' version '1.0.12.RELEASE'
    id 'com.google.cloud.tools.jib' version '3.3.1'
}

repositories {
    // Use Maven Central for resolving dependencies.
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf:2.7.1'
    implementation 'org.springframework.boot:spring-boot-starter-web:2.7.1'
    developmentOnly 'org.springframework.boot:spring-boot-devtools:2.7.1'
}

version = '0.0.1-SNAPSHOT'
description = 'jenkins_docker'  // 프로젝트 설명
group = 'com.jenkins'   // 프로젝트 패키지

java.sourceCompatibility = JavaVersion.VERSION_11

jar {
    enabled = false
}

tasks.withType(JavaCompile) {
    options.encoding = 'UTF-8'
}

jib {
    from {
        image = 'adoptopenjdk/openjdk11:alpine-jre'
    }
    to {
        image = project.findProperty("docker.repository") + "/" + project.findProperty("docker.image.name")
        tags = [project.findProperty("docker.image.tag")]
        auth {
            username = project.findProperty("docker.repository.username")
            password = project.findProperty("docker.repository.password")
        }
    }
    container {
        mainClass = 'com.jenkins.StartApplication'
        jvmFlags = ['-Xms512m', '-Xmx512m', '-Xdebug', '-XshowSettings:vm', '-XX:+UnlockExperimentalVMOptions', '-XX:+UseContainerSupport']
        ports = ['8080']

        environment = [SPRING_OUTPUT_ANSI_ENABLED: "ALWAYS"]
        labels = [version:project.version, name:project.name, group:project.group]

        creationTime = 'USE_CURRENT_TIMESTAMP'
        format = 'Docker'
    }
    extraDirectories {
        paths {
            path {
                from = file('build/libs')
            }
        }
    }
}
```

Jenkinsfile은 다음과 같이 작성합니다.  
```Jenkinsfile
def mainDir="Docker/jenkins_docker"
def dockerRepository="<도커허브id>"
def dockerImageName="jenkins"
def deployHost="베포 서버 ip"

pipeline {
    agent any

    stages {
        stage('Pull Codes from Github') {
            steps{
                checkout scm
            }
        }
        stage('Build Codes by Gradle') {
            steps {
            withCredentials([usernamePassword(credentialsId: 'dockerhub-key',
                            usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                                script {
                                    sh """
                                    cd ${mainDir}
                                    ./gradlew clean build -Pdocker.repository=${dockerRepository} \
                                                          -Pdocker.repository.username=${USERNAME} \
                                                          -Pdocker.repository.password=${PASSWORD} \
                                                          -Pdocker.image.name=${dockerImageName} \
                                                          -Pdocker.image.tag=${currentBuild.number}
                                    """
                                }
                            }
            }
        }
        stage('Build Docker Image by Jib & Push image to Dockerhub') {
            steps {
            withCredentials([usernamePassword(credentialsId: 'dockerhub-key',
                            usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                                script {
                                    sh """
                                        cd ${mainDir}
                                        ./gradlew jib -Pdocker.repository=${dockerRepository} \
                                                      -Pdocker.repository.username=${USERNAME} \
                                                      -Pdocker.repository.password=${PASSWORD} \
                                                      -Pdocker.image.name=${dockerImageName} \
                                                      -Pdocker.image.tag=${currentBuild.number}
                                    """
                                }
                            }
            }
        }
        stage('Deploy') {
            steps {
                sshagent(credentials: ["deploy-key"]){
                    sh "ssh -o StrictHostKeyChecking=no yoonjae@${deployHost} \
                        docker run -d -p 80:8080 -t ${dockerRepository}/${dockerImageName}:${currentBuild.number}"
                }
            }
        }
    }
}
```

## Jenkins pipeline 생성
코드 작업과 인증 설정이 완료되었다면 Jenkins에서 pipeline을 생성해 CI/CD 작업이 돌도록 해주어야 합니다.  
Jenkins 메인 페이지 좌측 메뉴의 가장 위 `+ 새로운 Item` 메뉴를 선택합니다.  
![](/assets/img/2022-12/2022-12-21-jenkins2/make_pipeline.png)
그리고 위의 화면처럼 적당한 이름을 짓고, pipeline을 선택하여 아이템을 생성합니다.  
아이템 생성 후에 나오는 페이지에서는 수동으로 빌드를 진행할 것이기 때문에 위쪽에는 따로 체크할 것이 없고, Pipeline 항목에서 `Pipeline script from SCM`을 선택한 후 SCM 은 Git을 선택해 줍니다.  
그리고 사용중인 깃허브 레포지토리 https 주소를 Repository URL에 작성해 줍니다.  
![](/assets/img/2022-12/2022-12-21-jenkins2/pipeline_detail.png)
참고로 가장 밑의 Script Path는 Jenkins파일을 작성한 github repository상의 파일 경로를 적어주셔야 합니다.  

또한 이 과정이 끝난 후에는 메인 페이지 좌측 탭의 Jenkins 관리 > Configure Global Security 메뉴에 가서 가장 밑의 Git Host Key Verification Configuration 파일 설정을 No verification으로 설정해줍니다.  

## 빌드하기
위의 과정이 모두 완료되었다면 방금 만든 pipeline 페이지의 좌측 탭의 `지금 빌드`를 클릭하여 빌드를 트리거할 수 있습니다.  
