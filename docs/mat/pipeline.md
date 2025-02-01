# MAT - Continuous Deployment i Testing

![MAT Logo](../assests/senyal.jpg){ align=right width="200" }

## Continuous Deployment Preproducció - Microserveis

<figure markdown>
  ![Diagrama de Continuous Deployment](../images/pipeline.png){ loading=lazy }
  <figcaption>Diagrama del procés de Continuous Deployment</figcaption>
</figure>

## Fases del Procés

!!! note "Procés Automatitzat"
    El procés de Continuous Deployment està completament automatitzat, assegurant una entrega ràpida i consistent.

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

!!! tip "Millora Contínua"
    El procés de CD està en constant evolució. Si tens suggeriments de millora, no dubtis en compartir-los!

---

<div class="grid cards" markdown>

- :fontawesome-solid-magnifying-glass-chart: **Monitoratge en Temps Real**
- :fontawesome-solid-code-branch: **Control de Versions**
- :fontawesome-solid-robot: **Automatització Intel·ligent**
- :fontawesome-solid-shield-halved: **Seguretat Integrada**

</div>


