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
(We use this one!)

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
* Field scope can be confusing

---

### Some Jenkinsfile Functions and Globals

* `stage`: Defines a stage of the build. IE: build, test, push
* `withEnv`: Defines a list of variables to inject into the environment
* `env`: Allow access to environment variables. IE: `env.DEBUG`
* `checkout`: Checkout a repo to the current directory
* `sh`: Execute a shell script or application

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
    /* print(a_script_field)  // No Access! */
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
        binding.setPropery('A_GLOBAL_FIELD', 'the global field value')  // Something like this
        def a_script_field = 'the script field value'
        doWork('the argument value')
        return this
    }
    static void main(String[] args) {
        InvokerHelper.runScript(Example, args)
    }
}
```

---

### Loading Other Scripts

* In some cases you may want to load a script after checkout
* Use the `load` function!
* Takes a path and then compiles and runs the script
* If you want to call functions declared in the script you _must_ "`return this`"
```groovy
def example = load('example.gdsl')
example.doWork('please!')
```

---

### How We Build

* Each module we build contains a groovy file with the required steps
* We run a script that performs a `git diff` against master to determine what modules we should include
* We load the groovy script for each of the included modules
* Run each build function

---

### How We Build
#### Loading the Module Groovy Files

```groovy
def loadModules() {
    def jenkinsfile_paths = sh(script: 'scripts/jenkinsfiles', returnStdout: true).split('\n').findAll({it.trim()})
    if (!jenkinsfile_paths) {
        print("No modules detected")
        return []
    }
    
    print("Detected the following ${jenkinsfile_paths.size()} modules: ${jenkinsfile_paths}")
    return jenkinsfile_paths.collect({load(it)})
}
```

---

### How We Build
#### What a module looks like

```groovy
import groovy.transform.Field
@Field def module = 'a-module'

def build() {
    sh('a_module/scripts/build')
}

def test() {
    sh('a_module/scripts/unit')
}

return this
```

---

### How We Build
#### Running a Build

```groovy
modules = loadModules()
modules.each({
    stage("Build: ${it.module}") {
        it.build()
    }
    stage("Test: ${it.module}") {
        it.test()
    }
})
```

---

# Questions?
