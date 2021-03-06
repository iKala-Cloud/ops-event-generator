# Ops Event Generator (Demeter)

[![demeter-docker-image-ci](https://github.com/iKala-Cloud/ops-event-generator/actions/workflows/docker-image.yml/badge.svg)](https://github.com/iKala-Cloud/ops-event-generator/actions/workflows/docker-image.yml)

This project is for iKala AIOps project - Event Generator part.

## Detail

[Project description](https://gcp.corp.ikala.tv/projects/gcp/wiki/Progress_MVP)

![](img/arch_eg.png)

## Quickstart

### Before you start install

1. Please make sure you had register in ER run. 
2. Please make sure you have the service account of your EG project be added to cloud-tech-dev-2021.
### Install

1. Open a cloud shell in your GCP console
2. `source <(curl -s curl -sL https://raw.githubusercontent.com/iKala-Cloud/ops-event-generator/master/install.sh)`

## Architecture

```shell
.
├── publisher.py
├── subscriber.py
├── Dockerfile
└── main.py
```


> NOTICE: The best practice is to avoid upload key-paris to third-party repository which is risky to key leaking.

## Permission

The roles for service account that contain permissions to receive message from eventarc and invoke Cloud Run service.

![](img/eg_roles_20211001.png)

## Cloud Run
### Deployment

1. Deploy from source using the following command:

```shell
$ gcloud config set project customer-a-project-1

$ gcloud run deploy demeter --region=asia-east1 --set-env-vars TOPIC_ID=<ER PUB/SUB TOPIC ID FOR EG PROJECT> --set-env-vars CUSTOMER=<CUSTOMER NAME> --source=.
```
Then wait a few moments until the deployment is complete. On success, the command line displays the service URL.

2. Visit your deployed service by opening the service URL in a web browser.

**NOTICE**
[New project first deploy EG CR]

1. EG 第一次 deploy 需要到 console Enable cloud build API，已切換 service account credential的 gcloud 會一直無法

2. Deploy 完畢要用 /connect 建立 eventarc trigger 出現：
    ```
    ERROR: (gcloud.eventarc.triggers.create) PERMISSION_DENIED: Permission "eventarc.events.receiveAuditLogWritten" denied on "1097778675156-compute@developer.gserviceaccount.com"
    ```
    但是 service account 已經是 Editor

    Benson 表示 default service account 需要 **Eventarc Event Receiver**，Editor role 確實沒有這個，加上去後就成功呼叫 /connect 來建立 eventarc trigger 了

### Test

To test API, you can use `curl` to access via service endpoint:

```shell
$ curl -H \
"Authorization: Bearer $(gcloud auth print-identity-token)" \
https://demeter-i7wsmqzjha-de.a.run.app
```

**View Eventarc messages in Cloud Run when IAM policy changed**
![](img/eg_result_20211001.png)

## Reference

- [Cloud Pub/Sub - Using client libraries](https://cloud.google.com/pubsub/docs/quickstart-client-libraries#pubsub-client-libraries-python)
- [Cloud Run - Build and deploy (Python)](https://cloud.google.com/run/docs/quickstarts/build-and-deploy/python)

## CI/CD
Currently, we use GitHub Action to build the demeter image and push to Google Artifact Registry. The configuration of GitHub Action is **[.github/workflows/docker-image.yml](.github/workflows/docker-image.yml)**.

Here we setup which branches should trigger a build. Currently, we only build any push commit or merge commit of branch **master**.
```yaml
on:
  push:
    branches: [ master ]
  #   branches: [ master, feature-/*, hotfix-/* ] # move to another workflow
  # pull_request: # move to another workflow
  #   branches: [ master, feature-/*, hotfix-/* ] # move to another workflow
```

In order to build and push to Google Container Registry, we have to setup gcloud CLI tool:

```yaml
    # Setup gcloud CLI
    # https://github.com/google-github-actions/setup-gcloud
    - uses: google-github-actions/setup-gcloud@master
      with:
        version: '286.0.0'
        service_account_email: ${{ secrets.GCP_SA_EMAIL }}
        service_account_key: ${{ secrets.GCP_SA_KEY }}
        project_id: ${{ secrets.GCP_PROJECT_ID }}
        export_default_credentials: true

```

Finally, we make a build and push image.
```yaml
  # Build and push image to Google Container Registry
    - name: Build
      run: |-
        gcloud builds submit \
          --quiet \
          --tag "gcr.io/$PROJECT_ID/$SERVICE_NAME:latest"
```
## Contributor
- Veck Hsiao (@fbukevin)
- Jacky Liu (@JackywithaWhiteDog)
- Sammy Kang (@weihsuankang)
