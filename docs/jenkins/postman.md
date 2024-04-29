# Explicació del jenkinsfile
Aquest és un fitxer Jenkinsfile que s'usa per conduir tests utilitzant Postman i Newman en Jenkins.

```groovy
properties([ 
    parameters([ 
        string(name: 'REPO_URL', 
        defaultValue: 'http://gitea.gitea/devsecops/backend-test-java.git', 
        description: 'URL del repositori de codi font'), 
        string(name: 'APP_NAME', 
        defaultValue: 'template', 
        description: 'Nom de la aplicació') 
    ]) 
]) 
```
Aquesta secció defineix les propietats i els paràmetres que es poden configurar per a la construcció, com l'URL del repositori de codi font i el nom de l'aplicació.

```groovy
podTemplate(label: 'mypodpostman', 
    containers: [ 
        containerTemplate( 
        name: "postman", 
        image: "public.ecr.aws/t1l0g7t9/huge-worker:latest", 
        command: "cat", 
        ttyEnabled: true 
        ) 
    ]) 
```
Definim un template de pod 'mypodpostman' que conté un contenidor 'postman' basat en la imatge 'public.ecr.aws/t1l0g7t9/huge-worker:latest'.

```groovy
node("mypodpostman") { 
    def build_status = 'SUCCESS' 
...
}
```

Aquest bloc inclou el codi que s'executa en el node 'mypodpostman'. La variable 'build_status' s'inicialitza amb 'SUCCESS'.

```groovy
stage("Checkout repo") { 
    checkout([ $class: "GitSCM", 
    branches: [[name: "*/main"]], 
    doGenerateSubmoduleConfigurations: false, 
    extensions: [], 
    submoduleCfg: [], 
    userRemoteConfigs: [[url: "${params.REPO_URL}"]] 
    ]) 
}
```

En l'estadi "Checkout repo", es fa checkout del codi font del repositori de Git aconseguit amb la URL proporcionada.

```groovy
try { 
    stage("Newman run") { 
        container("postman") { 
            sh("newman run ${params.APP_NAME}-backend-postman.json -e environment.json -r cli,htmlextra --reporter-htmlextra-export newman/report.html") 
        } 
    } 
} catch (Exception e) { 
    echo "Newman failed with error: ${e}" 
    build_status = 'FAILURE' 
}
```

En cas de error durant l'execució de Newman, el procés captura l'exception i marca l'estat de construcció com a "FAILURE".

```groovy
stage("Publish report") { 
    publishHTML([ 
    reportName: 'PostmanReport', 
    reportDir: 'newman', 
    reportFiles: 'report.html', 
    keepAll: true, 
    allowMissing: false, 
    alwaysLinkToLastBuild: true, 
    useWrapperFileDirectly: true 
    ]) 
}
```

L'estadi "Publish report" publica l'informe generat després de l'execució de Newman.

```groovy
currentBuild.result = build_status 
```

Finalment, assigna l'estat del procés de construcció a la variable "build_status".