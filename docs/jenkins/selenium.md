# Pipeline d'Integració Contínua MAT { .md-typeset }

![Jenkins Pipeline](images/jenkins-pipeline.png){ align=right width="300" }

## Descripció General
Aquest pipeline Jenkins automatitza l'execució de proves funcionals integrat al Marc d'Automatització de Testing (MAT) del CTTI.

<div class="grid cards" markdown>

-   :material-git: __Integració amb GitHub__
-   :material-jira: __Sincronització amb JIRA__
-   :material-chart-line: __Mètriques en temps real__
-   :material-shield-check: __Quality Gates Integrats__

</div>

## Diagrama del Flux

```mermaid
%%{init: {'theme':'neutral'}}%%
flowchart TD
A([Inici]) --> B[Validació de Paràmetres]
B --> C[Checkout Codi]
C --> D[Execució de Proves]
D --> E[Pujada de Resultats a JIRA]
E --> F[Publicació d'Informes]
F --> G{Avaluació Quality Gate}
G -->|Èxit| H[Notificació d'Èxit]
G -->|Error| I[Notificació d'Error]
H --> Z([Fi])
I --> Z
```

## Paràmetres del Pipeline

| Paràmetre | Descripció | Valors Permesos |
|-----------|------------|-----------------|
| `REPO_URL` | Repositori de proves | URL GitHub vàlida |
| `ENV_TO_TEST` | Entorn de proves | Desenvolupament, Integració, Preproducció, Producció |
| `BRANCH` | Branca a provar | Nom de branca vàlid |
| `QUALITY_GATE` | Control de qualitat | true/false |

## Etapes Principals

### 1. Configuració Inicial
```mermaid
sequenceDiagram
participant J as Jenkins
participant G com a GitHub
participant K com a Kubernetes
J->>G: Clona repositori
G-->>J: Codi font
J->>K: Desplega Pod Maven
K-->>J: Confirmació
```

### 2. Execució de Proves
??? tip "Tecnologies Utilitzades"
    - Selenium per a proves funcionals
    - Maven com a gestor de dependències
    - Allure per a informes executives


### 3. Gestió de Resultats

| Eina | Funció | Integració |
|------|--------|------------|
| JIRA | Pujada de resultats | Xray Test Management |
| GitHub | Vinculació a PRs | Comentaris automàtics |
| InfluxDB | Emmagatzematge mètriques | Grafana Dashboards |

## Qualitat i Seguretat

!!! danger "Control d'Errors"
    El pipeline inclou mecanismes avançats de gestió d'errors:
    - Validació de tickets JIRA
    - Avaluació de llindars d'error
    - Notificacions multi-canal

