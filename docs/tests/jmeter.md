<h1 align="center"><img src="https://jmeter.apache.org/images/logo.svg" alt="Logotip d'Apache JMeter" /></h1>

Github: https://github.com/apache/jmeter/tree/master

Una aplicació Java de Codi Obert dissenyada per mesurar el rendiment i fer proves de càrrega d'aplicacions.

Per The Apache Software Foundation

[![codecov](https://codecov.io/gh/apache/jmeter/branch/master/graph/badge.svg)](https://codecov.io/gh/apache/jmeter)
[![Llicència](https://img.shields.io/:llicència-apache-llum_verd.svg)](https://www.apache.org/licenses/LICENSE-2.0.html)
[![Stack Overflow](https://img.shields.io/:stack%20overflow-jmeter-llum_verd.svg)](https://stackoverflow.com/questions/tagged/jmeter)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/org.apache.jmeter/ApacheJMeter/badge.svg)](https://maven-badges.herokuapp.com/maven-central/org.apache.jmeter/ApacheJMeter)
[![Twitter](https://img.shields.io/twitter/url/https/github.com/apache/jmeter.svg?style=social)](https://twitter.com/intent/tweet?text=Rendiment%20amb%20Apache%20JMeter:&url=https://jmeter.apache.org)

## Què és?

Apache JMeter pot mesurar el rendiment i fer proves de càrrega d'aplicacions web estàtiques i dinàmiques.

Pot ser utilitzat per simular una càrrega pesada en un servidor, grup de servidors,
xarxa o objecte per provar la seva resistència o per analitzar el rendiment en general sota diferents tipus de càrrega.

![Pantalla de JMeter](https://raw.githubusercontent.com/apache/jmeter/master/xdocs/images/screenshots/jmeter_screen.png)

## Característiques

Completa portabilitat i 100% Java.

Multiprocés que permet mostrejar concurrent per molts fils i
mostreig simultani de diferents funcions per grups de fils separats.

### Protocols

Capacitat per carregar i fer proves de rendiment en moltes aplicacions/server/tipus de protocols:

- Web - HTTP, HTTPS (Java, NodeJS, PHP, ASP.NET,...)
- Serveis Web SOAP / REST
- FTP
- Base de dades a través de JDBC
- LDAP
- Middleware orientat a missatges (MOM) a través de JMS
- Correu - SMTP(S), POP3(S) i IMAP(S)
- Comandes natives o scripts de shell
- TCP
- Objectes Java

### IDE

IDE de Proves completament funcional que permet una **gravació** ràpida de Pla de Proves
(des de Navegadors o aplicacions natives), **construcció** i **depuració**.

### Línia de comandes

[Mode de línia de comandes (Mode no gràfic / mode headless)](https://jmeter.apache.org/usermanual/get-started.html#non_gui)
per fer proves de càrrega des de qualsevol OS compatible amb Java (Linux, Windows, Mac OSX, ...)

### Informes

Un complet i a punt per presentar [informe HTML dinàmic](https://jmeter.apache.org/usermanual/generating-dashboard.html)

[Informes en temps real](https://jmeter.apache.org/usermanual/realtime-results.html)
en bases de dades de tercers com InfluxDB o Graphite

### Correlació

Fàcil correlació a través de la capacitat per extreure dades dels formats de resposta més populars,
[HTML](https://jmeter.apache.org/usermanual/component_reference.html#CSS/JQuery_Extractor),
[JSON](https://jmeter.apache.org/usermanual/component_reference.html#JSON_Extractor),
[XML](https://jmeter.apache.org/usermanual/component_reference.html#XPath_Extractor) o
[qualsevol format textual](https://jmeter.apache.org/usermanual/component_reference.html#Regular_Expression_Extractor)

### Nucle altament extensible

- Els Moldejadors enfichables permeten capacitats de prova il·limitades.
- **Moldejadors Scriptables** (llenguatges compatibles JSR223 com Groovy).
- Diverses estadístiques de càrrega es poden triar amb **tiers enfichables**.
- Els plugins d'anàlisi de dades i **visualització** permeten una gran extensibilitat i personalització.
- Les funcions es poden utilitzar per proporcionar entrada dinàmica a una prova o proporcionar manipulació de dades.
- Fàcil Integració Contínua mitjançant llibreries d'Open Source de tercers per a Maven, Gradle i Jenkins.

## La última versió

Els detalls de l'última versió es poden trobar a la
[pàgina web del projecte Apache JMeter](https://jmeter.apache.org/)

## Requisits

Els següents requisits són necessaris per a l'execució d'Apache JMeter:

- Intèrpret de Java:

  Es requereix un entorn d'execució de Java 8 completament compatible
  per a l'execució d'Apache JMeter. Un JDK amb l'eina `keytool` és més adequat
  per a la gravació de llocs web HTTPS.

- Jars opcionals:

  Alguns jars no s'inclouen amb JMeter.
  Si es requereix, aquests s'han de descarregar i col·locar al directori lib
  - JDBC - disponible des del proveïdor de la base de dades
  - JMS - disponible des del proveïdor de JMS
  - [Bouncy Castle](https://www.bouncycastle.org/) -
  només és necessari per a SMIME Assertion

- Compilador de Java (*OPCIONAL*):

  No es necessita un compilador de Java ja que la distribució inclou un
  arxiu binari de Java precompilat.
  > **Nota** que es requereix un compilador per construir plugins per a Apache JMeter.

## Instruccions d'instal·lació

> **Nota** que els espais en els noms de directors poden causar problemes.

- Construccions de llançament

  Desempaquetar l'arxiu binari en una estructura de directoris adequada.

## Diseño del proyecto 

```
performance-test
│   TestPlanCTTI.jmx            -> Arxiu JMX amb la configuració del test de rendiment
│   user.properties             -> Arxiu de propietats
│
└───.devcontainer               -> Definició del contenidor IDE VSCode
    │   devcontainer.json

```

## Clonar repositori
```bash
git clone https://git.intranet.gencat.cat/devsecopsctti/3632-devsecopsctti-performance-test.git
```
Si utilitza VS Code & Docker el projecte es pot executar dins d'un contenidor

## Configurar variables

1. Obri l'arxiu user.properties 
2. Canvia els valors segons sigui necessari

- `threadCount` és el nombre d'usuaris
- `rampup` és el temps de calentament
- `testDuration` és la durada del test
- `url` és l'URL de l'aplicació
- `protocol` és el protocol

## Com executar JMeter

1. Canvia al directori `bin` o estableix PATH amb la carpeta bin de jmeter
2. Crear carpeta on es guadarán els informes HTML p.e `HTMLReports`
3. Executa el fitxer `jmeter` (Unix)
```bash
jmeter  -n -t TestPlanCTTI.jmx  -e -o HTMLReports -l log.jtl -q user.properties
```
4. o el fitxer `jmeter`(Windows)
```bash
jmeter.bat  -n -t TestPlanCTTI.jmx  -e -o HTMLReports -l log.jtl -q user.properties
```

## Informació per a desenvolupadors

La construcció i contribució s'explica en detall a
[construir JMeter](https://jmeter.apache.org/building.html)
i [CONTRIBUTING.md](CONTRIBUTING.md). Hi ha més informació disponible sobre les tasques disponibles per
construir JMeter amb Gradle a [gradle.md](gradle.md).

Es pot obtenir el codi des de:

- https://github.com/apache/jmeter
- https://gitbox.apache.org/repos/asf/jmeter.git

## Llicència i Informació Legal

Per a informació legal i de llicència, si us plau, consulteu els següents fitxers:

- [LICENSE](LICENSE)
- [NOTICE](NOTICE)