# Example NodeJS Provider - Dredd

<!-- Build Badge -->

[![Build Status](https://github.com/pactflow/example-bi-directional-provider-dredd/actions/workflows/build.yml/badge.svg)](https://github.com/pactflow/example-bi-directional-provider-dredd/actions)

<!-- Can I Deploy Badge -->

[![Can I deploy Status](https://testdemo.pactflow.io/pacticipants/pactflow-example-bi-directional-provider-dredd/branches/master/latest-version/can-i-deploy/to-environment/production/badge)](https://testdemo.pactflow.io/pacticipants/pactflow-example-bi-directional-provider-dredd/branches/master/latest-version/can-i-deploy/to-environment/production/badge)

- [Example NodeJS Provider - Dredd](#example-nodejs-provider---dredd)
  - [Overview of Example](#overview-of-example)
    - [Key points](#key-points)
  - [Overview of Part of Bi-Directional Contract Testing Flow](#overview-of-part-of-bi-directional-contract-testing-flow)
  - [Compatibile with Consumers](#compatibile-with-consumers)
  - [Pre-requisites](#pre-requisites)
    - [Environment variables](#environment-variables)
  - [Usage](#usage)
    - [Steps](#steps)
  - [OS/Platform specific considerations](#osplatform-specific-considerations)
    - [Windows](#windows)
  - [Caveats](#caveats)
  - [Related topics / posts / discussions](#related-topics--posts--discussions)
  - [Other examples of how to do this form of testing](#other-examples-of-how-to-do-this-form-of-testing)
  - [Found an issue?](#found-an-issue)

## Overview of Example

<!-- Provider Overview -->

This is an example of a NodeJS "Product" API Provider that uses Dredd, Pact, [Pactflow](https://pactflow.io) and GitHub Actions to generate and publish Pact provider contracts.

It performs pre-deployment cross-compatability checks to ensure that it is compatible with specified consumers using the Bi-Directional contract capability of Pactflow.

<!-- General -->

See the full [Pactflow Bi-Directional Workshop](https://docs.pactflow.io/docs/workshops/bi-directional-contract-testing) for which this can be substituted in as the "provider".


### Key points

It:

* Is an API written in Express JS
* Has OAS 3.0 spec documenting the API
* Uses Dredd for API testing to check spec compliance

What is uploaded to Pactflow is an OpenAPI specification that represents what you actually tested with Dredd, to give us confidence it is compatible with a Pact consumer.

## Overview of Part of Bi-Directional Contract Testing Flow

<!-- Provider Overview -->

In the following diagram, you can see how the provider testing process works.

When we call "can-i-deploy" the cross-contract validation process kicks off on Pactflow, to ensure any consumer consumes a valid subset of the OAS for the provider.

![Provider Test](docs/provider-scope.png "Provider Test")

The project uses a Makefile to simulate a very simple build pipeline with two stages - test and deploy.

When you run the CI pipeline (see below for doing this), the pipeline should perform the following activities (simplified):

* Test
  * Run tests to check spec compliance with openAPI spec
  * Create branch tag via Pact CLI
  * Publish openAPI spec, along with a version with the name of the current branch
  * Check if we are safe to deploy to Production with `can-i-deploy` (ie. has the cross-contract validation has been successfully performed)
* Deploy (only from master)
  * Deploy app to Production
  * Record the Production deployment in the Pact Broker


![Provider Pipeline](docs/provider-pipeline.png "Provider Pipeline")


## Compatibile with Consumers

<!-- Consumer Compatability -->

This project is currently compatible with the following consumers(s):

* [pactflow-example-bi-directional-consumer-nock](https://github.com/pactflow/example-bi-directional-consumer-nock)
* [pactflow-example-bi-directional-consumer-msw](https://github.com/pactflow/example-bi-directional-consumer-msw)
* [pactflow-example-bi-directional-consumer-wiremock](https://github.com/pactflow/example-bi-directional-consumer-wiremock)
* [pactflow-example-bi-directional-consumer-mountebank](https://github.com/pactflow/example-bi-directional-consumer-mountebank)
<!-- * [pactflow-example-bi-directional-consumer-dotnet](https://github.com/pactflow/example-bi-directional-consumer-dotnet) -->

See [Environment variables](#environment-variables) on how to set these up

## Pre-requisites

**Software**:

- Tools listed at: https://docs.pactflow.io/docs/workshops/ci-cd/set-up-ci/prerequisites/
- A pactflow.io account with an valid [API token](https://docs.pactflow.io/docs/getting-started/#configuring-your-api-token)

### Environment variables

To be able to run some of the commands locally, you will need to export the following environment variables into your shell:

- `PACT_BROKER_TOKEN`: a valid [API token](https://docs.pactflow.io/docs/getting-started/#configuring-your-api-token) for Pactflow
- `PACT_BROKER_BASE_URL`: a fully qualified domain name with protocol to your pact broker e.g. https://testdemo.pactflow.io

<!-- CONSUMER env vars -->

Set `PACT_PROVIDER` to one of the following

- `PACT_PROVIDER=pactflow-example-bi-directional-provider-dredd`: Dredd - (https://github.com/pactflow/example-bi-directional-provider-dredd)
- `PACT_PROVIDER=pactflow-example-bi-directional-provider-postman`: Postman - (https://github.com/pactflow/example-bi-directional-provider-postman)
- `PACT_PROVIDER=pactflow-example-bi-directional-provider-restassured`:  Rest Assured - (https://github.com/pactflow/example-bi-directional-provider-restassured)
  
## Usage

### Steps

* `make test` - run the tests locally
* `make fake_ci` - run the CI process, but locally

## OS/Platform specific considerations

The makefile is configured to run on Unix based systems such as you would find in most common CI/CD pipelines. 

They can be run locally on Unix/Mac, or on Windows via [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install). 

### Windows 

You can still try this example locally on Windows using powershell and running commands manually. 

<details>
  <summary>Click to see windows specific instructions here</summary>


  These will be the same commands that are used in the makefile with a few manual tweaks.

  1. Make sure you have set all of the environment variables, in powershell they can be set like so.

    ```
     $env:GIT_BRANCH="main"
    ```

  2. Publish the pact that was generated. The step uses the pact-cli docker image to publish the pact to your pactflow account.
  The path for `<path_to_project_root>` needs to be converted from Windows paths to UNIX ones as the Docker container is using UNIX. Either hard code this or set it as another environment variable.

      `C:\Users\Person\Documents\example-bi-directional-consumer-dotnet` 
      
      becomes
      
      `/c/Users/Candy/Documents/Pactflow/example-bi-directional-consumer-dotnet`

      $env:VARIABLE_NAME refers to the environment variables in windows.

      ```
      docker run --rm -v <path_to_project_root>:<path_to_project_root> -e PACT_BROKER_BASE_URL -e PACT_BROKER_TOKEN pactfoundation/pact-cli publish <path_to_pacts_folder> --consumer-app-version $env:GIT_COMMIT --tag $env:GIT_BRANCH

      ```

  4. Check can-i-deploy to see if your provider is compatible with your pact.

      ```
      docker run --rm -v <path_to_project_root>:<path_to_project_root> -e PACT_BROKER_BASE_URL -e PACT_BROKER_TOKEN pactfoundation/pact-cli  broker can-i-deploy --pacticipant pactflow-example-bi-directional-consumer-dotnet --version $env:GIT_COMMIT --to-environment production  --retry-while-unknown 0 --retry-interval 10
      ```

5. Have a look at what other commands are available in the Makefile. All of them can be ran locally from Powershell by changing the windows paths to UNIX and replacing the environment variable references. Any variable referenced as `${VARIABLE}` can be changed to `$env:VARIABLE` to reference environment variables in Powershell.

</details>

## Caveats

- [OAS considerations](https://docs.pactflow.io/docs/bi-directional-contract-testing/contracts/oas#considerations)
-  you are responsible for ensuring sufficient OAS coverage. To highlight this point, in our example, we do _not_ test the 404 case on the provider, but the consumer has a pact for it and it's tests still pass! _NOTE: We plan to address this problem in the future_
- _implementing_ a spec is not the same as being _compatible_ with a spec. Most tools only tell you that what you’re doing is _not incompatible_ with the spec. _NOTE: We plan to address this problem in the future_
  
## Related topics / posts / discussions

- [Consumer Side Bi-Directional Contract Testing Guide](https://docs.pactflow.io/docs/bi-directional-contract-testing/consumer)
- [Provider Side Bi-Directional Contract Testing Guide](https://docs.pactflow.io/docs/bi-directional-contract-testing/provider)

## Other examples of how to do this form of testing

- TBC
  
## Found an issue?

Reach out via a GitHub Issue, or reach us over in the [Pact foundation Slack](https://slack.pact.io)



Presentation:
Pact Demo

Intro
So here we have a very basic site frontend and backend. We’ve got a Master branch, a dev branch we’ve created from it, and a feature branch title ‘delete-product’ on both repos. 
We’re going to start up both of them.
Run ‘npm start’ for both of them
A front end catalog, and a product detail page.
While it all seems to be working, we’d like contract test the relationship between these two services. Lets start, as one should, with the backend.

Backend Section
To test the backend provider all we need to do is make sure that the OpenAPI spec we have written is fulfilled as expected by the service. 
I need to meet with Holly about our current usage of API testing tools (and change this repo to match it) but when developing this I picked Dredd.
We’re going to run make test which will run Dredd against our OAS defined here. 
Run ‘make test’
Once it runs, the results will be outputted to output/report.md. This is the document that’ll be uploaded to Pact
We know the oas is green, so lets mimic the CI process in our terminal before we see it run in Github. It’s going to run the tests again, publish the results, check can-i-deploy (which will pass, since there is no existing consumer YET), then mark it as ‘deployed to production’
See the Pactflow entry for it has appeared.

Frontend Section
Now, lets look at doing the same for our frontend.
For frontend testing things are a little more involved, we need to create Cypress tests that mock the provider for each endpoint we tested in the OAS. We do this by using the Pact Cypress adapter.
Shows the cypress tests under cypress/integrations/
Now were going to run these tests.
Run ‘make test’
Now that the tests all pass we know its working on this end, now we just have to make sure that the resultant pact file from this, viewable here:
See cypress/pacts
…Is compatible with the report.md we posted to Pactflow. So we’ll do what we did before and mimic the deployment pipeline.
THis will run the tests, publish the results, check can-i-deploy then record a deployment to production if alls green.
Run make fake_ci
We can see in Pact now that this contract is valid and fullfilled. 
Were going to very quickly do that again for dev, but we’ll do by running the pipeline in Github Actions (which triggers on a commit aswell)

Feature branch
So thats the process on a working provider and consumers, let imagine were adding a new feature branch that will allow us to delete products. As would be the case in development, lets work from the backend forward. Here’s the backend branch, you can see we’ve added a new entry to the OAS and a new API route. Let me start the service and show the functionality
Run “npm start” then use the delete product Postman request
Now, as before we always want to start with testing this ourselves locally. 
Run ‘make test’
All the tests are passing, so lets commit this to our branch. This will start the pipeline for it and will can-i-deploy check it against the Dev consumer (but wont record a deployment, since its a feature branch)
Show makefile
While thats running, lets check the consumer equivalent

Feature Consumer branch
Here we’ve added an extra section to the product page cypress test to accomodate our new functionality. Let’s run our backend  and front end see how its supposed to work.
Run npm start for both
Now we run the tests and hope they pass. Which they do! Now let’s make a commit to this branch. Like the provider it will run the test, publish results, then can-i-deploy. Howeber, this time it will fail since its testing against the dev provider, which lacks the functionality its expecting. To test against the OTHER feature branch we run:
First lets look at what wiill hapen in the deployment we just made. Run make can_i_deploy. it will fail
Now run make ‘dev_can_i_deploy” which will pass
Now all we need to do is merge the provider delete-product into dev, wait for it to record deployment, THEN merge in the consumer delete-product.