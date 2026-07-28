# 06 Pipelines <!-- omit in toc -->

# 1. Crear el Jenkins file
- [Link](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/)

Dentro del proyecto public-api
## 1.1. Crear el archivo jenkinsfile
```js
pipeline {
    agent any
    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

# 2. Agregar el plugin de GitLab
Reiniciar.

# 3. Conexion con GitLab
Esta conexión permite la comunicación entre aplicaciones (Jenkins y GitLab)

## 3.1. Crear el Access Token
En GitLab:
- Edit Profile > Personal Access Tokens
- Add new - Legacy
- name: demo
- scope: API
- create

En Jenkins:
- Manage Jenkins > System > GitLab
- Connection Name: Demo
- URL: https://gitlab.com
- Credentials: add
  - Scope: Global
  - Kind: GitLab API Token
  - API Token
  - ID: git
- Test Connection: Success

# 4. Crear las credenciales del usuario Jenkins
Esta conexión permite la comunicación entre un Job y GitLab
```sh
sudo su
su jenkins

ssh-keygen -q -N "" -C "jenkins@mail.com"

cat /var/lib/jenkins/.ssh/id_ed25519.pub
```


## 4.1. Cambiar la configuración de seguridad
- manage jenkins > Security: Git Host Key Verification: Accept first connection

Esto permite aceptar la conexión on la llave SSH

## 4.2. Agregar las nuevas llaves a GitLab
- Agregar las llaves a GitLab

# 5. Pipeline
- Create a job
- name: pipeline-test
- type: pipeline
- OK
- GitLab Connection
- Pipeline
  - Definition: Pipeline script from SCM
  - SCM: Git
  - Repository URL: git@gitlab.com:<USERNAME>/public-api.git
  - Credentials: add SSH
    - User with ssh
    - username: jenkins
    - add private key
	- Branch: */main
- Script Path: Jenkinsfile
- Save

## 5.1. Error conocido
> credentials Failed to connect to repository

Ejecutar el comando en la terminal para crear el acceso
```sh
# ejemplo
git ls-remote -h -- git@gitlab.com:<USERNAME>/public-api.git HEAD

