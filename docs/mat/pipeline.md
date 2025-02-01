# Continuous Deployment i Testing Preproducció - Microserveis { .md-typeset }
![Jenkins Pipeline](../images/jenkins-pipeline.png){ align=right width="150" }

## Fases del Procés

!!! note "Procés Automatitzat"
    El procés de Continuous Deployment està completament automatitzat, assegurant una entrega ràpida i consistent.

<figure markdown>
  ![Diagrama de Continuous Deployment](../images/pipeline.png){ loading=lazy }
  <figcaption>Diagrama del procés de Continuous Deployment</figcaption>
</figure>


### 1. Inici del Desplegament (CD) :rocket:

Punt d'entrada del procés de desplegament continu. S'inicia automàticament quan es detecten canvis al repositori.

### 2. Verificació d'Artefactes :package:

Comprovació dels components necessaris pel desplegament, assegurant que tots els artefactes estiguin disponibles i siguin vàlids.

### 3. Configuració de l'Entorn :gear:

Preparació de la matriu d'entorns necessària pel desplegament, configurant les variables i dependències específiques.

### 4. Auditoria Prèvia :shield:

Avaluació de seguretat i compliment normatiu abans del desplegament per identificar possibles riscos.

### 5. Desplegament :arrow_up:

Execució del procés de desplegament dels microserveis a l'entorn corresponent.

### 6. Verificació de Salut :heartbeat:

Comprovació de l'estat dels serveis després del desplegament, verificant la seva disponibilitat i funcionament correcte.

### 7. Testing Automatitzat (MAT) :microscope:

Execució de les proves automatitzades utilitzant el Marc d'Automatització de Testing:

| Tipus de Prova | Descripció |
|----------------|------------|
| Proves d'API | Validació dels endpoints i integracions |
| Proves Funcionals | Verificació del comportament esperat |
| Proves de Rendiment | Avaluació del rendiment sota càrrega |

### 8. Auditoria Posterior :clipboard:

Verificació final de seguretat i qualitat després del desplegament i les proves.

### 9. Comunicació :envelope:

Notificació automàtica dels resultats del desplegament i les proves als interessats via correu electrònic.

### 10. Gestió d'Incidències (ITSM - REMEDY) :wrench:

Sistema integrat de gestió d'incidències que s'activa automàticament en cas de detectar problemes durant qualsevol fase del procés.

## Paràmetres d'Entrada de la Pipeline

Els següents paràmetres s’utilitzen per configurar l'execució de la pipeline i s’obtenen del Jenkinsfile:

### MAT-PROVES-FUNCIONAL

| **Paràmetre**             | **Valor per defecte**                                                               | **Descripció**                                                        |
|---------------------------|-------------------------------------------------------------------------------------|-----------------------------------------------------------------------|
| `FUNC_REPO_URL`           | `https://github.com/ctti-dev/3632.00-mat-functional-tests`                          | URL del repositori de proves funcionals                               |
| `FUNC_BRANCH`             | `master`                                                                            | Rama del repositori de proves funcionals                              |
| `FUNC_ENV_TO_TEST`        | `Produccio`                                                                         | Entorn per executar les proves funcionals                             |
| `FUNC_URL_APP`            | `https://qualitat.solucions.gencat.cat`                                               | URL de l'aplicació objectiu de les proves funcionals                    |
| `FUNC_UMBRAL`             | `20`                                                                                | Umbral de fallades per a les proves funcionals                          |
| `FUNC_QUALITY_GATE`       | `false`                                                                             | Activar/desactivar el Quality Gate per les proves funcionals            |
| `FUNC_JIRA_PROJECT_KEY`   | (Cadena buida)                                                                      | Clau del projecte Jira associat a les proves funcionals                 |
| `FUNC_JIRA_ISSUE_KEY`     | (Cadena buida)                                                                      | Clau de la incidencia Jira (Test Plan) per a les proves funcionals         |

### MAT-PROVES-RENDIMENT

| **Paràmetre**             | **Valor per defecte**                                                      | **Descripció**                                                                                     |
|---------------------------|----------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| `PERF_REPO_URL`           | `https://github.com/ctti-dev/3632.00-mat-performance-tests`                | URL del repositori de proves de rendiment                                                          |
| `PERF_BRANCH`             | `master`                                                                   | Rama del repositori de proves de rendiment                                                         |
| `PERF_PROTOCOL`           | `https`                                                                    | Protocol de connexió per a les proves de rendiment                                                 |
| `PERF_URL_APP`            | `qualitat.solucions.gencat.cat`                                            | URL de l'aplicació objectiu per a proves de rendiment                                               |
| `PERF_ENV_TO_TEST`        | `Produccio`                                                                | Entorn per executar les proves de rendiment                                                        |
| `PERF_TEST_DURATION`      | `10`                                                                       | Durada de les proves de rendiment (en minuts)                                                      |
| `PERF_RAMP_UP_TIME`       | `60`                                                                       | Temps de rampa per a augment gradual de càrrega                                                    |
| `PERF_THREAD_COUNT`       | `20`                                                                       | Nombre de fils (hilos) a simular en la prova de rendiment                                          |
| `PERF_QUALITY_GATE`       | `false`                                                                    | Activar/desactivar el Quality Gate per a proves de rendiment                                       |
| `PERF_UMBRAL`             | `20`                                                                       | Umbral de fallades per a les proves de rendiment                                                   |
| `PERF_JIRA_PROJECT_KEY`   | (Cadena buida)                                                             | Clau del projecte Jira per a proves de rendiment                                                   |
| `PERF_JIRA_ISSUE_KEY`     | (Cadena buida)                                                             | Clau de la incidència Jira per al Test Plan de proves de rendiment                                   |

### MAT-PROVES-API

| **Paràmetre**             | **Valor per defecte**                                                      | **Descripció**                                                                          |
|---------------------------|----------------------------------------------------------------------------|-----------------------------------------------------------------------------------------|
| `API_REPO_URL`            | `https://github.com/ctti-dev/3632.00-mat-api-tests.git`                    | URL del repositori de codi font per a proves d'API                                       |
| `API_BRANCH`              | `master`                                                                   | Rama del repositori de proves d'API                                                     |
| `API_APP_NAME`            | `conference`                                                               | Nom de l'aplicació objectiu per a les proves d'API                                        |
| `API_ENV_TO_TEST`         | *(Utilitza opcions: Desenvolupament, Integracio, Preproduccio, Produccio)*   | Entorns disponibles per a provar l'API                                                  |
| `API_JIRA_PROJECT_KEY`    | `DEVSECOPS2`                                                               | Clau del projecte Jira per a les proves d'API                                             |
| `API_JIRA_ISSUE_KEY`      | `DEVSECOPS2-251`                                                           | Clau de la incidència Jira (Test Plan) per a les proves d'API                              |

---

!!! tip "Millora Contínua"
    El procés de CD està en constant evolució. Si tens suggeriments de millora, no dubtis en compartir-los!

---

<div class="grid cards" markdown>

- :fontawesome-solid-magnifying-glass-chart: **Monitoratge en Temps Real**
- :fontawesome-solid-code-branch: **Control de Versions**
- :fontawesome-solid-robot: **Automatització Intel·ligent**
- :fontawesome-solid-shield-halved: **Seguretat Integrada**

</div>


