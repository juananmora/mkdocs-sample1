#+AUTHOR: DevSecOps CTTI
#+TITLE: MAT Functional Tests :: ~template~

* Table of Contents :toc:
- [[#requirements][Requirements]]
- [[#usage][Usage]]
  - [[#selenium-grid-installation][Selenium Grid installation]]
  - [[#first-execution][First execution]]
  - [[#creating-a-test][Creating a test]]

* Requirements

- OpenJDK >= 11
- Maven > 3

* Input parameters

La llibreria que utilitzem, ~mat-selenium~, accepta i necessita un conjunt de parametres. Aquests son els obligatoris:

- ~app~ :: Nom de l'aplicació a ser provada. Acceptat via env. var. (i.e. ~MAT_TF_APP=...~), propietat de la JVM (i.e. ~-Dapp=...~), o arxiu de config. (i.e. ~app = ...~).
- ~app_url~ :: URL de l'aplicació a ser provada. Acceptat via env. var. (i.e. ~MAT_TF_APP_URL=...~), propietat de la JVM (i.e. ~-Dapp_url=...~), o arxiu de config. (i.e. ~app_url = ...~).
- ~maintainer~ :: ID del lot de manteniment a carreg del projecte. Acceptat via env. var. (i.e. ~MAT_TF_MAINTAINER=...~), propietat de la JVM (i.e. ~-Dmaintainer=...~), o arxiu de config. (i.e. ~maintainer = ...~).
- ~selenium_url~ :: URL del endpoint del webdriver del HUB de Selenium Grid. Acceptat via env. var. (i.e. ~MAT_TF_SELENIUM_URL=...~), propietat de la JVM (i.e. ~-Dselenium_url=...~), o arxiu de config. (i.e. ~selenium_url = ...~).

A més, per disposar de funcionalitat extra i/o personalitzar el comportament de l'execució, tenim aquests parametres extres:

- ~headless~ :: Controla si els navegadors s'inicien. ~true~ per defecte. Només s'accepta via una propietat de la JVM (i.e. ~-Dheadless=[true|false]~).
- ~selenium_firefox_driver~ :: /Path/ al binari del webdriver de Firefox (/Gecko/). Acceptat via env. var. (i.e. ~MAT_TF_SELENIUM_FIREFOX_DRIVER=...~) o propietat de la JVM (i.e. ~-Dselenium_firefox_driver=...~).
- ~influxdb_url~ :: ...
- ~influxdb_token~ :: ...
- ~influxdb_bucket~ :: ...
- ~influxdb_company~ :: ...
- ~environment~ :: ...
- ~build_id~ :: ...
- ~job_name~ :: ...
- ~jira_pk~ :: ID del projecte de Jira associat al projecte de testing a executar.
- ~jira_issue~ :: ID de la issue de Jira (Test Plan) la qual representa el projecte de testing, on s'asociaran els casos de prova implementats.

* Usage

Aquesta es la plantilla oficial del MAT per utilitzar i interactuar amb la llibreria ~mat-selenium~.

** Selenium Grid installation

Donat el cas que ja tinguem el /Selenium Grid/ desplegat i disponible a la nostra xarxa (al nostre propi PC o a un altre equip de la xarxa local), ja podem executar la plantilla i, posteriorment, fer-la nostra i ajustar-la a les necessitats de la nostra aplicació.

En cas contrari, en aquest repositori posem a lliure disposició l'arxiu ~docker-compose.yaml~, el qual desplegarà els 3 navegadors i el /hub/, cadascun al seu propi contenidor mitjançant /Docker/ i /Docker Compose/. Aquesta es la comanda per tal de dur-ho a terme:

#+begin_src sh
docker-compose up -d
#+end_src

** First execution

Una vegada tot està preparat, i assumint que el /Selenium Grid/ està desplegat i disponible en ~http://localhost:4444~, podem executar la plantilla amb el test d'exemple amb la següent comanda:

#+begin_src sh
mvn clean test -Dselenium_url="http://localhost:4444/wd/hub"
#+end_src

En acabar l'execució, podem consultar el /report/ HTML generat en ~target/report/index.html~.

** Creating a test

(...)

#+begin_src java
// src/test/java/Pages/GooglePage.java

package cat.gencat.mat.pages;

public final class GooglePage {
  //  ... attributes here ...

  public static void search(String s) {
    // ... implementation here ...
  }
}
#+end_src

(...)

#+begin_src java
// src/test/java/GoogleTest.java

import cat.gencat.mat.Utils;
import cat.gencat.mat.BaseTest;
import java.lang.reflect.Method;
import org.testng.annotations.Test;
import cat.gencat.mat.pages.GooglePage;
import org.testng.annotations.Parameters;
import app.getxray.xray.testng.annotations.XrayTest;
import app.getxray.xray.testng.annotations.Requirement;

public final class GoogleTest extends BaseTest {
  @Test @Parameters(value = {"browser"})
  @XrayTest(key = "...")     // Jira's `Test` key
  @Requirement(key = "...")  // Jira's `Story` key
  public void googleTest(String browser, Method method) throws Throwable {
    try {
      Utils.step("Enter website");
      Utils.gotoApp();
      Utils.maximize();
      Utils.anotate(Utils.LogLevel.PASS, "Website's homepage is accessible");
      Utils.screenshot("Homepage");

      // ... code continues ...

      Utils.step("Send search request");
      GooglePage.search("This is a search request");
      Utils.anotate(Utils.LogLevel.PASS, "Google search made successfully");
      Utils.screenshot("Google search results");

      // ... code continues ...

      Utils.endTestAsOK(browser, method);
    }
    catch (Exception | AssertionError e) { Utils.endTestAsKO(browser, method, e); }
  }
}
#+end_src






