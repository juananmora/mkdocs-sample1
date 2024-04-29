# MAT Functional Tests: Job

Use the MAT framework to execute functional tests on demand.

> The Git repo where your functional tests live NEEDS to be a fork of the official MAT template.

## Parameters

- `repositorio` :: URL where functional tests are hosted.
- `entorno` :: Environments to test in.
- `rama` :: Branch of Test Repository.
- `urlapp` :: Application URL.
- `umbral` :: Test Failed Threshold.
- `quality_gate` :: Enable or disable quality gate
- `JIRA_PROJECT_KEY` :: Project key of the project in Jira
- `JIRA_ISSUE_KEY` :: Test plan key in Jira.

## Diagram

                      Start
                        |
                        v
                 Check Parameters
                        |
                        v
                   Checkout Repo
                        |
                        v
                    Run Tests
                        |
                        v
             Jira Upload Results
                        |
                        v
           Jenkins Publish Report
                        |
                        v
            Jira Upload Report
                        |
                        v
              Thresholds Evaluation
                        |
                        v
                 End of Pipeline

## Stages

The main pipeline is executed within a podTemplate that uses a Maven container. The pipeline flow is as follows:

1. `checkout-repo` :: Checkout the Git repo from within the `repositorio` URL + the `rama` branch.
2. `run-test` :: Tests are executed using Maven. The previously defined parameters are passed to Maven as variables. (`entorno`, `rama`, `jira_project_key`, `jira_issue_key` ) 
3. `jira-uploade-result` :: Test results are uploaded to Jira.
4. `jenkins-publish-report` :: An HTML report is published from a specific directory..
5. `'jira-upload-report` :: A ZIP file containing the report is uploaded to Jira.
6. `thresholds-eval` :: Evaluate the ratio of succeeded-failed tests against the threshold specified by the `umbral` param.

## Jira functions. Integration Jenkins - Jira

Functions provide mechanisms to integrate Jenkins with Jira and maintain a record of the pipeline's state.

- `run_jira_cmd` :: Executes a block of code only if certain Jira parameters are provided.
- `jira_add_comment` :: Adds a comment to a Jira issue.
- `jira_upload_results` :: Uploads test results to Jira.
- `jira_upload_report` :: Uploads a ZIP file as an attachment to Jira.
