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
%%{init: {'theme':'neutral'}}%%
flowchart TD
    A[Inici Pipeline] --> B[Validació Paràmetres]
    B --> C[Checkout Codi]
    C --> D[Execució Proves Selenium]
    D --> E[Pujada Resultats JIRA]
    E --> F[Publicació Informe HTML]
    F --> G{Avaluació Quality Gate}
    G -->|Èxit| H[Notificació Èxit]
    G -->|Error| I[Notificació Error]
    H --> Z([Fi])
    I --> Z

    classDef stage fill:#4CAF50,stroke:#388E3C,color:white;
    classDef decision fill:#FFC107,stroke:#FFA000;
    classDef error fill:#F44336,stroke:#D32F2F,color:white;
    class A,B,C,D,E,F stage
    class G decision
    class I,H error

```

### 2. Execució de Proves
??? tip "Tecnologies Utilitzades"
    - Selenium per a proves funcionals
    - Maven com a gestor de dependències
    - Extent per a informes executives

```mermaid
%%{init: {'theme':'neutral'}}%%
flowchart TD
A([Inici]) --> B[Validació Paràmetres]
B --> C[Checkout Codi]
C --> D[Validar Issue JIRA]
D --> E[Executar Proves Maven]
E --> F[Pujar Resultats JIRA]
F --> G[Publicar Informe HTML]
G --> H[Pujar Informe JIRA]
H --> I{{QUALITY_GATE?}}
I -->|Activat| J[Avaluar Umbral]
I -->|Desactivat| K[Saltar Control]
J --> L{Errors < UMBRAL?}
L -->|Sí| M[Notificar Èxit]
L -->|No| N[Aturar Pipeline]
K --> M
M --> O[Adjuntar MD a GitHub PR]
N --> O
O --> P([Fi])
classDef stage fill:#4CAF50,stroke:#388E3C,color:white;
classDef decision fill:#FFC107,stroke:#FFA000;
classDef error fill:#F44336,stroke:#D32F2F,color:white;
class A,B,C,D,E,F,G,H,J stage
class I,L decision
class N error



```

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
Notificació --> Teams: Notifica equip
JIRA --> []
Teams --> [*]
```

## Integració amb Ecosistema MAT

<div class="grid cards" markdown>

-   [Documentació Tècnica](https://ctti.gencat.cat/mat-docs){ .md-button }
-   [Exemples d'Implementació](../examples){ .md-button }
-   [Guia de Troubleshooting](../troubleshooting){ .md-button }

</div>
