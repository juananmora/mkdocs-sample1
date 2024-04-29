# MAT Functional Tests: Job

Use the MAT framework to execute functional tests on demand.

> The Git repo where your functional tests live NEEDS to be a fork of the official MAT template.

## Parameters

- `repositorio` :: URL where functional tests are hosted.
- `entorno` :: Environments to test in.
- `rama` :: Branch of Test Repository.
- `urlapp` :: Application URL.
- `umbral` :: Test Failed Threshold.

## Stages

1. `checkout-repo` :: Checkout the Git repo from within the `repositorio` URL + the `rama` branch.
2. `valueedge-convert-format` :: TBD.
3. `test` :: Execute the tests from within the project with Maven (using the `entorno` and `urlapp` params).
4. `valueedge-publish-results` :: TBD.
5. `thresholds-eval` :: Evaluate the ratio of succeeded-failed tests against the threshold specified by the `umbral` param.
6. `publish-report` :: Publish the HTML results report to Jenkins itself.
