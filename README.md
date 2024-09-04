# hackathon
We already provide a working demo serving-template in the `demo-team` folder. Feel free to use this as a template.

## Setup
1. We use [Bruno](https://www.usebruno.com/) as the API Client to communicate with SAP AI Core. 
2. You should have [Python](https://www.python.org/downloads/) installed
3. We provide a Bruno Collection and a partially preconfigured Bruno environment file for you to access AI Core
4. You should have [Docker]() installed.
5. At least one person in your team should have a [DockerHub](https://hub.docker.com/) account
6. You will train your ML Models locally on your own machine
7. Follow the guide in this README and in the following example repository: https://github.com/ironedr/hackathon-demo

## How to Authenticate to the SAP AI-Core API:
1. Set the client-id and client-secret in the hackathon bruno environment (if not already set)
2. Use the "Authenticate" POST request in the bruno collection

## How to develop the ML Services:
1. There will be 2 GitHub repositories: a) the repository containing the ml source code that the team creates. b) our hackathon repository where they provide the serving-template for their own ml-service
2. The teams should follow the guidelines in this repository: https://github.com/ironedr/hackathon-demo to create their own source-code repository and start development
2. After writing their API and building a docker-image, each team should provide a serving-template by forking the following repository and creating a PR request:
https://github.com/EkinTiras/hackathon/tree/main

## How to start running an ML Service on AI Core:
In SAP AI Core, ML-Services that provide an API are called Deployments. The following steps are necessary to create a deployment for your ML-app:
1. Prerequisites:
    - The app-code is ready
    - the docker image has been pushed to dockerhub
    - the team's `serving-template.yaml` is correct and has been merged into this repository
    - the teams have configured their `resource_group_id` and `application_name` in the bruno environment config
    - set the `scenario_id` value in your bruno environment (each team has defined this in their serving-template in the field:
        ```yaml
        labels:
          scenarios.ai.sap.com/id: <scenario-id>
        ```
    - set the value for the field `execution_id` in the bruno environment:
        ```yaml
        metadata:
          name: <your-executable-id>
        ```
2. set a configuration name for your team in the `config_name` field in the bruno environment
3. Select the `Create Configuration` request in your Bruno Collection
    - Configure environment variables (for example hyper parameters) that your service needs to run. (If you don't need any just set the name, `executableId`, `scenarioId` and `name`)
      ```json
      {
        "name": "{{config_name}}",
        "executableId": "{{executable_id}}",
        "scenarioId": "{{scenario_id}}",
        "your_env_variable": "<provide_your_value>"
      }
      ```
5. Send request. The configuration will be created for you and a configurationId will be returned. We have a bruno script that automatically copies this value to the `configuration_id` field in your bruno environment
6. Create the deployment by sending the request. This is how your request body should look like:
    ```json
    {
      "configurationId": "{{configuration_id}}"
    }
    ```
7. The `deployment-id` is automatically stored in the bruno environment.
8. Check the state of your deployment by running the "Get information about specific deployment" request.
9. (Optional) Debugging: use the `Get logs for specific deployment` request to search for clues in case of errors.
10. Do not create more than 1 deployment. If you need to create a new one, make sure to the delete the old one first


## How to delete deployments:
1. Select the PATCH request to `Update the target status of a deployment` and in the body, set the targetStatus to `STOPPED`:
    ```json
    {
      "targetStatus": "STOPPED"
    }
    ```
2. This operation will be scheduled but not executed immediately. Check the status of your deployment periodically with the `Get information about specific deploymen` request. Once the `status` field is set to `STOPPED` continue with the next step.
3. Execute the "Mark deployment as deleted" request. Now you can go ahead and create a new deployment you want to.


## How to use your deployment:
1. Specify the value for the `endpoint` field in your bruno environment (that's the endpoint your flask server is offering but do not include the `/v2/` part e.g. `/v2/classify` becomes `classify` or `/v2/predict` becomes `predict`)
2. Select the `Send request to your deployment` request and configure the request body. The data you put here, is the one that your flask application expects.
3. Send the request
