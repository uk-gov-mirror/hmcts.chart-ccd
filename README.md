# chart-ccd

[![Build Status](https://dev.azure.com/hmcts/CNP/_apis/build/status/Helm%20Charts/chart-ccd)](https://dev.azure.com/hmcts/CNP/_build/latest?definitionId=75)

* [Introduction](#introduction)
* [Configurable Variables](#Configurable-Variables)
* [Configuration](#Configuration)
    * [Demo default services](#Demo---default-services)
    * [Demo default services and frontend](#Demo---default-services-and-frontend)
  *   [Demo default services, frontend and dependent services](#Demo---default-services-and-frontend-and-dependent-services)
* [Importers](#Importers)
* [Access PR URL](#Accessing-an-app-using-this-chart-on-a-pull-request)
* [IDAM](#IDAM)
* [Local Testing](#Development-and-Testing)
* [Important Notes:](#Notes)
    * [DM Store and Blob Store](#DM-Store-and-Blob-Store)
* [General Information](#General-Information)


## Introduction

Helm product chart for Core Case Data (CCD)

This chart installs CCD as a self contained product. By default only one
database is installed which will be shared between CCD services and
dependent services.

Features:
 
* back-ends only deployment (default)
* back-ends and front-ends deployment
* back-ends, front-ends and dependent services deployment
* can be used standalone or included in other charts
* supports deploy on multiple environment e.g. Demo, Preview 
* supports deploy of multiple instances of CCD under the same namespace
  or different namespaces 
* out of the box user profiles and definition setup


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
| `devMode`                    | CCD backend apps require APPINSIGHTS_INSTRUMENTATIONKEY configuration property. Setting devMode to true provides this transparently.| true | true       |


## Configuration

### Demo - secrets

When deploying on a vault-less environment like Demo, secrets must be
provided in the form of sealed secrets in the flux config. For
reference, the sealed secrets required by this chart can be found at
https://github.com/hmcts/cnp-flux-config/tree/master/k8s/demo/common/ccd/sealed-secrets

To use this chart on a certain namespace, all the required sealed
secrets must be separately provided. It's currently not possible to just
copy the CCD ones because the namespace we used to generate them is part
of the encryption (that might change in the future).

CCD sealed secrets can be generated by using the following script:  
https://github.com/hmcts/cnp-flux-config/blob/master/k8s/demo/common/ccd/bin/vault-to-sealedsecret.sh


### Demo - default services

```
    global:
      idamApiUrl: https://idam-api.demo.platform.hmcts.net
      idamWebUrl: https://idam-web-public.demo.platform.hmcts.net
      ccdAdminWebIngress: ccd-admin-{{ .Release.Name }}.demo.platform.hmcts.net
      devMode: true

    ccd-admin-web:
      nodejs:
        ingressClass: traefik-no-proxy
        ingressHost: ccd-admin-{{ .Release.Name }}.demo.platform.hmcts.net
        secrets:
          IDAM_OAUTH2_AW_CLIENT_SECRET:
            secretRef: ccd-admin-web-oauth2-client-secret
            key: key
    #importers are enabled by default. Make sure you properly configure them or else explicitly disable them           
    ccd-user-profile-importer:
      users:
       - auto.test.cnp@gmail.com|AUTOTEST1|AAT|TODO
    ccd-definition-importer:
      definitions:
       - https://github.com/hmcts/ccd-definition-store-api/raw/master/aat/src/resource/CCD_CNP_27.xlsx
      userRoles:
       - caseworker-autotest1
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
      devMode: true
      
    ccd-admin-web:
      nodejs:
        ingressClass: traefik-no-proxy
        ingressHost: ccd-admin-{{ .Release.Name }}.demo.platform.hmcts.net
        secrets:
          IDAM_OAUTH2_AW_CLIENT_SECRET:
            secretRef: ccd-admin-web-oauth2-client-secret
            key: key  
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
    #importers are enabled by default. Make sure you properly configure them or else explicitly disable them           
    ccd-user-profile-importer:
      users:
       - auto.test.cnp@gmail.com|AUTOTEST1|AAT|TODO
    ccd-definition-importer:
      definitions:
       - https://github.com/hmcts/ccd-definition-store-api/raw/master/aat/src/resource/CCD_CNP_27.xlsx
      userRoles:
       - caseworker-autotest1
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
      devMode: true
      
    ccd-admin-web:
      nodejs:
        ingressClass: traefik-no-proxy
        ingressHost: ccd-admin-{{ .Release.Name }}.demo.platform.hmcts.net
        secrets:
          IDAM_OAUTH2_AW_CLIENT_SECRET:
            secretRef: ccd-admin-web-oauth2-client-secret
            key: key
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
    #importers are enabled by default. Make sure you properly configure them or else explicitly disable them           
    ccd-user-profile-importer:
      users:
       - auto.test.cnp@gmail.com|AUTOTEST1|AAT|TODO
    ccd-definition-importer:
      definitions:
       - https://github.com/hmcts/ccd-definition-store-api/raw/master/aat/src/resource/CCD_CNP_27.xlsx
      userRoles:
       - caseworker-autotest1     
          
``` 

#### Enabling upload history on Admin Web

In the configurations above the history of definition uploads in CCD
Admin Web, [shown here](#Admin-Web-Definition-file-import), is disabled.
To enable it, use the following configuration:

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

Note: blobstorage is used also for other purposes so it might already be
enabled on some configs

## Setup user profiles and ccd definitions

This chart provides two ways of setting up user profiles and importing
definitions into CCD

* CCD Admin Web [see steps](#Admin-Web-Definition-file-import)

* Importers 

While the importers are useful to create some initial desired setup at
deploy time, the Admin Web allows to import additional definitions later
on

### Importers

By default the chart will deploy some helper pods called importers.
These are used to setup specified definitions and user profiles at
deploy time. Make sure you properly configure them or else disable them.

* user profile importer setup example:
    ```
    ccd-user-profile-importer:
        users:
          - <USER_ID>|<JURISDICTION>|<CASE_TYPE>|<CASE_STATE>
    ```

* ccd definition importer setup example:
    ```
    ccd-definition-importer:
        definitions:
          - <DEFINITIION_FILE_URL>
        userRoles:
          - <USER_ROLES>
    ```
  
For more advanced configuration refer to the importers documentation:
- https://github.com/hmcts/ccd-docker-definition-importer
- https://github.com/hmcts/ccd-docker-user-profile-importer
  
The configuration examples in this guide will import a simple test
definition and setup a test user 'auto.test.cnp@gmail.com':

```
ccd-user-profile-importer:
    users:
      - auto.test.cnp@gmail.com|AUTOTEST1|AAT|TODO
```

```
ccd-definition-importer:
    definitions:
      - https://github.com/hmcts/ccd-definition-store-api/raw/master/aat/src/resource/CCD_CNP_27.xlsx
    userRoles:
      - caseworker-autotest1
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

## Ready for take-off 🛫

You can log in to CCD UI at the following url:

    www-{{ .Release.Name }}.demo.platform.hmcts.net
    
Ask the CCD team for the default test user credentials if you want to do
a quick sanity check that the installation is successful by using the
test case type installed by default
