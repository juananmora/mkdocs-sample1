# Explicació del jenkinsfile
Aquest és un fitxer Jenkinsfile que s'està utilitzant per executar tests de performance en Jenkins.

```groovy
properties([ 
    parameters([ 
        string(name: 'REPO_URL', 
        defaultValue: 'http://gitea.gitea/devsecops/performance-test', 
        description: 'URL del repositori de codi font') 
    ]) 
]) 
```

Aquesta secció defineix les propietats i els paràmetres que es poden configurar per a la construcció, com per exemple, la URL del repositori de codi font.

```groovy
podTemplate(label: 'mypodjmeter', 
    containers: [ 
        containerTemplate( 
        name: "jmeter", 
        image: "public.ecr.aws/t1l0g7t9/huge-worker:latest", 
        command: "cat", 
        ttyEnabled: true 
        ) 
    ]) 
```
Definim un podTemplate amb una etiqueta 'mypodjmeter' que conté un contenidor 'jmeter' amb la imatge 'public.ecr.aws/t1l0g7t9/huge-worker:latest'.

```groovy
node("mypodjmeter") { 
Indica que els següents estadis s'executaran en el pod 'mypodjmeter'.

stage("Checkout repo") { 
    checkout([ $class: "GitSCM", 
    branches: [[name: "*/master"]], 
    doGenerateSubmoduleConfigurations: false, 
    extensions: [], 
    submoduleCfg: [], 
    userRemoteConfigs: [[url: "${params.REPO_URL}"]] 
    ]) 
}
```

L'estadi "Checkout repo" fa una verificació del codi font dins del repositori de Git que correspon a l'URL proporcionada.

```groovy
stage("JMeter run perf test") { 
    container("jmeter") { 
        sh("jmeter -n -t TestPlanCTTI.jmx -e -o html.d -l log.jtl -q user.properties") 
    } 
}
```

En l'estadi "JMeter run perf test", s'executa un script shell que executa la prova de rendiment de JMeter.

```groovy
stage("Publish report") { 
    publishHTML([ 
    reportName: 'JMeterReport', 
    reportDir: 'html.d', 
    reportFiles: 'index.html', 
    keepAll: true, 
    allowMissing: false, 
    alwaysLinkToLastBuild: true, 
    useWrapperFileDirectly: true 
    ]) 
}
```	

L'estadi "Publish report" publica l'informe generat a través de la prova de rendiment de JMeter.

```groovy
stage("Check perf thresholds") { 
    perfReport compareBuildPrevious: true, 
    errorFailedThreshold: 80, 
    errorUnstableThreshold: 100, 
    filterRegex: "", 
    relativeFailedThresholdNegative: -10.0, 
    relativeFailedThresholdPositive: -10.0, 
    relativeUnstableThresholdNegative: -10.0, 
    relativeUnstableThresholdPositive: -10.0, 
    sourceDataFiles: "log.jtl" 
}
```	
Finalment, l'estadi "Check perf thresholds" comprova si el rendiment de la prova compleix els llindars establerts.