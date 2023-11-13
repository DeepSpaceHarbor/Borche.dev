title: Run Cypress tests in Google Cloud Build
summary: Real world example of Cypress tests running in Google Cloud Build.
tags: []
categories: []
featured_image: files/images/featured/google-cloud.jpg
date: '2022-04-15T22:04:38.000Z'
---
On a recent project I got the task of setting up end-to-end tests using Cypress. The project is hosted entirely on the Google cloud platform and has unit tests running as Cloud Build check. We decided to follow the same method for the Cypress tests.
There are two different approaches for this, you could set up a custom docker file and run the tests inside the container, or use the [official Cypress docker image](https://hub.docker.com/u/cypress) as part of the Cloud Build step.
[Gleb Bahmutov](https://github.com/bahmutov/google-cloud-build-example) and [Applitools](https://applitools.com/blog/google-cloud-build/) have already shared code and tutorials on the custom docker file approach. 
I will show you a slightly different way to achieve the same result.  
To keep things simple, let's create a new project and add Cypress.
```bash
$ yarn init
$ yarn add cypress
```
This is the initial Cloud Build config that will install dependencies and run the tests.
```yaml
steps:
  ###
  # Install dependencies
  #
  - name: 'gcr.io/cloud-builders/yarn'
    id: install-dependencies
    entrypoint: 'yarn'
    args:
      - 'install'
  ###
  # Run cypress tests
  #
  - name: 'cypress/base'
    waitFor: 
      - install-dependencies
    id: run-tests
    entrypoint: 'bash'
    args:
      - '-c'
      - 'yarn cypress run'
```
Go ahead and trigger the Cloud Build check. At this stage, everything should be successful 🥳.
The next step is to collect the artifacts (videos, screenshots, test reports, etc) produced by Cypress.
This is done by capturing the exit status code of the Cypress test runner and saving it to a file. In a separate step we can handle the artifacts upload, and exit with the status code that came from the Cypress test runner.
```yaml
steps:
  ###
  # Install dependencies
  #
  - name: 'gcr.io/cloud-builders/yarn'
    id: install-dependencies
    entrypoint: 'yarn'
    args:
      - 'install'
  ###
  # Run cypress tests
  #
  - name: 'cypress/base'
    waitFor: 
      - install-dependencies
    id: run-tests
    entrypoint: 'bash'
    args:
      - '-c'
      -  yarn cypress run;
         echo $? > .result;
         echo "Testing is completed!"
         
  ###
  # Save artifacts
  #
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    id: save-artifacts
    waitFor:
      - run-tests
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        echo "Uploading artifacts..."
        exit $(< .result);
```

The reason for modifying the exit code of these steps is because the status of a single Cloud Build step is determined by the exit code of the last executed command. And we know that if the previous step fails, the build execution will stop.
If we don’ modify the exit code, a failed test will stop the build execution right before the save-artifacts step is executed and leave us without the much-needed data to debug the failure.
If we tried to save artifacts in the run-tests step, the last executed command would be saving artifacts. This command always succeeds and gives us the green status, even when tests have failed!