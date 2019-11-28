# chart-ccd

[![Build Status](https://dev.azure.com/hmcts/CNP/_apis/build/status/Helm%20Charts/chart-ccd)](https://dev.azure.com/hmcts/CNP/_build/latest?definitionId=75)

* [Introduction](#introduction)
* [Configurable Variables](#Configurable-Variables)
* [Configuration](#Configuration)
    * [Demo default services](#Demo---default-services)
    * [Demo default services and frontend](#Demo---default-services-and-frontend)
    * [Demo default services, frontend and dependent services](#Demo---default-services-and-frontend-and-dependent-services)
    * [Preview - default services and frontend](#Preview---default-services-and-frontend)
* [Example Configuration](#Example-Configuration)    
* [Overriding existings services](#Override-Services)
    * [S2S Config](#S2S-Config)
* [Importers](#Importers)
* [Deployment on Preview](#Config-To-Deploy-on-Preview)
* [Access PR URL](#Accessing-an-app-using-this-chart-on-a-pull-request)
* [IDAM](#IDAM)
* [Local Testing](#Development-and-Testing)
* [Important Notes:](#Notes)
    * [DM Store and Blob Store](#DM-Store-and-Blob-Store)
* [General Information](#General-Information)


## Introduction

Helm product chart for Core Case Data

This chart installs Core Case Data (CCD) as a self contained product.
All dependent services can be deployed via configuration. By default the
main back-ends and services for the import of definitions are installed.
By default only one database is installed which will be shared between
CCD services and dependent services. This chart can be used to deploy on
various environments like Demo and Preview, but the configuration might
have to be tweaked to meet env specific requirements. This chart can be
used standalone or can be included in other product charts/applications.

Default Services:
* data store * - https://github.com/hmcts/ccd-data-store-api
* definition store * - https://github.com/hmcts/ccd-definition-store-api
* user profile * - https://github.com/hmcts/ccd-user-profile-api
* admin web * - https://github.com/hmcts/ccd-admin-web
* s2s - https://github.com/hmcts/service-auth-provider-app
* postgresql - https://github.com/helm/charts/tree/master/stable/postgresql
* definition importer - https://github.com/hmcts/ccd-docker-definition-importer
* user profile importer - https://github.com/hmcts/ccd-docker-user-profile-importer

Optional Services:
* case management web * -
  https://github.com/hmcts/ccd-case-management-web
* api gateway * - https://github.com/hmcts/ccd-api-gateway
* print service * - https://github.com/hmcts/ccd-case-print-service
* activity service * - https://github.com/hmcts/ccd-case-activity-api
* dm store - https://github.com/hmcts/document-management-store-app
* payment api - https://github.com/hmcts/ccpay-payment-app
* draft store - https://github.com/hmcts/draft-store
 
(*) services owned by CCD

## Configurable Variables

| Parameter                    | Description                     | Mandatory                       | Global     |
| --------------------------   | --------------------------------| ----------                      | ---------- |
| `idamWebUrl`                 | url of Idam Web                 | true                            | true       |
| `idamApiUrl`                 | url of Idam Api                 | true                            | true       |
| `ccdAdminWebIngress`         | url of CCD Admin Web            | true                            | true       |
| `ccdApiGatewayIngress`       | url of CCD API Gateway          | true when frontend enabled      | true       |
| `ccdCaseManagementWebIngress`| url of CCD Management Web       | true when frontend enabled      | true       |


## Configuration


### Demo - default services

```
    global:
      idamApiUrl: https://idam-api.demo.platform.hmcts.net
      idamWebUrl: https://idam-web-public.demo.platform.hmcts.net
      ccdAdminWebIngress: ccd-admin-{{ .Release.Name }}.demo.platform.hmcts.net

    ccd-admin-web:
      nodejs:
        ingressClass: traefik-no-proxy
        ingressHost: ccd-admin-{{ .Release.Name }}.demo.platform.hmcts.net
        secrets:
          IDAM_OAUTH2_AW_CLIENT_SECRET:
            secretRef: ccd-admin-web-oauth2-client-secret
            key: key
        environment:
          ADMINWEB_LOGIN_URL: '{{ .Values.global.idamWebUrl }}/login'
``` 

 
### Demo - default services and frontend 

```
    ccd:  
      managementWeb:
        enabled: true
      apiGatewayWeb:
        enabled: true

    global:
      idamApiUrl: https://idam-api.demo.platform.hmcts.net
      idamWebUrl: https://idam-web-public.demo.platform.hmcts.net
      ccdAdminWebIngress: ccd-admin-{{ .Release.Name }}.demo.platform.hmcts.net
      ccdApiGatewayIngress: gateway-{{ .Release.Name }}.demo.platform.hmcts.net
      ccdCaseManagementWebIngress: www-{{ .Release.Name }}.demo.platform.hmcts.net
      
    ccd-admin-web:
      nodejs:
        ingressClass: traefik-no-proxy
        ingressHost: ccd-admin-{{ .Release.Name }}.demo.platform.hmcts.net
        secrets:
          IDAM_OAUTH2_AW_CLIENT_SECRET:
            secretRef: ccd-admin-web-oauth2-client-secret
            key: key
        environment:
          ADMINWEB_LOGIN_URL: '{{ .Values.global.idamWebUrl }}/login'  
    ccd-case-management-web:
      nodejs:
        ingressHost: www-{{ .Release.Name }}.demo.platform.hmcts.net
    ccd-api-gateway-web:
      nodejs:
        ingressClass: traefik-no-proxy
        ingressHost: gateway-{{ .Release.Name }}.demo.platform.hmcts.net
        secrets:
          IDAM_OAUTH2_CLIENT_SECRET:
            secretRef: ccd-api-gateway-oauth2-client-secret
            key: key
    
``` 


### Demo - default services, frontend and dependent services 

```
    ccd:  
      managementWeb:
        enabled: true
      apiGatewayWeb:
        enabled: true
      emAnnotation:
        enabled: true
      ccpay:
        enabled: true
      draftStore:
        enabled: true
      dmStore:
        enabled: true
      activityApi:
        enabled: true
      blobstorage:
        enabled: true
      printService:
        enabled: true  

    global:
      idamApiUrl: https://idam-api.demo.platform.hmcts.net
      idamWebUrl: https://idam-web-public.demo.platform.hmcts.net
      ccdAdminWebIngress: ccd-admin-{{ .Release.Name }}.demo.platform.hmcts.net
      ccdApiGatewayIngress: gateway-{{ .Release.Name }}.demo.platform.hmcts.net
      ccdCaseManagementWebIngress: www-{{ .Release.Name }}.demo.platform.hmcts.net
      
    ccd-admin-web:
      nodejs:
        ingressClass: traefik-no-proxy
        ingressHost: ccd-admin-{{ .Release.Name }}.demo.platform.hmcts.net
        secrets:
          IDAM_OAUTH2_AW_CLIENT_SECRET:
            secretRef: ccd-admin-web-oauth2-client-secret
            key: key
        environment:
          ADMINWEB_LOGIN_URL: '{{ .Values.global.idamWebUrl }}/login'
    ccd-case-management-web:
      nodejs:
        ingressHost: www-{{ .Release.Name }}.demo.platform.hmcts.net
    ccd-api-gateway-web:
      nodejs:
        ingressClass: traefik-no-proxy
        ingressHost: gateway-{{ .Release.Name }}.demo.platform.hmcts.net
        secrets:
          IDAM_OAUTH2_CLIENT_SECRET:
            secretRef: ccd-api-gateway-oauth2-client-secret
            key: key
    ccd-case-activity-api:
      nodejs:
        environment:
          CORS_ORIGIN_WHITELIST: '{{ .Values.global.ccdCaseManagementWebIngress }}'
    dm-store:
      java:
        environment:
          MAX_FILE_SIZE: '100MB'      
          
``` 

#### Enabling upload history on Admin Web

To enable the history of definition uploads in CCD Admin Web,
[shown here](#Admin-Web-Definition-file-import), use the following
configuration. In the configurations above the upload history is
disabled:

```
    blobstorage:
        enabled: true
        
    ccd-definition-store-api:
      java:
        secrets:
          STORAGE_ACCOUNT_NAME:
            disabled: false
          STORAGE_ACCOUNT_KEY:
            disabled: false
        environment:
          AZURE_STORAGE_DEFINITION_UPLOAD_ENABLED: true
```

## Using your own Services

If you have any services already dependent in your chart, then you want
to override: eg.,

**S2S Config**
If you already have s2s dependencies in your own chart
Then Overrride with below environment variables with relevant s2s uri:
```
ccd:
  rpe-service-auth-provider:
    java:
      ingressHost: ""
      releaseNameOverride: "{{ .Release.Name }}-s2s"

```
And set below secrets to s2s installation:
```
rpe-service-auth-provider:
  java:
    MICROSERVICEKEYS_CCD_ADMIN: AAAAAAAAAAAAAAAA
      MICROSERVICEKEYS_CCD_DATA: AAAAAAAAAAAAAAAA
      MICROSERVICEKEYS_CCD_DEFINITION: AAAAAAAAAAAAAAAA
      MICROSERVICEKEYS_CCD_GW: AAAAAAAAAAAAAAAA
      MICROSERVICEKEYS_CCD_PS: AAAAAAAAAAAAAAAA
```    


## Setup users profiles and ccd definitions

There are two ways of setting up user profiles and importing definitions into CCD

a) Using Admin Web interface [see steps](#Admin-Web-Definition-file-import)

b) Using Importers 

### Importers

By default the chart will deploy some helper pods called importers.
These are used to import some initial definitions and setup user
profiles

* In the `cnp-flux-config` project, add additional user profiles to the `ccd-user-profile-importer` config in the file `/k8s/demo/common/ccd/latest-ccd-chart.yaml`:
    ```
    ccd-user-profile-importer:
            users:
              - auto.test.cnp@gmail.com|AUTOTEST1|AAT|TODO
              - <USER_ID>|<JURISDICTION>|<CASE_TYPE>|<CASE_STATE>
    ```

* In the `cnp-flux-config` project, add additional definition files to the `ccd-definition-importer` config in the file `/k8s/demo/common/ccd/latest-ccd-chart.yaml`:
  ```
  ccd-definition-importer:
        definitions:
          - https://github.com/hmcts/ccd-definition-store-api/raw/master/aat/src/resource/CCD_CNP_27.xlsx
          - <DEFINITIION_FILE_URL>
        userRoles:
          - caseworker-autotest1
  ```

## Accessing an app using this chart on a pull request

DNS will be automatically registered for most of the CCD pods, the ccd component will be prefixed to the regular url,
The prefixes can be found here:
https://github.com/hmcts/chart-ccd/blob/master/ccd/templates/ingress.yaml#L23

Note: To access these URLs you need to run F5 - VPN and enable FoxyProxy in your browser 

An example url for accessing case management web would be:
```
https://case-management-web-sscs-cor-backend-pr-189.service.core-compute-preview.internal/
```

### IDAM 
**Whitelisting for web components**
Managed using https://github.com/hmcts/chart-idam-pr (version 2.0.0 and above).

To enable add following to your values.preview.yaml:
```
tags:
  ccd-idam-pr: true

ccd:
  # other ccd config

  idam-pr:
    releaseNameOverride: ${SERVICE_NAME}-ccd-idam-pr
    redirect_uris:
      CCD:
        - https://case-management-web-${SERVICE_FQDN}/oauth2redirect
      CCD Admin:
        - https://admin-web-${SERVICE_FQDN}/oauth2redirect
```



## Development and Testing

Default configuration (e.g. default image and ingress host) is setup for sandbox. This is suitable for local development and testing.

- Ensure you have logged in with `az cli` and are using `sandbox` subscription (use `az account show` to display the current one).
- For local development see the `Makefile` for available targets.
- To execute an end-to-end build, deploy and test run `make`.
- to clean up deployed releases, charts, test pods and local charts, run `make clean`

`helm test` will deploy a busybox container alongside the release which performs a simple HTTP request against the service health endpoint. If it doesn't return `HTTP 200` the test will fail. **NOTE:** it does NOT run with `--cleanup` so the test pod will be available for inspection.

### Testing inside another chart locally

You can easily include this chart in another chart for testing:

requirements.yaml
```
  - name: ccd
    version: '>1.0.0'
    repository: file://<path-to-repository-can-be-relative-or-absolute>/chart-ccd/ccd
```

## Azure DevOps Builds
Builds are run against the 'nonprod' AKS cluster.

### Pull Request Validation
A build is triggered when pull requests are created. This build will run `helm lint`, deploy the chart using `ci-values.yaml` and run `helm test`.

### Release Build
Triggered when the repository is tagged (e.g. when a release is created). Also performs linting and testing, and will publish the chart to ACR on success.



 
## Notes

**DM Store and Blob Store**
 By default dm store and blob store are disabled. 
 But dm store needs blobstore. 
 So in case your chart needs dm store then don't forget to  enable blob store.

 Default config is as follows
```
 ccd:
  dmStore:
    enabled: false
  blobstorage:
    enabled: false
```
 If users want to see the history of uploads on Admin Web, then they need to enable blobstore and enable azure_store upload flag in definition-store as below:

 ```
 ccd:
  blobstorage:
    enabled: true
 ```   
And
 ```
 ccd-definition-store-api:
   environment:
     AZURE_STORAGE_DEFINITION_UPLOAD_ENABLED: true
```     

## Admin Web Definition file import

* Open the Admin Web in a web browser, and login with the test credential.
* Click on `Import Case Definition` to navigate to the importer section.

![Admin web import](/images/import_home.png)

* Click on the `Choose file` button and select a definition file from the file menu.

![File menu](/images/file_menu.png)

![File chosen](/images/file_chosen.png)

* Press the `Submit` button.
* The message `Case Definition data successfully imported` is displayed if the definition file is successfully imported.
* A record is added to the import audit table.

![Import successful](/images/file_imported.png)

## Admin Web Create User Profile

* In admin web, click on the `Manage User Profiles` link.

![Profile home](/images/admin_web_home.png)

* Select a jurisdiction.
* Click on the `Submit` button.

![Profile home](/images/select_jurisdiction.png)

* Click the `Create User` button 

![Profile home](/images/create_user_button.png)

* Enter the `User idam ID`. Select the default jurisdiction, case type and case state.
* Click the `Create` button
* Verify that the user profile has been created.

![Profile home](/images/create_user_profile.png)

## General Information
The following table lists the configurable parameters which comes from base chart . chart-java.
This is just for information, as chart-ccd already set necessary attributes
If in case need to increase due to performance issues, these params can be modified in your chart

| Parameter                  | Description                                | Default  |
| -------------------------- | ------------------------------------------ | ----- |
| `appInsightsKey`           | Application insights key for full CCD stack | `fake-key`|
| `memoryRequests`           | Requests for memory | `512Mi` **|
| `cpuRequests`              | Requests for cpu | `250m` **|
| `memoryLimits`             | Memory limits| `2048Mi` **|
| `cpuLimits`                | CPU limits | `1500m` **|
| `ingressHost`              | Host for ingress controller to map the container to | `nil` (required, provided by the pipeline)  **|
| `ingressIP`              | Ingress controllers IP address | `nil` (required, provided by the pipeline)  **|
| `consulIP`              | Consul servers IP address | `nil` (required, provided by the pipeline) **|
| `readinessPath`            | Path of HTTP readiness probe | `/health` **|
| `readinessDelay`           | Readiness probe inital delay (seconds)| `30` **|
| `readinessTimeout`         | Readiness probe timeout (seconds)| `3` **|
| `readinessPeriod`          | Readiness probe period (seconds) | `15` **|
| `livenessPath`             | Path of HTTP liveness probe | `/health` **|
| `livenessDelay`            | Liveness probe inital delay (seconds)  | `30` **|
| `livenessTimeout`          | Liveness probe timeout (seconds) | `3` **|
| `livenessPeriod`           | Liveness probe period (seconds) | `15` **|
| `livenessFailureThreshold` | Liveness failure threshold | `3`  **|
