## Contents

* [Project layout](#project-layout)
* [Quick start (Local Machine not Dockerized)](#quick-start-(local-machine-not-dockerized))
* [Execution modes](#execution-modes)
  + [Standalone](#standalone)
  + [Hub and Nodes](#hub-and-nodes)
    - [Docker networking](#docker-networking)
    - [Using different machines/VMs](#using-different-machinesvms)
    - [Docker Compose](#docker-compose)
  + [Fully distributed mode - Router, Queue, Distributor, EventBus, SessionMap and Nodes](#fully-distributed-mode---router-queue-distributor-eventbus-sessionmap-and-nodes)
* [Video recording](#video-recording)
* [Dynamic Grid](#dynamic-grid)
  + [Configuration example](#configuration-example)
  + [Execution with Hub & Node roles](#execution-with-hub--node-roles)
  + [Execution with Standalone roles](#execution-with-standalone-roles)
  + [Using Dynamic Grid in different machines/VMs](#using-dynamic-grid-in-different-machinesvms)
  + [Execution with Docker Compose](#execution-with-docker-compose)
  + [Video recording, screen resolution, and time zones in a Dynamic Grid](#video-recording-screen-resolution-and-time-zones-in-a-dynamic-grid)
* [Kubernetes](#deploying-to-kubernetes)
* [Configuring the containers](#configuring-the-containers)
  + [SE_OPTS Selenium Configuration Options](#se_opts-selenium-configuration-options)
  + [SE_JAVA_OPTS Java Environment Options](#se_java_opts-java-environment-options)
  + [Node configuration options](#node-configuration-options)
  + [Setting Sub Path](#setting-sub-path)
  + [Setting Screen Resolution](#setting-screen-resolution)
  + [Grid Url and Session Timeout](#grid-url-and-session-timeout)
  + [Session request timeout](#session-request-timeout)
  + [Increasing session concurrency per container](#increasing-session-concurrency-per-container)
  + [Running in Headless mode](#running-in-headless-mode)
* [Building the images](#building-the-images)
* [Waiting for the Grid to be ready](#waiting-for-the-grid-to-be-ready)
* [Debugging](#debugging)
* [Install certificates for Chromium based browsers](#install-certificates-for-Chromium-based-browsers)
* [Tracing in Grid](#tracing-in-grid)
* [Troubleshooting](#troubleshooting)


## Project layout 

```
functional-test
│   docker-compose-v3.yml      -> Docker Compose file for Selenium Grid.
│   TestNG.xml                 -> Definición de las pruebas
│   pom.xml                    -> Fichero de dependencias
│   README.md
│
└───.devcontainer                  -> Definición contenedores IDE VSCode
│   │   devcontainer.json
└───resources                  
│   │   resources.properties    -> Definición variables
│   |   spark-config.json       -> Definición variables informe html
├───src
│   └───main/java                         -> Clases de inicialización de los drivers
│   │   │   CapabilityFactory.java         
│   │   │   Init.java
│   │   │   ExecutionListener.java
|   |   |   ResultSender.java
│   │   │   OptionsManager.java
|   |   |   ExtentManager.java
│   │   │
│   │   └───Pages                         
│   │   │      QualityGenCatPage.java
│   │   │
│   │   └───Utilities
│   │          Common.java
│   │          QualityGenCatPage.java
│   │ 
│   └───main/resources                         
│   │   │   log4j2.properties           -> Fichero de configuración de la librería log4j
│   └───test/java     
│           QualityGenCatTest.java      -> Ejemplo Clase de prueba 
```

1. Classes Diagram (UML)

```mermaid
classDiagram
    class QualityGenCatTest {
        +qualityGenCatTest()
    }

    class QualityGenCatPage {
        -driver: WebDriver
        -log: Logger
        +QualityGenCatPage(driver: WebDriver, log: Logger)
        +navigateTo()
        +clickDestacatsLink()
        +clickIniciLink()
        +clickEinesLink()
        +clickSeleniumLink()
        +getTextDestacats(): String
    }

    class PageElements {
        <<final>>
        +qualitatGenCatUrl: String
        +cttiCatUrl: String
        <<final>>
        +usernameCssLocator: By
        <<final>>
        +passwordCssLocator: By
        <<final>>
        +loginMenuButtonLocator: By
        <<final>>
        +loginButtonLocator: By
        <<final>>
        +destacatsLocator: By
        <<final>>
        +ejemploLocator: By
        <<final>>
        +einesLink: By
        <<final>>
        +seleniumLink: By
        <<final>>
        +iniciLink: By
    }

    class Init {
        -driver: ThreadLocal<RemoteWebDriver>
        +setup(browser: String)
        +getDriver(): WebDriver
        +tearDown()
        +terminate()
    }

    class OptionsManager {
        <<static>>
        +getChromeOptions(): ChromeOptions
        <<static>>
        +getFirefoxOptions(): FirefoxOptions
        <<static>>
        +getEdgeOptions(): EdgeOptions
    }

    class CapabilityFactory {
        +capabilities: Capabilities
        +getCapabilities(browser: String): Capabilities
    }

    class Common {
        -driver: WebDriver
        +Common(driver: WebDriver)
        +setSize()
    }

    class ExtentManager{
        -static ExtentReports extent
        -static ExtentSparkReporter sparkReporter
        -static ThreadLocal<ExtentTest> extentTest
        +static getInstance() : ExtentReports
        +static createTest(testName : String) : ExtentTest
        +static getTest() : ExtentTest
        +static flushReports() : void
    }

    class ResultSender {
        -static application2: String
        -static token2: char[]
        -static influxDBClient: InfluxDBClient
        -static writeApi: WriteApiBlocking
        +static send(point: Point)
        +static getApplication(): String
    }

    class ExecutionListener implements ITestListener {
        +onTestStart(iTestResult: ITestResult)
        +onTestSuccess(iTestResult: ITestResult)
        +onTestFailure(iTestResult: ITestResult)
        +onTestSkipped(iTestResult: ITestResult)
        +onTestFailedButWithinSuccessPercentage(iTestResult: ITestResult)
        +onStart(iTestContext: ITestContext)
        +onFinish(iTestContext: ITestContext)
        -sendTestMethodStatus(iTestResult: ITestResult, context: ITestContext, status: String)
        -sendTestClassStatus(iTestContext: ITestContext)
    }

    QualityGenCatTest --|> Init
    QualityGenCatTest --> QualityGenCatPage
    QualityGenCatTest --> Common
    QualityGenCatPage --> PageElements
    Init --> CapabilityFactory
    QualityGenCatTest --> ExtentManager
    CapabilityFactory --> OptionsManager
    ResultSender --|> Init
    ExecutionListener --|> ITestListener



```

## Usage Recomended

Visit for more information: https://github.com/SeleniumHQ/docker-selenium/tree/trunk

1. Clone the example repo:

```bash
git clone https://git.intranet.gencat.cat/devsecopsctti/functional-test.git
```

2. Modify properties is resources.properties file
```bash
selenium.server.url.docker=http://admin:admin@selenium-router.selenium-grid:4444/wd/hub           //Selenium grid URL
selenium.server.url.local=http://host.docker.internal:4444/wd/hub    
selenium.driver.remoto = true
selenium.geckodriver.local = C:\\Program Files\\Mozilla Firefox\\firefox.exe
password = 1234

application=petclinic
maintainer=CTTI

//InfluxDB

token=N8q9E5ffCYlhJa0woci0
#urlInfluxDB=http://influxdb.ctti-eks.aws
urlInfluxDB=http://influxdb.monitor:8086
bucket=selenium
company=primary
```
3. Install Selenium Grid & Launch Test Execution

```bash
# Install Selenium Grid Mesh
docker-compose -f docker-compose-v3.yml up -d --scale chrome=1 --scale firefox=1 --scale edge=1

# test project execution
 mvn clean package -Dbuildnumber="valor_build" -Djobname="valor_job"  -Denvironment="entorno" -Durlgrid="http://localhost:4444/wd/hub"

```
1. Las variables: ```buildnumber, jobname```, son variables que serán actualizadas por Jenkins, pero en una ejecución local pueden tener cualquier valor.
2. ```environment```:tendrá en valor del entorno donde se están ejecutando las pruebas p.e.: desarrollo, preproduccion, produccion.
3. ```urlgrid```: es la url de Selenium Grid

## Quick start (Local Machine not Dockerized)

1. Download Selenium Server (Grid) from https://www.selenium.dev/downloads/

2. Download Chrome And Firefox Drivers compatibles with your navigator versions and copy them in the same folder that Selenium Server jar

3. Execute:

```bash
java -jar selenium-server-4.10.0.jar standalone
```

4. Open the url http://localhost:4444/wd/hub and check Selenium Grid is Up & Running

5. Modify properties is resources.properties file
```bash
selenium.server.url.docker=http://localhost:4444/wd/hub
selenium.server.url.local=http://host.docker.internal:4444/wd/hub    //if you are running test from a container you must put the url of your host, if not put localhost. Probably you must create some internal firewall rules like open 4444 port.
selenium.driver.remoto = false
selenium.geckodriver.local = C:\\Program Files\\Mozilla Firefox\\firefox.exe
password = 1234

application=petclinic
maintainer=CTTI

//InfluxDB

token=N8q9E5ffCYlhJa0woci0
#urlInfluxDB=http://influxdb.ctti-eks.aws
urlInfluxDB=http://influxdb.monitor:8086
bucket=selenium
company=primary

```
6. Test Project Execution

```bash
mvn clean package -Dbuildnumber="valor_build" -Djobname="valor_job"  -Denvironment="entorno" -Durlgrid="http://localhost:4444/wd/hub"
```

7. Test Project Execution in CI Tool

Pass Selenium Grid Url as a Property

```bash
mvn clean package -Dbuildnumber=$BUILD_NUMBER -Djobname=$JOB_NAME  -Denvironment="entorno" -Durlgrid="http://localhost:4444/wd/hub"
```
## Execution modes

### Standalone

![Firefox](https://raw.githubusercontent.com/alrra/browser-logos/main/src/firefox/firefox_24x24.png) Firefox 
```bash
docker run -d -p 4444:4444 --shm-size="2g" selenium/standalone-firefox:4.12.1-20230912
```

![Chrome](https://raw.githubusercontent.com/alrra/browser-logos/main/src/chrome/chrome_24x24.png) Chrome 
```bash
docker run -d -p 4444:4444 --shm-size="2g" selenium/standalone-chrome:4.12.1-20230912
```

![Edge](https://raw.githubusercontent.com/alrra/browser-logos/main/src/edge/edge_24x24.png) Edge
```bash
docker run -d -p 4444:4444 --shm-size="2g" selenium/standalone-edge:4.12.1-20230912
```

_Note: Only one Standalone container can run on port_ `4444` _at the same time._

___

### Hub and Nodes

There are different ways to run the images and create a Grid with a Hub and Nodes, check the following options.

#### Docker networking
The Hub and Nodes will be created in the same network and they will recognize each other by their container name.
A Docker [network](https://docs.docker.com/engine/reference/commandline/network_create/) needs to be created as a first step.

##### macOS/Linux

```bash
$ docker network create grid
$ docker run -d -p 4442-4444:4442-4444 --net grid --name selenium-hub selenium/hub:4.12.1-20230912
$ docker run -d --net grid -e SE_EVENT_BUS_HOST=selenium-hub \
    --shm-size="2g" \
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
    selenium/node-chrome:4.12.1-20230912
$ docker run -d --net grid -e SE_EVENT_BUS_HOST=selenium-hub \
    --shm-size="2g" \
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
    selenium/node-edge:4.12.1-20230912
$ docker run -d --net grid -e SE_EVENT_BUS_HOST=selenium-hub \
    --shm-size="2g" \
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
    selenium/node-firefox:4.12.1-20230912
```

##### Windows PowerShell

```powershell
$ docker network create grid
$ docker run -d -p 4442-4444:4442-4444 --net grid --name selenium-hub selenium/hub:4.12.1-20230912
$ docker run -d --net grid -e SE_EVENT_BUS_HOST=selenium-hub `
    --shm-size="2g" `
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 `
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 `
    selenium/node-chrome:4.12.1-20230912
$ docker run -d --net grid -e SE_EVENT_BUS_HOST=selenium-hub `
    --shm-size="2g" `
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 `
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 `
    selenium/node-edge:4.12.1-20230912
$ docker run -d --net grid -e SE_EVENT_BUS_HOST=selenium-hub `
    --shm-size="2g" `
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 `
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 `
    selenium/node-firefox:4.12.1-20230912
```

When you are done using the Grid, and the containers have exited, the network can be removed with the following command:

``` bash
# Removes the grid network
$ docker network rm grid
```

#### Using different machines/VMs
The Hub and Nodes will be created on different machines/VMs, they need to know each other's IPs to
communicate properly. If more than one node will be running on the same Machine/VM, they must be
configured to expose different ports.

##### Hub - Machine/VM 1
```bash
$ docker run -d -p 4442-4444:4442-4444 --name selenium-hub selenium/hub:4.12.1-20230912
```

##### Node Chrome - Machine/VM 2

###### macOS/Linux

```bash
$ docker run -d -p 5555:5555 \
    --shm-size="2g" \
    -e SE_EVENT_BUS_HOST=<ip-from-machine-1> \
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
    -e SE_NODE_HOST=<ip-from-machine-2> \
    selenium/node-chrome:4.12.1-20230912
```

###### Windows PowerShell

```powershell
$ docker run -d -p 5555:5555 `
    --shm-size="2g" `
    -e SE_EVENT_BUS_HOST=<ip-from-machine-1> `
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 `
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 `
    -e SE_NODE_HOST=<ip-from-machine-2> `
    selenium/node-chrome:4.12.1-20230912
```


##### Node Edge - Machine/VM 3

###### macOS/Linux

```bash
$ docker run -d -p 5555:5555 \
    --shm-size="2g" \
    -e SE_EVENT_BUS_HOST=<ip-from-machine-1> \
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
    -e SE_NODE_HOST=<ip-from-machine-3> \
    selenium/node-edge:4.12.1-20230912
```

###### Windows PowerShell

```powershell
$ docker run -d -p 5555:5555 `
    --shm-size="2g" `
    -e SE_EVENT_BUS_HOST=<ip-from-machine-1> `
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 `
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 `
    -e SE_NODE_HOST=<ip-from-machine-3> `
    selenium/node-edge:4.12.1-20230912
```

##### Node Firefox - Machine/VM 4

###### macOS/Linux

```bash
$ docker run -d -p 5555:5555 \
    --shm-size="2g" \
    -e SE_EVENT_BUS_HOST=<ip-from-machine-1> \
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
    -e SE_NODE_HOST=<ip-from-machine-4> \
    selenium/node-firefox:4.12.1-20230912
```

###### Windows PowerShell

```powershell
$ docker run -d -p 5555:5555 `
    --shm-size="2g" `
    -e SE_EVENT_BUS_HOST=<ip-from-machine-1> `
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 `
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 `
    -e SE_NODE_HOST=<ip-from-machine-4> `
    selenium/node-firefox:4.12.1-20230912
```

##### Node Chrome - Machine/VM 4

###### macOS/Linux

``` bash
$ docker run -d -p 5556:5556 \
    --shm-size="2g" \
    -e SE_EVENT_BUS_HOST=<ip-from-machine-1> \
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
    -e SE_NODE_HOST=<ip-from-machine-4> \
    -e SE_NODE_PORT=5556 \
    selenium/node-chrome:4.12.1-20230912
```

###### Windows PowerShell

```powershell
$ docker run -d -p 5556:5556 `
    --shm-size="2g" `
    -e SE_EVENT_BUS_HOST=<ip-from-machine-1> `
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 `
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 `
    -e SE_NODE_HOST=<ip-from-machine-4> `
    -e SE_NODE_PORT=5556 `
    selenium/node-chrome:4.12.1-20230912
```

#### Docker Compose
Visit offical github repository mentioned above in order to obtain yaml files!

[Docker Compose](https://docs.docker.com/compose/) is the simplest way to start a Grid. Use the
linked resources below, save them locally and check the execution instructions on top of each file.

##### Version 2
[`docker-compose-v2.yml`](docker-compose-v2.yml)

##### Version 3
[`docker-compose-v3.yml`](docker-compose-v3.yml)

To stop the Grid and cleanup the created containers, run `docker-compose down`.

##### Version 3 with Swarm support 
[`docker-compose-v3-swarm.yml`](docker-compose-v3-swarm.yml)

___

### Fully distributed mode - Router, Queue, Distributor, EventBus, SessionMap and Nodes

It is possible to start a Selenium Grid with all its components apart. For simplicity, only an
example with docker-compose will be provided. Save the file locally, and check the execution 
instructions on top of it.

[`docker-compose-v3-full-grid.yml`](docker-compose-v3-full-grid.yml)

___

## Video recording

Tests execution can be recorded by using the `selenium/video:ffmpeg-4.3.1-20230912`
Docker image. One container is needed per each container where a browser is running. This means if you are
running 5 Nodes/Standalone containers, you will need 5 video containers, the mapping is 1-1.

Currently, the only way to do this mapping is manually (either starting the containers manually or through
`docker-compose`). We are iterating on this process and probably this setup will be more simple in the future.

The video Docker image we provide is based on the ffmpeg Ubuntu image provided by the 
[jrottenberg/ffmpeg](https://github.com/jrottenberg/ffmpeg) project, thank you for providing this image and
simplifying our work :tada:

**Notes**:
- If you have questions or feedback, please use the community contact points shown [here](https://www.selenium.dev/support/). 
- Please report any bugs through GitHub [issues](https://github.com/SeleniumHQ/docker-selenium/issues/new/choose), and provide
all the information requested on the template.
- Video recording for headless browsers is not supported. 
- Video recording tends to use considerable amounts of CPU. Normally you should estimate 1CPU per video container, 
and 1 CPU per browser container.
- Videos are stored in the `/videos` directory inside the video container. Map a local directory to get the videos.
- If you are running more than one video container, be sure to overwrite the video file name through the `FILE_NAME`
environment variable to avoid unexpected results.

This example shows how to start the containers manually:

``` bash
$ docker network create grid
$ docker run -d -p 4444:4444 -p 6900:5900 --net grid --name selenium --shm-size="2g" selenium/standalone-chrome:4.12.1-20230912
$ docker run -d --net grid --name video -v /tmp/videos:/videos selenium/video:ffmpeg-4.3.1-20230912
# Run your tests
$ docker stop video && docker rm video
$ docker stop selenium && docker rm selenium
```
After the containers are stopped and removed, you should see a video file on your machine's `/tmp/videos` directory.

Here is an example using a Hub and a few Nodes:

[`docker-compose-v3-video.yml`](docker-compose-v3-video.yml)

___

## Dynamic Grid

Grid 4 has the ability to start Docker containers on demand, this means that it starts
a Docker container in the background for each new session request, the test gets executed
there, and when the test completes, the container gets thrown away.

This execution mode can be used either in the Standalone or Node roles. The "dynamic"
execution mode needs to be told what Docker images to use when the containers get started.
Additionally, the Grid needs to know the URI of the Docker daemon. This configuration can
be placed in a local `toml` file.

### Configuration example

You can save this file locally and name it, for example, `config.toml`.
```toml
[docker]
# Configs have a mapping between the Docker image to use and the capabilities that need to be matched to
# start a container with the given image.
configs = [
    "selenium/standalone-firefox:4.12.1-20230912", '{"browserName": "firefox"}',
    "selenium/standalone-chrome:4.12.1-20230912", '{"browserName": "chrome"}',
    "selenium/standalone-edge:4.12.1-20230912", '{"browserName": "MicrosoftEdge"}'
]

# URL for connecting to the docker daemon
# Most simple approach, leave it as http://127.0.0.1:2375, and mount /var/run/docker.sock.
# 127.0.0.1 is used because internally the container uses socat when /var/run/docker.sock is mounted 
# If var/run/docker.sock is not mounted: 
# Windows: make sure Docker Desktop exposes the daemon via tcp, and use http://host.docker.internal:2375.
# macOS: install socat and run the following command, socat -4 TCP-LISTEN:2375,fork UNIX-CONNECT:/var/run/docker.sock,
# then use http://host.docker.internal:2375.
# Linux: varies from machine to machine, please mount /var/run/docker.sock. If this does not work, please create an issue.
url = "http://127.0.0.1:2375"
# Docker image used for video recording
video-image = "selenium/video:ffmpeg-4.3.1-20230912"

# Uncomment the following section if you are running the node on a separate VM
# Fill out the placeholders with appropriate values
#[server]
#host = <ip-from-node-machine>
#port = <port-from-node-machine>
```

### Execution with Hub & Node roles

This can be expanded to a full Grid deployment, all components deployed individually. The overall
idea is to have the Hub in one virtual machine, and each of the Nodes in separate and more powerful
virtual machines. 

#### macOS/Linux

```bash
$ docker network create grid
$ docker run -d -p 4442-4444:4442-4444 --net grid --name selenium-hub selenium/hub:4.12.1-20230912
$ docker run -d --net grid -e SE_EVENT_BUS_HOST=selenium-hub \
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
    -v ${PWD}/config.toml:/opt/bin/config.toml \
    -v ${PWD}/assets:/opt/selenium/assets \
    -v /var/run/docker.sock:/var/run/docker.sock \
    selenium/node-docker:4.12.1-20230912
```

#### Windows PowerShell

```powershell
$ docker network create grid
$ docker run -d -p 4442-4444:4442-4444 --net grid --name selenium-hub selenium/hub:4.12.1-20230912
$ docker run -d --net grid -e SE_EVENT_BUS_HOST=selenium-hub `
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 `
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 `
    -v ${PWD}/config.toml:/opt/bin/config.toml `
    -v ${PWD}/assets:/opt/selenium/assets `
    -v /var/run/docker.sock:/var/run/docker.sock `
    selenium/node-docker:4.12.1-20230912
```

To have the assets saved on your host, please mount your host path to `/opt/selenium/assets`.

When you are done using the Grid, and the containers have exited, the network can be removed with the following command:

``` bash
# Removes the grid network
$ docker network rm grid
```

### Execution with Standalone roles

#### macOS/Linux

```bash
docker run --rm --name selenium-docker -p 4444:4444 \
    -v ${PWD}/config.toml:/opt/bin/config.toml \
    -v ${PWD}/assets:/opt/selenium/assets \
    -v /var/run/docker.sock:/var/run/docker.sock \
    selenium/standalone-docker:4.12.1-20230912
```

#### Windows PowerShell

```bash
docker run --rm --name selenium-docker -p 4444:4444 `
    -v ${PWD}/config.toml:/opt/bin/config.toml `
    -v ${PWD}/assets:/opt/selenium/assets `
    -v /var/run/docker.sock:/var/run/docker.sock `
    selenium/standalone-docker:4.12.1-20230912
```

### Using Dynamic Grid in different machines/VMs

#### Hub - Machine/VM 1

```bash
$ docker run -d -p 4442-4444:4442-4444 --name selenium-hub selenium/hub:4.12.1-20230912
```

#### Node Chrome - Machine/VM 2

#### macOS/Linux

```bash
$ docker run -d -p 5555:5555 \
    -e SE_EVENT_BUS_HOST=<ip-from-machine-1> \
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
    -v ${PWD}/config.toml:/opt/bin/config.toml \
    -v ${PWD}/assets:/opt/selenium/assets \
    -v /var/run/docker.sock:/var/run/docker.sock \
    selenium/node-docker:4.12.1-20230912
```

#### Windows PowerShell

```bash
$ docker run -d -p 5555:5555 `
    -e SE_EVENT_BUS_HOST=<ip-from-machine-1> `
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 `
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 `
    -v ${PWD}/config.toml:/opt/bin/config.toml `
    -v ${PWD}/assets:/opt/selenium/assets `
    -v /var/run/docker.sock:/var/run/docker.sock `
    selenium/node-docker:4.12.1-20230912
```

Complete the `[server]` section in the `config.toml` file.
```toml
[docker]
# Configs have a mapping between the Docker image to use and the capabilities that need to be matched to
# start a container with the given image.
configs = [
    "selenium/standalone-firefox:4.12.1-20230912", "{\"browserName\": \"firefox\"}",
    "selenium/standalone-chrome:4.12.1-20230912", "{\"browserName\": \"chrome\"}",
    "selenium/standalone-edge:4.12.1-20230912", "{\"browserName\": \"MicrosoftEdge\"}"
    ]

# URL for connecting to the docker daemon
# Most simple approach, leave it as http://127.0.0.1:2375, and mount /var/run/docker.sock.
# 127.0.0.1 is used because interally the container uses socat when /var/run/docker.sock is mounted 
# If var/run/docker.sock is not mounted: 
# Windows: make sure Docker Desktop exposes the daemon via tcp, and use http://host.docker.internal:2375.
# macOS: install socat and run the following command, socat -4 TCP-LISTEN:2375,fork UNIX-CONNECT:/var/run/docker.sock,
# then use http://host.docker.internal:2375.
# Linux: varies from machine to machine, please mount /var/run/docker.sock. If this does not work, please create an issue.
url = "http://127.0.0.1:2375"
# Docker image used for video recording
video-image = "selenium/video:ffmpeg-4.3.1-20230912"

# Uncomment the following section if you are running the node on a separate VM
# Fill out the placeholders with appropriate values
[server]
host = <ip-from-node-machine>
port = <port-from-node-machine>
```

To have the assets saved on your host, please mount your host path to `/opt/selenium/assets`.

### Execution with Docker Compose

Here is an example using a Hub and a Node:

[`docker-compose-v3-dynamic-grid.yml`](docker-compose-v3-dynamic-grid.yml)


### Configuring the child containers

Containers can be further configured through environment variables, such as `SE_NODE_SESSION_TIMEOUT`
and `SE_OPTS`. When a child container is created, all environment variables prefixed with `SE_` will
be forwared and set in the container. You can set the desired environment variables in the 
`standalone-docker` or `node-docker` containers. The following example sets the session timeout to
700 seconds for all sessions:

#### macOS/Linux

```bash
docker run --rm --name selenium-docker -p 4444:4444 \
    -e SE_NODE_SESSION_TIMEOUT=700 \
    -v ${PWD}/config.toml:/opt/bin/config.toml \
    -v ${PWD}/assets:/opt/selenium/assets \
    -v /var/run/docker.sock:/var/run/docker.sock \
    selenium/standalone-docker:4.12.1-20230912
```

#### Windows PowerShell

```bash
docker run --rm --name selenium-docker -p 4444:4444 `
    -e SE_NODE_SESSION_TIMEOUT=700 `
    -v ${PWD}/config.toml:/opt/bin/config.toml `
    -v ${PWD}/assets:/opt/selenium/assets `
    -v /var/run/docker.sock:/var/run/docker.sock `
    selenium/standalone-docker:4.12.1-20230912
```



### Video recording, screen resolution, and time zones in a Dynamic Grid
To record your WebDriver session, you need to add a `se:recordVideo` 
field set to `true`. You can also set a time zone and a screen resolution,
for example:

```json
{
  "browserName": "firefox",
  "platformName": "linux",
  "se:recordVideo": "true",
  "se:timeZone": "US/Pacific",
  "se:screenResolution": "1920x1080"  
}
```

After running a test, check the path you mounted to the Docker container, 
(`${PWD}/assets`), and you should see videos and session information. 
___

## Deploying to Kubernetes

We offer a Helm chart to deploy these Docker images to Kubernetes.
Read more details at the Helm [readme](https://github.com/SeleniumHQ/docker-selenium/blob/trunk/charts/selenium-grid/README.md).

___

## Configuring the containers

### SE_OPTS Selenium Configuration Options

You can pass `SE_OPTS` variable with additional command line parameters for starting a hub or a node.

``` bash
$ docker run -d -p 4444:4444 -e SE_OPTS="--log-level FINE" --name selenium-hub selenium/hub:4.12.1-20230912
```

### SE_JAVA_OPTS Java Environment Options

You can pass `SE_JAVA_OPTS` environment variable to the Java process.

``` bash
$ docker run -d -p 4444:4444 -e SE_JAVA_OPTS=-Xmx512m --name selenium-hub selenium/hub:4.12.1-20230912
```

### Node configuration options

The Nodes register themselves through the Event Bus. When the Grid is started in its typical Hub/Node
setup, the Hub will be the one acting as the Event Bus, and when the Grid is started with all its five
elements apart, the Event Bus will be running on its own.

In both cases, it is necessary to tell the Node where the Event Bus is, so it can register itself. That is
the purpose of the `SE_EVENT_BUS_HOST`, `SE_EVENT_BUS_PUBLISH_PORT` and `SE_EVENT_BUS_SUBSCRIBE_PORT` environment
variables.

In some cases, for example, if you want to tag a node, it might be necessary to supply a custom stereotype to the node config. The environment variable `SE_NODE_STEREOTYPE`
sets the stereotype entry in the node's `config.toml`. An example config.toml file can be found here: [Setting custom capabilities for matching specific Nodes](https://www.selenium.dev/documentation/grid/configuration/toml_options/#setting-custom-capabilities-for-matching-specific-nodes).

Here is an example with the default values of these environment variables:
```bash
$ docker run -d \
  -e SE_EVENT_BUS_HOST=<event_bus_ip|event_bus_name> \
  -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
  -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 -e SE_NODE_STEREOTYPE="{\"browserName\":\"${SE_NODE_BROWSER_NAME}\",\"browserVersion\":\"${SE_NODE_BROWSER_VERSION}\",\"platformName\": \"Linux\"}" \
  --shm-size="2g" selenium/node-chrome:4.12.1-20230912
```

### Setting Sub Path

By default, Selenium is reachable at `http://127.0.0.1:4444/`. Selenium can be configured to use a custom subpath by specifying the `SE_SUB_PATH`
environmental variable. In the example below Selenium is reachable at `http://127.0.0.1:4444/selenium-grid/`

```bash
$ docker run -d -p 4444:4444 -e SE_SUB_PATH=/selenium-grid/ --name selenium-hub selenium/hub:4.9.0-20230421
```

### Setting Screen Resolution

By default, nodes start with a screen resolution of 1360 x 1020 with a color depth of 24 bits and a dpi of 96. 
These settings can be adjusted by specifying `SE_SCREEN_WIDTH`, `SE_SCREEN_HEIGHT`, `SE_SCREEN_DEPTH`, and/or `SE_SCREEN_DPI` 
environmental variables when starting the container.

``` bash
docker run -d -e SE_SCREEN_WIDTH=1366 -e SE_SCREEN_HEIGHT=768 -e SE_SCREEN_DEPTH=24 -e SE_SCREEN_DPI=74 selenium/standalone-firefox:4.12.1-20230912
```

### Grid Url and Session Timeout

In some use cases, you might need to set the Grid URL to the Node, for example, if you'd like to access the BiDi/CDP endpoint. 
This is also needed when you want to use the new `RemoteWebDriver.builder()` or `Augmenter()` present in Selenium 4 
(since they setup the BiDi/CDP connection implicitly). You can do that through the `SE_NODE_GRID_URL` environment 
variable. Setting this env var is needed if you want to see the live view while sessions are executing.

Grid has a default session timeout of 300 seconds, where the session can be in a stale state until it is killed. You can use
`SE_NODE_SESSION_TIMEOUT` to overwrite that value in seconds.


### Session request timeout

A new session request is placed in the Session Queue before it is processed, and the request sits in the queue until a matching
slot is found across the registered Nodes. However, the new session request might timeout if no slot was found. By default, a 
request will stay in the queue for up to 300 seconds before it a timeout is reached. In addition, an attempt to process the request
is done every 5 seconds (by default).

It is possible to override those values through environment variables in the Hub and the SessionQueue (`SE_SESSION_REQUEST_TIMEOUT`
and `SE_SESSION_RETRY_INTERVAL`). For example, a timeout of 500 seconds would be `SE_SESSION_REQUEST_TIMEOUT=500` and a retry 
interval of 2 seconds would be `SE_SESSION_RETRY_INTERVAL=2`.

### Increasing session concurrency per container

By default, only one session is configured to run per container through the `SE_NODE_MAX_SESSIONS` environment variable. It is
possible to increase that number up to the maximum available processors, this is because more stability is achieved when one
container/browser has 1 CPU to run. 

However, if you have measured performance and based on that, you think more sessions can be executed in each container, you can
override the maximum limit by setting both `SE_NODE_MAX_SESSIONS` to a desired number and `SE_NODE_OVERRIDE_MAX_SESSIONS` to 
`true`. Nevertheless, running more browser sessions than the available processors is not recommended since you will be overloading
the resources.

Overriding this setting has an undesired side effect when video recording is enabled since more than one browser session might be
captured in the same video.

### Running in Headless mode

[Firefox](https://developer.mozilla.org/en-US/docs/Mozilla/Firefox/Headless_mode), 
[Chrome](https://developers.google.com/web/updates/2017/04/headless-chrome), 
When using headless mode, there's no need for the [Xvfb](https://en.wikipedia.org/wiki/Xvfb) server to be started.

To avoid starting the server you can set the `START_XVFB` environment variable to `false` 
(or any other value than `true`), for example:

``` bash
$ docker run -d --net grid -e SE_EVENT_BUS_HOST=selenium-hub -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
  -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 -e START_XVFB=false --shm-size="2g" selenium/node-chrome:4.12.1-20230912
```

For more information, see this GitHub [issue](https://github.com/SeleniumHQ/docker-selenium/issues/567).

### Stopping the Node/Standalone after N sessions have been executed

In some environments, like Docker Swarm or Kubernetes, it is useful to shut down the Node or Standalone
container after N tests have been executed. For example, this can be used in Kubernetes to terminate the
pod and then scale a new one after N sessions. Set the environment variable `SE_DRAIN_AFTER_SESSION_COUNT` to
a value higher than zero to enable this behaviour. 

``` bash
$ docker run -e SE_DRAIN_AFTER_SESSION_COUNT=5 --shm-size="2g" selenium/standalone-firefox:4.12.1-20230912
```

With the previous command, the Standalone container will shut down after 5 sessions have been executed.

___

## Building the images

Clone the repo and from the project directory root you can build everything by running:

``` bash
$ VERSION=local make build
```

If you need to configure environment variables in order to build the image (http proxy for instance), 
simply set an environment variable `BUILD_ARGS` that contains the additional variables to pass to the 
docker context (this will only work with docker >= 1.9)

``` bash
$ BUILD_ARGS="--build-arg http_proxy=http://acme:3128 --build-arg https_proxy=http://acme:3128" make build
```

_Note: Omitting_ `VERSION=local` _will build the images with the released version but replacing the date for the 
current one._

___

## Waiting for the Grid to be ready

It is a good practice to check first if the Grid is up and ready to receive requests, this can be done by checking the `/wd/hub/status` endpoint.

A Grid that is ready, composed of a hub and two nodes, could look like this:

```json
{
  "value": {
    "ready": true,
    "message": "Selenium Grid ready.",
    "nodes": [
      {
        "id": "6c0a2c59-7e99-469d-bbfc-313dc638797c",
        "uri": "http:\u002f\u002f172.19.0.3:5555",
        "maxSessions": 4,
        "stereotypes": [
          {
            "capabilities": {
              "browserName": "firefox"
            },
            "count": 4
          }
        ],
        "sessions": [
        ]
      },
      {
        "id": "26af3363-a0d8-4bd6-a854-2c7497ed64a4",
        "uri": "http:\u002f\u002f172.19.0.4:5555",
        "maxSessions": 4,
        "stereotypes": [
          {
            "capabilities": {
              "browserName": "chrome"
            },
            "count": 4
          }
        ],
        "sessions": [
        ]
      }
    ]
  }
}
```

The `"ready": true` value indicates that the Grid is ready to receive requests. This status can be polled through a
script before running any test, or it can be added as a [HEALTHCHECK](https://docs.docker.com/engine/reference/run/#healthcheck)
when the docker container is started.

### Adding a [HEALTHCHECK](https://docs.docker.com/engine/reference/run/#healthcheck) to the Grid

The script [check-grid.sh](Base/check-grid.sh), which is included in the images, can be used to poll the Grid status.

This example checks the status of the Grid every 15 seconds, it has a timeout of 30 seconds when the check is done,
and it retries up to 5 times until the container is marked as unhealthy. Please use adjusted values to fit your needs,
(if needed) replace the `--host` and `--port` parameters for the ones used in your environment.

``` bash
$ docker network create grid
$ docker run -d -p 4442-4444:4442-4444 --net grid --name selenium-hub \
  --health-cmd='/opt/bin/check-grid.sh --host 0.0.0.0 --port 4444' \
  --health-interval=15s --health-timeout=30s --health-retries=5 \
  selenium/hub:4.12.1-20230912
$ docker run -d --net grid -e SE_EVENT_BUS_HOST=selenium-hub \
    --shm-size="2g" \
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
    selenium/node-chrome:4.12.1-20230912
$ docker run -d --net grid -e SE_EVENT_BUS_HOST=selenium-hub \
    --shm-size="2g" \
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
    selenium/node-edge:4.12.1-20230912
$ docker run -d --net grid -e SE_EVENT_BUS_HOST=selenium-hub \
    --shm-size="2g" \
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
    selenium/node-firefox:4.12.1-20230912

```
**Note:** The `\` line delimiter won't work on Windows-based terminals, try either `^` or a backtick.

The container health status can be checked by doing `docker ps` and verifying the `(healthy)|(unhealthy)` status or by
inspecting it in the following way:

```bash
$ docker inspect --format='{{json .State.Health.Status}}' selenium-hub
"healthy"
```

### Using a bash script to wait for the Grid

A common problem known in docker is that a running container does not always mean that the application inside it is ready.
A simple way to tackle this is by using a "wait-for-it" script, more information can be seen [here](https://docs.docker.com/compose/startup-order/).

The following script is an example of how this can be done using bash, but the same principle applies if you want to do this with the programming language used to write the tests.
In the example below, the script will poll the status endpoint every second. If the grid does not become ready within 30 seconds, the script will exit with an error code.

```bash
#!/bin/bash
# wait-for-grid.sh

set -e
url="http://localhost:4444/wd/hub/status"
wait_interval_in_seconds=1
max_wait_time_in_seconds=30
end_time=$((SECONDS + max_wait_time_in_seconds))
time_left=$max_wait_time_in_seconds

while [ $SECONDS -lt $end_time ]; do
    response=$(curl -sL "$url" | jq -r '.value.ready')
    if [ -n "$response"  ]  && [ "$response" ]; then
        echo "Selenium Grid is up - executing tests"
        break
    else
        echo "Waiting for the Grid. Sleeping for $wait_interval_in_seconds second(s). $time_left seconds left until timeout."
        sleep $wait_interval_in_seconds
        time_left=$((time_left - wait_interval_in_seconds))
    fi
done

if [ $SECONDS -ge $end_time ]; then
    echo "Timeout: The Grid was not started within $max_wait_time_in_seconds seconds."
    exit 1
fi
```
> Will require `jq` installed via `apt-get`, else the script will keep printing `Waiting` without completing the execution.

**Note:** If needed, replace `localhost` and `4444` for the correct values in your environment. Also, this script is polling indefinitely, you might want
to tweak it and establish a timeout.

Let's say that the normal command to execute your tests is `mvn clean test`. Here is a way to use the above script and execute your tests:

```bash
$ ./wait-for-grid.sh mvn clean test
```

Like this, the script will poll until the Grid is ready, and then your tests will start.

___

## Install certificates for Chromium-based browsers

If you need to install custom certificates, CA, intermediate CA, or client certificates (for example enterprise internal CA)
you can use the different utils that come from libnss3-tools.
The chromium-based browser uses nssdb as a certificate store.
Create a new nssdb:  
```bash
certutil -d sql:$HOME/.pki/nssdb -N
```
For example, to trust a root CA certificate for issuing SSL server certificates, use
```bash
certutil -d sql:$HOME/.pki/nssdb -A -t "C,," -n <certificate nickname> -i <certificate filename>
```
To import an intermediate CA certificate, use
```bash
certutil -d sql:$HOME/.pki/nssdb -A -t ",," -n <certificate nickname> -i <certificate filename>
```
You can find more information [here](https://chromium.googlesource.com/chromium/src/+/master/docs/linux/cert_management.md)
Usage example:
If your company has an internal CA you can create your own dockerimage from selenium node image.
You can then install all required internal certificates in your Dockerfile like this:
```bash
FROM selenium/node-edge:latest
USER root
COPY certs/ /etc/certs # copy over the certificate file

#=========
# libnss3-tools
# Network Security Service tools
# Manage certificates in nssdb (certutil, pk12util, modutil, shlibsign, signtool, ssltap)
#=========
RUN apt-get update -qqy \
  && apt-get -qqy install \
    libnss3-tools \
  && rm -rf /var/lib/apt/lists/* /var/cache/apt/*

RUN mkdir -p -m755 /home/seluser/.pki/nssdb \ #create nssdb folder
    && certutil -d sql:/home/seluser/.pki/nssdb -N --empty-password \ # create new db without password
    && certutil -d sql:/home/seluser/.pki/nssdb -A -t "C,," -n companyca -i /etc/certs/companeca.pem \ #trust company CA
    && pk12util -d sql:/home/seluser/.pki/nssdb -i client_cert.p12 -W password_of_clent_cert # client certificate install
```
This way the certificates will be installed and the node will start automatically as before.
___

## Alternative method: Add certificates to existing Selenium based images for browsers

As an alternative, you can add your certificate files to existing Selenium images. This practical example
assumes you have a known image to use as a build image and have a way to publish new images to your local
docker registry.

This example uses a RedHat-based distro as a build image (Rocky Linux) but it can be *any* Linux image of your choice.
Please note that build instruction will vary between distributions. You can check the instructions for Ubuntu
in the previous example.

The example also assumes your internal CA is already in */etc/pki/ca-trust/source/anchors/YOUR_CA.pem*,
the default location for Rocky Linux. Alternatively, you can also provide these files from your host and 
copy them into the build image.

For Chrome and Edge browsers, the recipe is the same, just adapt the image name (node-chrome or node-edge):
```
# Get a standard image for creating nssdb file
FROM rockylinux:8.6 as build
RUN yum install -y nss-tools
RUN mkdir -p -m755 /seluser/.pki/nssdb \
    && certutil -d sql:/seluser/.pki/nssdb -N --empty-password \
    && certutil -d sql:/seluser/.pki/nssdb -A -t "C,," -n YOUR_CA -i /etc/pki/ca-trust/source/anchors/YOUR_CA.pem \
    && chown -R 1200:1201 /seluser

# Start from Selenium image and add relevant files from build image
FROM selenium/node-chrome:4.12.1-20230912
USER root
COPY --from=build /seluser/ /home/seluser/
USER seluser
```

Example for Firefox:
```
# Get a standard image for working on
FROM rockylinux:8.6 as build
RUN mkdir -p "/distribution" "/certs" && \
    cp /etc/pki/ca-trust/source/anchors/YOUR_CA*.pem /certs/ && \
    echo '{ "policies": { "Certificates": { "Install": ["/opt/firefox-latest/YOUR_CA.pem"] }} }' >"/distribution/policies.json"

# Start from Selenium image and add relevant files from build image
FROM selenium/node-firefox:4.12.1-20230912
USER root
COPY --from=build /certs /opt/firefox-latest
COPY --from=build /distribution /opt/firefox-latest/distribution
USER seluser
```
___

## Debugging

This project uses [x11vnc](https://github.com/LibVNC/x11vnc) as a VNC server to allow users to inspect what is happening
inside the container. Users can connect to this server in two ways:

### Using a VNC client

The VNC server is listening to port 5900, you can use a VNC client and connect to it. Feel free to map port 5900 to 
any free external port that you wish.

The internal 5900 port remains the same because that is the configured port for the VNC server running inside the container. 
You can override it with the `SE_VNC_PORT` environment variable in case you want to use `--net=host`.

Here is an example with the standalone images, the same concept applies to the node images.
``` bash
$ docker run -d -p 4444:4444 -p 5900:5900 --shm-size="2g" selenium/standalone-chrome:4.12.1-20230912
$ docker run -d -p 4445:4444 -p 5901:5900 --shm-size="2g" selenium/standalone-edge:4.12.1-20230912
$ docker run -d -p 4446:4444 -p 5902:5900 --shm-size="2g" selenium/standalone-firefox:4.12.1-20230912
```

Then, you would use in your VNC client:
- Port 5900 to connect to the Chrome container
- Port 5901 to connect to the Edge container
- Port 5902 to connect to the Firefox container

If you get a prompt asking for a password, it is: `secret`. If you wish to change this, 
you can set the environment variable `SE_VNC_PASSWORD`.

If you want to run VNC without password authentication you can set the environment variable `SE_VNC_NO_PASSWORD=1`.

If you want to run VNC in view-only mode you can set the environment variable `SE_VNC_VIEW_ONLY=1`.

### Using your browser (no VNC client is needed)

This project uses [noVNC](https://github.com/novnc/noVNC) to allow users to inspect visually container activity with
their browser. This might come in handy if you cannot install a VNC client on your machine. Port 7900 is used to start
noVNC, so you will need to connect to that port with your browser.

Similarly to the previous section, feel free to map port 7900 to any free external port that you wish.
You can also override it with the `SE_NO_VNC_PORT` environment variable in case you want to use `--net=host`.

Here is an example with the standalone images, the same concept applies to the node images.
``` bash
$ docker run -d -p 4444:4444 -p 7900:7900 --shm-size="2g" selenium/standalone-chrome:4.12.1-20230912
$ docker run -d -p 4445:4444 -p 7901:7900 --shm-size="2g" selenium/standalone-edge:4.12.1-20230912
$ docker run -d -p 4446:4444 -p 7902:7900 --shm-size="2g" selenium/standalone-firefox:4.12.1-20230912
```

Then, you would use in your browser:
- http://localhost:7900/ to connect to the Chrome container
- http://localhost:7901/ to connect to the Edge container
- http://localhost:7902/ to connect to the Firefox container

If you get a prompt asking for a password, it is: `secret`.

### Disabling VNC
If You are running low on resources, or simply don't need to inspect running sessions, it is possible to not run VNC at all.
Just set 
```SE_START_VNC=false```
environment variable on the grid startup.

___

## Tracing in Grid

In order to enable tracing in the Selenium Grid container, the following commands can be executed:

```bash
docker network create grid
docker run -d -p 16686:16686 -p 14250:14250 --net grid --name jaeger jaegertracing/all-in-one:1.17
docker run -d -p 4442-4444:4442-4444 --net grid --name selenium-hub selenium/hub:4.12.1-20230912
docker run -d --net grid -e SE_EVENT_BUS_HOST=selenium-hub \
    --shm-size="2g" \
	-e SE_ENABLE_TRACING=true \
	-e JAVA_OPTS="-Dotel.traces.exporter=jaeger -Dotel.exporter.jaeger.endpoint=http://jaegar:14250 -Dotel.resource.attributes=service.name=selenium-hub" \
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
    selenium/node-chrome:4.12.1-20230912
docker run -d --net grid -e SE_EVENT_BUS_HOST=selenium-hub \
    --shm-size="2g" \
	-e SE_ENABLE_TRACING=true \
	-e JAVA_OPTS="-Dotel.traces.exporter=jaeger -Dotel.exporter.jaeger.endpoint=http://jaegar:14250 -Dotel.resource.attributes=service.name=selenium-node-edge" \
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
    selenium/node-edge:4.12.1-20230912
docker run -d --net grid -e SE_EVENT_BUS_HOST=selenium-hub \
    --shm-size="2g" \
	-e SE_ENABLE_TRACING=true \
	-e JAVA_OPTS="-Dotel.traces.exporter=jaeger -Dotel.exporter.jaeger.endpoint=http://jaegar:14250 -Dotel.resource.attributes=service.name=selenium-node-firefox" \
    -e SE_EVENT_BUS_PUBLISH_PORT=4442 \
    -e SE_EVENT_BUS_SUBSCRIBE_PORT=4443 \
    selenium/node-firefox:4.12.1-20230912
```

You can also refer to the below docker-compose yaml files to be able to start a simple grid (or) a dynamic grid.

* Simple Grid [v3 yaml file](docker-compose-v3-tracing.yml)
* Simple Grid [v2 yaml file](docker-compose-v2-tracing.yml)
* Dynamic Grid [v3 yaml file](docker-compose-v3-full-grid-tracing.yml)

You can view the [Jaegar UI](http://localhost:16686/) and trace your request.
___

## Troubleshooting

All output gets sent to stdout, so it can be inspected by running:
``` bash
$ docker logs -f <container-id|container-name>
```

You can increase the log output by passing environment variable to the containers:
```
SE_OPTS="--log-level FINE"
```

### `--shm-size="2g"`

Why is `--shm-size 2g` necessary?
> This is a known workaround to avoid the browser crashing inside a docker container, here are the documented issues for
[Chrome](https://code.google.com/p/chromium/issues/detail?id=519952) and [Firefox](https://bugzilla.mozilla.org/show_bug.cgi?id=1338771#c10).
The shm size of 2gb is arbitrary but known to work well, your specific use case might need a different value, it is recommended
to tune this value according to your needs.


### Headless

If you see the following selenium exceptions:

`Message: invalid argument: can't kill an exited process`

or

`Message: unknown error: Chrome failed to start: exited abnormally`

The reason _might_ be that you've set the `START_XVFB` environment variable to "false", but forgot to 
actually run Firefox, Chrome or Edge in headless mode.

### Mounting volumes to retrieve downloaded files

A common scenario is mounting a volume to the browser 
container in order to retrieve downloaded files. This
works well in Windows and macOS but not without 
workarounds in Linux. For more details, check this
well-documented [issue](https://github.com/SeleniumHQ/docker-selenium/issues/1095).

For example, while using Linux, you might be starting a
container in the following way:

```bash
docker run -d -p 4444:4444 --shm-size="2g" \
  -v /home/ubuntu/files:/home/seluser/files \
  selenium/standalone-chrome:4.12.1-20230912
```

That will mount the host `/home/ubuntu/files` directory
to the `/home/seluser/files` inside the container. The
problem happens because the volume will be mounted as
`root`; therefore, the browser cannot write a file to
that directory because it is running under the user 
`seluser`. This happens because that is how Docker mounts
volumes in Linux, more details in this [issue](https://github.com/moby/moby/issues/2259).

A workaround for this is to create a directory on the
host and change its permissions **before mounting the volume**. 
Depending on your user permissions, you might need to use 
`sudo` for some of these commands:

```bash
mkdir /home/ubuntu/files
chown 1200:1201 /home/ubuntu/files
```

After doing this, you should be able to download files
to the mounted directory. If you have a better workaround,
please send us a pull request! 