```mermaid
stateDiagram-v2
[] --> Error
Error --> Notificació: Envia alerta
Notificació --> JIRA: Crea incidència
Notificació --> Slack: Notifica equip
JIRA --> []
Slack --> [*]
```
```
// Definition of properties for the pipeline, including input parameters
properties([
  parameters([
    string(
      name: 'REPO_URL',
      defaultValue: 'https://github.com/ctti-dev/3632.00-mat-functional-tests',
      description: 'URL where functional tests are hosted'
    ),
    choice(
      name: 'ENV_TO_TEST',
      choices: ['Desenvolupament', 'Integracio', 'Preproduccio', 'Produccio'],
      description: 'Environments to test in'
    ),
    choice(
      name: 'REPO_URL_INFLUXDB',
      choices: getInfluxApplications(), 
      description: 'URL where functional tests are hosted'
    ),
    string(
      name: 'BRANCH',
      defaultValue: 'master',
      description: 'Branch of Test Repository'
    ),
    string(
      name: 'URL_APP',
      defaultValue: 'https://qualitat.solucions.gencat.cat',
      description: 'Application URL'
    ),
    string(
      name: 'UMBRAL',
      defaultValue: '20',
      description: 'Test Failed Threshold'
    ),
    booleanParam(
      name: 'QUALITY_GATE',
      defaultValue: true,
      description: 'Enable/Disable Quality Gate'
    ),
    string(
      name: 'JIRA_PROJECT_KEY',
      defaultValue: '',
      description: 'Jira Project Key'
    ),
    string(
      name: 'JIRA_ISSUE_KEY',
      defaultValue: '',
      description: 'Jira `TEST PLAN` Issue Key'
    ),
    string(
      name: 'GITHUB_PULL_REQUEST_NUMBER',
      defaultValue: '',
      description: 'Pull Request GitHub'
    )
  ])
])


def getInfluxApplications() {
    def influxdb_url = "${env.MAT_INFLUXDB_URL}/api/v2/query?org=primary"
    def token

    withCredentials([string(credentialsId: 'influxdb-token', variable: 'INFLUXDB_TOKEN')]) {
        token = INFLUXDB_TOKEN

        // Consulta Flux2 para obtener datos de InfluxDB
    def query = """
from(bucket: "jenkins")
  |> range(start: -30d) 
  |> filter(fn: (r) => r["_measurement"] == "git_data")
  |> filter(fn: (r) => r["_field"] == "git_repository")
  |> filter(fn: (r) => r["_value"] =~ /functional/)  
  |> group()
  |> distinct(column: "_value") 
  |> keep(columns: ["_value"]) 
  |> yield(name: "last")
"""

        // Realizar solicitud HTTP para obtener los datos
        def response = httpRequest(
            url: influxdb_url,
            httpMode: 'POST',
            contentType: 'APPLICATION_JSON',
            requestBody: """{"query": "${query.replaceAll('\n', ' ').replaceAll('\"', '\\\\\"')}", "type": "flux"}""",
            customHeaders: [[name: 'Authorization', value: "Token ${token}"]]
        )

        // Imprimir el contenido de la respuesta
        echo "Response Content: ${response.content}"

        // Procesar la respuesta como texto (CSV)
        def lines = response.content.split('\n')
        def values = []

        // Comenzar desde la línea 1 para evitar la cabecera
        for (int i = 1; i < lines.length; i++) {
            def columns = lines[i].split(',')
            if (columns.size() > 3) {
                values.add(columns[3].trim()) // Agrega el valor de la columna "application"
            }
        }

        // Imprimir los valores extraídos
        echo "Valores extraídos: ${values}"

        // Devolver los valores como lista para el parámetro choice
        return values
    }
}

// Function that executes a JIRA command only if project and issue keys are provided
def run_jira_cmd(String cmd_name, Closure callback) {
  if (!params.JIRA_PROJECT_KEY.isEmpty() && !params.JIRA_ISSUE_KEY.isEmpty()) {
    callback()
  }
  else echo("[WARNING]: ${cmd_name} :: skipped due to missing parameters")
}

// Function to add a comment to a JIRA issue
def jira_add_comment(String msg) {
  run_jira_cmd("jira_add_comment", {
    withEnv(['JIRA_SITE=JIRA-CTTI']) {
      jiraAddComment(idOrKey: "${params.JIRA_ISSUE_KEY}", input: [body: msg])
    }
  })
}

// Function to upload test results to JIRA
def jira_upload_results() {
  run_jira_cmd("jira_upload_results", {
    step([
      $class: 'XrayImportBuilder',
      serverInstance: '226faf03-9189-4ee8-964e-1a691d60f62d',
      projectKey: "${params.JIRA_PROJECT_KEY}",
      testPlanKey: "${params.JIRA_ISSUE_KEY}",
      testEnvironments: "${params.ENV_TO_TEST}",
      endpointName: '/testng',
      importFilePath: '**/surefire-reports/testng-results.xml'
    ])
  })
}

// Function to upload compressed reports to JIRA
def jira_upload_report() {
  try {
    run_jira_cmd("jira_upload_report", {
      zip(zipFile: "${BUILD_NUMBER}-prova_funcional.zip", archive: false, dir: "target/report/")
      withEnv(['JIRA_SITE=JIRA-CTTI']) {
        jiraUploadAttachment(idOrKey: "${params.JIRA_ISSUE_KEY}", file: "${BUILD_NUMBER}-prova_funcional.zip")
      }
    })
  } catch (Exception e) {
    jira_add_comment("[ERROR] (#${BUILD_NUMBER}): ${e.message}")
  }
}

// Function to limit a value between a minimum and a maximum
def clamp = { value, min, max ->
  return ((value <= min) ? min : (value >= max) ? max : value)
}

// Function to validate Jira issue
def validate_jira_issue() {
  // Verificar que los parámetros necesarios están presentes
  if (!params?.JIRA_PROJECT_KEY?.trim() || !params?.JIRA_ISSUE_KEY?.trim()) {
    echo("[ERROR]: Missing parameters. Please ensure both JIRA_PROJECT_KEY and JIRA_ISSUE_KEY are provided.")
    return
  }

  withCredentials([usernamePassword(credentialsId: 'jira-step', usernameVariable: 'JIRA_USER', passwordVariable: 'JIRA_API_TOKEN')]) {
    try {
      def response = httpRequest(
        url: "${env.MAT_JIRA_URL}/rest/api/3/issue/${params.JIRA_ISSUE_KEY}",
        httpMode: 'GET',
        acceptType: 'APPLICATION_JSON',
        authentication: 'jira-step'
      )

      if (response.status != 200) {
        error("[ERROR] (#${env.BUILD_NUMBER}): Failed to retrieve Jira issue: HTTP ${response.status}")
      }

      def issue = readJSON text: response.content

      // Validar que el issue pertenece al proyecto esperado
      if (issue.fields.project.key != params.JIRA_PROJECT_KEY) {
        error("[ERROR] (#${env.BUILD_NUMBER}): The issue does not belong to the project ${params.JIRA_PROJECT_KEY}")
      }

      // Validar que el tipo de issue es el esperado
      if (issue.fields.issuetype.name != 'Test Plan') {
        error("[ERROR] (#${env.BUILD_NUMBER}): The issue type is not a Test Plan")
      }

      echo "[SUCCESS] (#${env.BUILD_NUMBER}): Issue ${params.JIRA_ISSUE_KEY} validation passed."
    } catch (hudson.AbortException e) {
      if (e.getMessage().contains("404")) {
        error("[ERROR] (#${env.BUILD_NUMBER}): Issue ${params.JIRA_ISSUE_KEY} not found.")
      } else if (e.getMessage().contains("403")) {
        error("[ERROR] (#${env.BUILD_NUMBER}): Forbidden. Check your API token permissions.")
      } else {
        error("[ERROR] (#${env.BUILD_NUMBER}): Failed to validate Jira issue: ${e.getMessage()}")
      }
    } catch (Exception e) {
      error("[ERROR] (#${env.BUILD_NUMBER}): Unexpected error: ${e.getMessage()}")
    }
  }
}

// Add a comment to JIRA at the beginning of the pipeline
// Check that required parameters are not empty
jira_add_comment("EXECUCIÓ INICIADA (#${BUILD_NUMBER})")
['REPO_URL', 'BRANCH', 'URL_APP'].each{p ->
  if (!params."$p".trim()) {
    jira_add_comment("[ERROR] (#${BUILD_NUMBER}): Missing required parameter `$p`")
    error("Missing required parameter `$p`")
  }
}

// Stage to validate Jira Issue
validate_jira_issue()

timestamps {
// Define a pod with a Maven container to run tests
try {
  podTemplate(
    label: 'selenium',
    containers: [
      containerTemplate(
        name: 'maven',
        image: 'matproacrwe1001.azurecr.io/maven-python:latest',
        command: 'cat',
        ttyEnabled: true,
        resourceRequestCpu: '1',    // CPU mínima requerida
        resourceRequestMemory: '1Gi', // Memoria mínima requerida
        resourceLimitCpu: '2',     // CPU máxima permitida
        resourceLimitMemory: '4Gi' // Memoria máxima permitida
      )
    ],
    volumes: [
      persistentVolumeClaim(
        claimName: 'maven-dependencies-pvc', // Nombre del PVC creado en Kubernetes
        mountPath: '/root/.m2/repository'   // Directorio donde Maven guarda las dependencias
      )
  ],
    nodeSelector: 'kubernetes.azure.com/agentpool=secondary'
  )


  {
    node('selenium') {
      stage('checkout-repo') {
        try {
          checkout([
              $class: 'GitSCM',
              branches: [[name: "*/${params.BRANCH.trim()}"]],
              doGenerateSubmoduleConfigurations: false,
              extensions: [],
              submoduleCfg: [],
              userRemoteConfigs: [[
                  credentialsId: 'githubMAT',
                  url: "${params.REPO_URL.trim()}",
                  refspec: "+refs/heads/*:refs/remotes/origin/* +refs/pull/*:refs/remotes/origin/pull/*"
              ]]
          ])

        // Error handling when the repository checkout fails
        } catch (Exception e) {
          jira_add_comment("[ERROR] (#${BUILD_NUMBER}): could not checkout repo; verify repo URL and branch, and try again")
          error("Could not checkout repo (${e.message})")
        }
      }
      // Run tests using Maven
stage('run-tests') {
    container('maven') {
        sh("""
            # Crear un enlace simbólico para Python
            ln -sf python3 /usr/bin/python

            # Crear un entorno virtual
            python3 -m venv /opt/venv
            ls /opt/venv/bin

            # Instalar pip y markdownify
            /opt/venv/bin/pip install --upgrade pip
            /opt/venv/bin/pip install markdownify

            # Verificar versiones de Python y pip
            /opt/venv/bin/python --version
            /opt/venv/bin/pip --version  

            # Ejecutar pruebas de Maven
            mvn clean test --settings maven/settings.xml                         \
                -Dbuild_id=${BUILD_NUMBER}            \
                -Djob_name=${JOB_NAME}                \
                -Denvironment=${params.ENV_TO_TEST}       \
                -Dapp_url=${params.URL_APP.trim()}     \
                -Djira_pk=${params.JIRA_PROJECT_KEY}  \
                -Djira_issue=${params.JIRA_ISSUE_KEY} \
                -Dheadless=false || true

            # Crear un script temporal para convertir HTML a Markdown
            echo 'import markdownify
import sys

# Leer el HTML desde stdin
html_content = sys.stdin.read()

# Función personalizada para transformar el HTML a Markdown
def custom_markdownify(html):
    return markdownify.markdownify(html, strip=["a"])  # Remueve etiquetas <a>

# Convertir el HTML a Markdown utilizando la función personalizada
md_content = custom_markdownify(html_content)

# Imprimir el Markdown convertido
print(md_content)
' > /opt/venv/convert.py

            # Convertir el archivo HTML a Markdown utilizando la función personalizada
            /opt/venv/bin/python /opt/venv/convert.py < target/surefire-reports/emailable-report.html > report/report.md
        """)
    }
}


      stage('jira-upload-results') {
        jira_upload_results()
      }
      stage('jenkins-publish-report') {
        publishHTML([
          allowMissing: false,
          alwaysLinkToLastBuild: true,
          keepAll: true,
          reportDir: 'target/surefire-reports/',
          reportFiles: 'index.html',
          reportName: 'HTML Report',
          reportTitles: '',
          useWrapperFileDirectly: true
        ])
      }
      stage('jira-upload-report') {
        jira_upload_report()
      }
      // Evaluate the quality threshold
      stage('thresholds-eval') {
        try {
          if (params.QUALITY_GATE) {
            def threshold = 0
            if (params.UMBRAL?.isNumber()) threshold = clamp(params.UMBRAL.toInteger(), 0, 100)
            else echo("[WARNING]: Threshold value not valid, using 0 instead")
            testNG(failedFails: threshold, unstableFails: 100)
          }
          else echo("[WARNING]: Quality Gate is disabled; pipeline won't raise errors")
        } catch (Exception e) {
          jira_add_comment("[ERROR] (#${BUILD_NUMBER}): threshold value not in bounds of i32 (-(2^32), (2^32)); verify threshold and try again")
          error("threshold value not in bounds of i32 (${e.message})")
        }
      }
    /*
      
        stage('push-markdown-to-github') {

            withCredentials([usernamePassword(credentialsId: 'githubMAT', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) 
            {

              script {
                  // Usar Groovy para manejar la interpolación de variables
                  def branch = params.BRANCH
                  def gitUrl = env.GIT_URL ?: params.REPO_URL
                  def urlParts = gitUrl.split('/')
                  def organization = urlParts[-2]  // Penúltimo elemento es la organización
                  def repoName = urlParts[-1].replace('.git', '')  // Último elemento es el repositorio (sin .git)

                /*  sh """
                      git config --global user.email 'juan.a.mora@accenture.com'
                      git config --global user.name 'GMRLJN0025_gencat'
                      git checkout ${branch}
                      git add report/report.md
                      git commit -m 'Adding Functional Test Markdown report for build ${BUILD_NUMBER}'
                      git push https://${GIT_USER}:${GIT_PASS}@github.com/${organization}/${repoName} ${branch}
                  """
            
              }
           }

          }   

        
      stage('attach-md-to-github-issue') {
              withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                  def gitUrl = env.GIT_URL ?: params.REPO_URL
                  def urlParts = gitUrl.split('/')
                  def organization = urlParts[-2]  // Penúltimo elemento es la organización
                  def repoName = urlParts[-1].replace('.git', '')  // Último elemento es el repositorio (sin .git)
                  def issueUrl = "https://api.github.com/repos/${organization}/${repoName}/issues/${params.GITHUB_ISSUE_NUMBER}/comments"

                  def mdFileContent = readFile('report/report.md') // Lee el contenido del archivo markdown

                  def payload = [
                      body: "Adjuntando el reporte de OWASP ZAP:\n\n" + mdFileContent // Incluye el contenido del archivo en el comentario
                  ]
                  
                  def response = httpRequest(
                      acceptType: 'APPLICATION_JSON',
                      contentType: 'APPLICATION_JSON',
                      httpMode: 'POST',
                      url: issueUrl,
                      requestBody: groovy.json.JsonOutput.toJson(payload),
                      customHeaders: [[name: 'Authorization', value: "Bearer ${GITHUB_TOKEN}"]],
                      validResponseCodes: '201'
                  )

                  echo "Archivo .md adjuntado a la issue de GitHub exitosamente."
              }
          }   */  
          stage('attach-md-to-github-pull-request') {
            try {
              if (params.GITHUB_PULL_REQUEST_NUMBER.isEmpty()) {
                echo("[WARNING]: attach-md-to-github-pull-request :: skipped due to missing parameters (Pull Request)")
              }
              else
              {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    def gitUrl = env.GIT_URL ?: params.REPO_URL
                    def urlParts = gitUrl.split('/')
                    def organization = urlParts[-2]  // Penúltimo elemento es la organización
                    def repoName = urlParts[-1].replace('.git', '')  // Último elemento es el repositorio (sin .git)
                    def pullRequestUrl = "https://api.github.com/repos/${organization}/${repoName}/issues/${params.GITHUB_PULL_REQUEST_NUMBER}/comments"

                    def mdFileContent = readFile('report/report.md') // Lee el contenido del archivo markdown

                    def payload = [
                        body: "Adjuntando el reporte de Pruebas Funcionales:\n\n" + mdFileContent // Incluye el contenido del archivo en el comentario
                    ]
                    
                    def response = httpRequest(
                        acceptType: 'APPLICATION_JSON',
                        contentType: 'APPLICATION_JSON',
                        httpMode: 'POST',
                        url: pullRequestUrl,
                        requestBody: groovy.json.JsonOutput.toJson(payload),
                        customHeaders: [[name: 'Authorization', value: "Bearer ${GITHUB_TOKEN}"]],
                        validResponseCodes: '201'
                    )

                    echo "Archivo .md adjuntado a la pull request de GitHub exitosamente."
                }
              }
            } catch (Exception e) {
              jira_add_comment("[ERROR] (#${BUILD_NUMBER}): attach-md-to-github-pull-request :: ${e.message}")
              error(" (${e.message})")
            }
          }          
      
      // Final message indicating the result of the pipeline
      def tail_msg = currentBuild.result == 'FAILURE' ? 'with errors, check the report published.' : 'successufully'
      jira_add_comment("EXECUCIÓ FINALITZADA (#${BUILD_NUMBER}) :: Finished ${tail_msg}")
    }
  }
  // General exception handling
} catch (Exception e) {
  jira_add_comment("[ERROR] (#${BUILD_NUMBER}): ${e.message}")
  throw e // Re-throw the exception to stop the pipeline
}
}

```


## Integració amb Ecosistema MAT

<div class="grid cards" markdown>

-   [Documentació Tècnica](https://ctti.gencat.cat/mat-docs){ .md-button }
-   [Exemples d'Implementació](../examples){ .md-button }
-   [Guia de Troubleshooting](../troubleshooting){ .md-button }

</div>
