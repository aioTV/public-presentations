---

## Jenkinsfiles Explained

---

* A Jenkinsfile defines the steps Jenkins should take when building your project
* Two different ways to write a Jenkinsfile
  * Declarative
  * Scripted

---

#### Declarative Pipeline

```
pipeline {
    agent { 
        label 'docker'
    }
    environment { 
        DEBUG = 'true'
    }
    stages {
        stage('Test') {
            steps {
                checkout scm
                sh 'scripts/test'
            }
        }
        post {
            always {
                slackSend color: '#00FF00', message: 'Done Test'
            }
        }
    }
}
```

---

### Declarative Pipeline

* Custom syntax developed for Jenkins builds
* Can be easier to write and understand for simple build tasks

---

#### Scripted Pipeline
##### We use this one!

```groovy
node('docker') {
    stage('Test') {
        checkout(scm)
        try {
            withEnv(["DEBUG=true"]) {
                sh('scripts/test')
            }
        } finally {
            slackSend(color: '#00FF00', message: 'Done Test')
        }
    }
}
```

---

### Scripted Pipeline

* Written in Groovy using GDSL (Groovy Domain Specific Language)
* We typically write Groovy scripts which are then compiled into classes
* Extremely powerful

---

### Some Jenkinsfile Language

* stage: Defines a stage of the build. IE: build, test, push
* withEnv: Defines a list of variables to inject into the environment
* env: Allow access to environment variables. IE: `env.DEBUG`
* checkout: Checkout a repo to the current directory
* sh: Execute a shell script or application

--- 

### What happens when a groovy script is compiled?

```groovy
import groovy.transform.Field
@Field def a_class_field = 'the class field value'
A_GLOBAL_FIELD = 'the global field value'
def a_script_field = 'the script field value'

doWork('the argument value')

def doWork(an_argument) {
    print(a_class_field)
    print(A_GLOBAL_FIELD)
    # print(a_script_field) # No Access!
    print(an_argument)
}

return this
```

---

### What happens when a groovy script is compiled?

```groovy
import org.codehaus.groovy.runtime.InvokerHelper
class Example extends Script {
    a_class_field = 'the class field value'
    def doWork(an_argument) {
        print(a_class_field)
        print(binding.getProperty('A_GLOBAL_FIELD')
        # print(a_script_field) # No Access!
        print(an_argument)
    }
    def run() {
        binding.setPropery('A_GLOBAL_FIELD', 'the global field value')  # Something like this
        def a_script_field = 'the script field value'
        doWork('the argument value')
        return this
    }
    static void main(String[] args) {
        InvokerHelper.runScript(Example, args)
    }
}
```
