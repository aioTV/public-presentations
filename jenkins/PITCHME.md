---

# Jenkinsfiles Explained

---

* A Jenkinsfile defines what steps Jenkins should take when building your project
* Two different ways to write a Jenkinsfile

---

### Declarative Style

```
pipeline {
    agent none 
    stages {
        stage('Example Build') {
            agent { docker 'maven:3-alpine' } 
            steps {
                echo 'Hello, Maven'
                sh 'mvn --version'
            }
        }
        stage('Example Test') {
            agent { docker 'openjdk:8-jre' } 
            steps {
                echo 'Hello, JDK'
                sh 'java -version'
            }
        }
    }
}
```

---

### Scripted Pipeline
#### We use this one!

```groovy
node {
    stage('Back-end') {
        docker.image('maven:3-alpine').inside {
            sh 'mvn --version'
        }
    }

    stage('Front-end') {
        docker.image('node:7-alpine').inside {
            sh 'node --version'
        }
    }
}
```
