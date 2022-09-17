# Jenkins CICD Advanced Demo Code and Configs

Contains Jenkins code and configuration examples as used within the [Jenkins CI/CD - Advanced](https://cloudacademy.com/course/jenkins-cicd-advanced) course.

## Demo 6: Blue Ocean - Install and Configure

UI instructions only

Refs
https://github.com/jeremycook123/devops-webapp1

## Demo 7: Blue Ocean - Create/Edit a Pipeline

UI instructions only

Refs
https://github.com/jeremycook123/devops-webapp2

Note: see commit [29eaa64](https://github.com/jeremycook123/devops-webapp2/tree/29eaa64fdd4b2a4f06ded4c68dfe0725984e9b03)

```
whoami
date
echo $PATH
pwd
ls -la
./gradlew build --info
```

```
pipeline {
  agent {
    node {
      label 'agent1'
    }

  }
  stages {
    stage('Clone') {
      steps {
        git(url: 'https://github.com/jeremycook123/devops-webapp2', branch: 'master')
      }
    }
    stage('Build') {
      parallel {
        stage('Build') {
          steps {
            sh '''whoami
date
echo $PATH
pwd
ls -la
./gradlew build --info'''
          }
        }
        stage('P1') {
          steps {
            sh '''date
echo run parallel!!'''
          }
        }
        stage('P2') {
          steps {
            sh '''date
echo run parallel!!'''
          }
        }
      }
    }
    stage('Publish') {
      steps {
        archiveArtifacts(artifacts: 'build/libs/*.war', fingerprint: true, onlyIfSuccessful: true)
      }
    }
  }
}
```

## Demo 9: Docker Based Builds - Install and Configure

```
sudo apt-get install -y docker.io
```

```
docker
```

```
docker ps
```

```
sudo -s
docker ps
docker info
cat /etc/group
usermod -aG docker jenkins
exit
```

```
sudo su - jenkins
whoami
docker ps
docker info
exit
```

```
pipeline {
    agent {
        node {
            label 'agent1'
        }
    }
    stages {
        stage('Docker') {
            steps {
                sh "whoami"
                sh "docker info"
                sh "docker ps"
            }
        }
    }
}
```

## Demo 10: Docker Based Builds - Create Pipeline

```
pipeline {
    agent none
    stages {
        stage('Maven') {
            agent {
                docker {
                    image 'maven:3.6.0-jdk-12-alpine'
                    label 'agent1'
                }
            }
            steps {
                echo 'Hello, Maven'
                sh 'mvn --version'
            }
        }
        stage('JDK') {
            agent {
                docker {
                    image 'openjdk:11.0.1-jdk'
                    label 'agent1'
                }
            }
            steps {
                echo 'Hello, JDK'
                sh 'java -version'
            }
        }
    }
}
```

## Demo 11: Blue Ocean - Build Package and Publish Docker Image to Docker Hub

Refs
https://github.com/jeremycook123/devops-webapp2

```
pipeline {
  agent {
    node {
      label 'agent1'
    }

  }
  stages {
    stage('Clone') {
      steps {
        git(url: 'https://github.com/jeremycook123/devops-webapp2', branch: 'master')
      }
    }
    stage('Build') {
      parallel {
        stage('Build') {
          steps {
            sh '''RELEASE=webapp.war
pwd
./gradlew build -PwarName=$RELEASE --info
ls -la build/libs/
cp ./build/libs/$RELEASE ./docker
'''
          }
        }
        stage('P1') {
          steps {
            sh '''date
echo run parallel!!'''
          }
        }
        stage('P2') {
          steps {
            sh '''date
echo run parallel!!'''
          }
        }
      }
    }
    stage('Packaging') {
      steps {
        sh '''pwd
cd ./docker
docker build -t cloudacademydevops/webapp1-2019:$BUILD_ID .
docker tag cloudacademydevops/webapp1-2019:$BUILD_ID cloudacademydevops/webapp1-2019:latest
docker images
'''
      }
    }
    stage('Publish') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: 'ca-dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]){
            sh '''
docker login -u="$DOCKER_USERNAME" -p="$DOCKER_PASSWORD"
docker push cloudacademydevops/webapp1-2019:$BUILD_ID
docker push cloudacademydevops/webapp1-2019:latest
'''
          }
        }

      }
    }
  }
}
```