```

## 5.2. Build Now
- Status
- Console

## 5.3. Variables
```js
pipeline {
    agent any
    environment {
	    name = 'CARLOS'
    }
    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }

        stage('Var') {
            steps {
                echo "Hello ${name}" // comillas dobles
            }
        }
    }
}
```

## 5.4. Variables de Sistema
```js
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}" // comillas dobles
            }
        }
    }
}
```

## 5.5. Variables dinámicas y condicionales
```js
pipeline {
    agent any
    environment {
        gitBranch = "main"
    }
    stages {
        stage('Hello') {
            steps {
                script {
                    branch = env.gitBranch == 'main' ? 'Production' : 'Development'
                    echo "Current Branch: ${branch}"
                }
            }
        }
    }
}
```

## 5.6. Variables Globales
- Manage Jenkins - System - Global properties - Environment Variables
- Add
- Name: GLOBAL_NAME
- Value: <NOMBRE>
- Save

```js
pipeline {
    agent any
    stages {
        stage('Hello') {
            steps {
              echo "Global Name: ${env.GLOBAL_NAME}"
              echo "Global Name: ${GLOBAL_NAME}"
            }
        }
    }
}
```
- En caso de error revise el log y repare.

## 5.7. Secrets y Credenciales
- Manage Jenkins - Credentials - System - Global credentials
- Add Credentials
- Kind: Secret Text
- Secret; 123456
- ID: secretName
```js
pipeline {
    agent any
    environment {
        name = credentials('secretName')
    }
    stages {
        stage('Hello') {
            steps {
              echo "Name: ${name}"
            }
        }
    }
}
```
> Warning: A secret was passed to "echo" using Groovy String interpolation, which is insecure.

## 5.8. Parámetros
```sh
pipeline {
    agent any
    parameters {
        string(name:'NOMBRE', description:'Nombre del estudiante')
    }

    stages {
        stage('Hello') {
            steps {
              echo "Parametro Nombre: ${params.name}"
              echo "Parametro Nombre: ${name}"
            }
        }
    }
}
```
- Build Now
- Refresh
- Build with Parameters
- Revisar la consola y corregir el error.

## 5.9. Expresiones
```js
pipeline {
    agent any
    environment {
        name = 'CARLOS'
    }
    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'

                // Simula un error en el pipeline
                script {
                    currentBuild.result = 'FAILURE'
                }
            }
        }

        stage('Var') {
            when {
              expression {
                currentBuild.result == 'SUCCESS'
              }
            }
            steps {
                echo "Hello ${name}"
            }
        }

        stage('Error') {
            when {
              expression {
                  currentBuild.result == 'FAILURE'
              }
            }
            steps {
                echo "NOTIFICA el error"
            }
        }
    }
}
```
En caso de error en el stage Hello, se muestra el siguiente mensaje:
> Stage "Var" skipped due to when conditional

## 5.10. Triggers
```js
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                script {
                    def currentDate = new Date()
                    echo "Current Date and Time: ${currentDate}"
                }
            }
        }
    }
}
```
- Configure
- Poll SCM
- H/2 * * * * (cada dos minutos Jenkins revisará GitLab por cambios)
- Save
- Agregar un cambio al pipeline

# 6. Práctica: Pipeline
## 6.1. Crear un nuevo Jenkins File en el proyecto con le nombre practica.jenkins
## 6.2. Agregar las variables
```
developer: <ESTUDIANTE>
app: practica
```
## 6.3. Stage: 'Uno', Imprimir en pantalla los textos (usar echo):
```
Nombre del desarrollador:
Nombre de la aplicación:
```
## 6.4. Crear una credencial tipo secret
```
password: abcde
```
## 6.5. Stage 'dos', Imprimir en pantalla el secret
## 6.6. Stage 'tres', crear dos variables dinámicas con los numeros 8 y 5, sumarlas e imprimir el
```
El resultado de sumar <NUMERO 1> + <NUMERO 2> es: <RESULTADO>
```
## 6.7. Crear un nuevo Job (Item) de tipo Pipeline y conectarlo con el Jenkins file creado en el paso 6.1

# 7. Multi-branch
## 7.1. (Opcional) Eliminar los pipelines anteriores
Para evitar ejecuciones por Trigger.

## 7.2. Agregar el brach dev al proyecto GitLab
```sh
git checkout -b dev
git push -u origin dev
```

## 7.3. Crear un nuevo item
- Nombre: multi
- Tipo: Multibranch pipeline
- OK
- Display Name: multi branch
- Branch Sources
  - Add: Git
  - Repository: git@gitlab.com:<USERNAME>/public-api.git
  - Credentials
  - Behaviors: discover branches + discover tags
- Build Configuration
	- by Jenkins File: Jenkinsfile
- Save

>  Resultado: Success
```
Scheduled build for branch: main
Scheduled build for branch: dev
Processed 2 branches
```
## 7.4. Ver el status
Se encuentran dos branches: dev & main

Cada uno funciona como un Pipeline separado

## 7.5. Ejecutar manualmente un build en cada branch

## 7.6. Webhook
- Plugin: Multibranch scan webhook trigger
- Configure
- Scan Multibranch Pipeline Triggers: Scan by webhook
- Trigger Token: < TOKEN > (es un nombre random)
  - Help: muestra la configuración que se debe ingresar en GitLab:
  - EJ: < JENKINS_URL >/multibranch-webhook-trigger/invoke?token=[Trigger token]
- Save

### 7.6.1. Crear el webhook en GitLab
- Public-api > settings - webhook
- Add new
  - URL: < JENKINS_URL >/multibranch-webhook-trigger/invoke?token=< TOKEN >
  - Name: Jenkins
  - Trigger: Push events
  - Enable SSL verification: false
- Add webhook
- Test

## 7.7. Test: Cambiar un valor del jenkinsfile y hacer push a dev
## 7.8. Validar le Pipeline del branch Dev

## 7.9. Variables por branch
Agregar al jenkinsfile el echo:
```js
echo "Branch: $BRANCH_NAME"
```
> la Variable BRANCH_NAME indica el nombre del branch que hizo el webhook


### 7.9.1. Crear stages por branch
```js
pipeline {
    agent any

    stages {
        stage('Build Dev') {
            when {
                branch 'dev'
            }
            steps {
                script {
                    echo "Construyendo el branch DEV"
                }
            }
        }
        stage('Build Prod') {
            when {
                branch 'main'
            }
            steps {
                script {
                    echo "Construyendo el branch PROD"
                }
            }
        }
    }
}
```

## 7.10. Merge dev -> main
- Delete source branch when merge request is accepted: false

### 7.10.1. Validar la ejecución en Jenkins, branch main
- status
- console
```
Stage "Build Dev" skipped due to when conditional
Construyendo el branch PROD
```
# 8. Práctica
## 8.1. Crear un nuevo proyecto en GitLab
- Nombre: practica2
- Crear una nueva carpeta en la VM con el nombre practica2
- Sincronizar el proyecto con GitLab en la VM con el branch
- Crear dos branches: main y stage
- Agregar el jenkinsfile
  - Para stage, mostrar la resta de dos números
  - Para main, mostrar la multiplicación de dos números
- Crear un Multi-branch Job y asignarle el webhook con el nombre del token: practica2
- Probar el webhook de ambos branches
- Checkpoint: el webhook lanza los pipelines de main y stage
