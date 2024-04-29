
# Explicació del Jenkinsfile
Aquest és un Jenkinsfile que utilitza OWASP ZAP per conduir un escaneig de seguretat en una aplicació web.

```groovy
properties([ 
    parameters([ 
        string ( name: 'WEB_URL', defaultValue: 'http://frontend.conference-app', description: 'URL de la web a scanear' ),
        choice ( name: 'test_type', choices: ["baseline", "full-scan", "api-scan"], description: 'Level of testing to use.' ),
        booleanParam ( name: 'w_error', defaultValue: false, description: 'Mark the checkbox to treat warnings as a error' ) 
    ]) 
]) 
```

Aquesta secció defineix les propietats i paràmetres que es poden configurar per a la construcció, com l'URL de la web a escanejar, el nivell de proves a utilitzar i si volem tractar els advertiments com errors.

```groovy
podTemplate(label: 'owaszap', 
    containers: [ 
        containerTemplate( 
            name: 'owaszap', 
            image: 'ghcr.io/zaproxy/zaproxy:stable', 
            command: 'cat', 
            ttyEnabled: true, 
            resources: [ 
                limits: [ memory: '2Gi', cpu: '2' ], 
                requests: [ memory: '2Gi', cpu: '2' ] 
            ] 
        ) 
    ]) 
```

Aquést bloc defineix el template del pod "owaszap" que utilitzarà la imatge de Docker "zaproxy/zaproxy:stable". També estableix límits i requests per a la memòria i el CPU.

```groovy
node('owaszap') { 
    def error_level = "-I" 
    if (params.w_error){ 
        error_level = "" 
    } 
    def api_type = "" 
    if (params.test_type == "api-scan"){ 
        api_type = "-f openapi" 
    } 
```

Estem fixant el nivell d'error i el tipus d'API segons els paràmetres proporcionats.

```groovy
stage('owaszap run') { 
    container('owaszap') { 
        sh("mkdir -p /zap/wrk && sed -i 's/traditional-html/high-level-report/' /zap/zap-${params.test_type}.py && zap-${params.test_type}.py -t ${params.WEB_URL} ${api_type} ${error_level} -r report.html && mv /zap/wrk/* . ") 
    } 
} 
```

En aquest estadi, el contenidor està executant-se, escanejant la aplicació web amb OWASP ZAP.

```groovy
stage("Publish report") { 
    publishHTML([ 
        reportName: 'OwaszapReport', 
        reportDir: '.', 
        reportFiles: 'report.html', 
        keepAll: true, 
        allowMissing: false, 
        alwaysLinkToLastBuild: true, 
        useWrapperFileDirectly: true 
    ]) 
} 
```

Finalment, l'estadi "Publish report" publica l'informe generat a través del escaneig de seguretat OWASP ZAP.