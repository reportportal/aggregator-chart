
# [ReportPortal.io](http://ReportPortal.io)

[![Join Slack chat!](https://img.shields.io/badge/slack-join-brightgreen.svg)](https://slack.epmrpp.reportportal.io/)
[![stackoverflow](https://img.shields.io/badge/reportportal-stackoverflow-orange.svg?style=flat)](http://stackoverflow.com/questions/tagged/reportportal)
[![GitHub contributors](https://img.shields.io/badge/contributors-102-blue.svg)](https://reportportal.io/community)
[![Docker Pulls](https://img.shields.io/docker/pulls/reportportal/service-api.svg?maxAge=25920)](https://hub.docker.com/u/reportportal/)
[![License](https://img.shields.io/badge/license-Apache-brightgreen.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Build with Love](https://img.shields.io/badge/build%20with-❤%EF%B8%8F%E2%80%8D-lightgrey.svg)](http://reportportal.io?style=flat)

ReportPortal is a TestOps service, that provides increased capabilities to speed up results analysis and reporting through the use of built-in analytic features.

## Prerequisites

> **Note:** The minimal requirements for a ReportPortal 1-node solution are 2 CPUs and 6Gi of memory

* Kubernetes v1.30+
* Helm Package Manager v3.10+

## Pre-Deployment Steps

### Creating Kubernetes Secrets

> **⚠️ Security Best Practice:** It is **strongly recommended** to use Kubernetes secrets for all sensitive credentials instead of inline values in `values.yaml`. This prevents sensitive data from being stored in version control and provides better security.

Before deploying the chart, you should create Kubernetes secrets for any credentials you plan to use. The aggregator supports secrets for the following services:

#### 1. X (formerly Twitter) Credentials

Create a secret containing X API credentials:

```bash
kubectl create secret generic x-credentials \
  --from-literal=x-consumer='your-consumer-key' \
  --from-literal=x-consumer-secret='your-consumer-secret' \
  --from-literal=x-token='your-token' \
  --from-literal=x-token-secret='your-token-secret' \
  --namespace=<your-namespace>
```

Then set in `values.yaml`:
```yaml
landing:
  x:
    secretName: "x-credentials"
```

#### 2. GitHub Token

Create a secret containing GitHub personal access token:

```bash
kubectl create secret generic github-credentials \
  --from-literal=github-token='your-github-token' \
  --namespace=<your-namespace>
```

Then set in `values.yaml`:
```yaml
landing:
  github:
    secretName: "github-credentials"
```

#### 3. Google API Credentials

Create a secret containing Google API key and reCAPTCHA key:

```bash
kubectl create secret generic google-credentials \
  --from-literal=google-api-key='your-google-api-key' \
  --from-literal=google-recaptcha-key='your-recaptcha-site-key' \
  --namespace=<your-namespace>
```

Then set in `values.yaml`:
```yaml
landing:
  google:
    secretName: "google-credentials"
```

#### 4. Google Application Credentials (Service Account JSON)

For Google Cloud service account authentication, create a secret containing the JSON credentials file content. **The secret will be mounted as a file volume, and the file path will be set as the `GOOGLE_APPLICATION_CREDENTIALS` environment variable.**

**Method 1: From file (Recommended)**

```bash
kubectl create secret generic google-app-credentials \
  --from-literal=credentials.json="$(cat /path/to/your/service-account-key.json)" \
  --namespace=<your-namespace>
```

**Method 2: Using the actual filename as the secret key**

If your JSON file has a specific name (e.g., `credentials.json`):

```bash
kubectl create secret generic google-app-credentials \
  --from-literal=google-credentials.json='{"type":"service_account","project_id":"your-project-id",...}' \
  --namespace=<your-namespace>
```

Then configure in `values.yaml`:
```yaml
landing:
  google:
    applicationCredentials:
      credentialsSecretName: "google-app-credentials"
      credentialsKeyName: "credentials.json"
      credentialsMountPath: "/etc/google"     # Where the secret will be mounted
      credentialsPath: "/etc/google/credentials.json"  # Full path to the mounted file
```

Or pass via Helm command:
```bash
helm install my-release reportportal/aggregator \
  --set landing.google.applicationCredentials.credentialsSecretName=google-app-credentials \
  --set landing.google.applicationCredentials.credentialsKeyName=credentials.json \
  --set landing.google.applicationCredentials.credentialsMountPath=/etc/google \
  --set landing.google.applicationCredentials.credentialsPath=/etc/google/credentials.json
```

**How it works:**
- The secret is **mounted as a volume** at `/etc/google` (or your custom `credentialsMountPath`)
- The JSON file from the secret is available at `/etc/google/credentials.json` (or your custom `credentialsPath`)
- The `GOOGLE_APPLICATION_CREDENTIALS` environment variable is set to the file path (e.g., `/etc/google/credentials.json`)
- Google Cloud libraries will read the credentials from this file path

**Note:** The JSON credentials file should contain service account information with the following structure:
```json
{
  "type": "service_account",
  "project_id": "your-project-id",
  "private_key_id": "key-id",
  "private_key": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n",
  "client_email": "service-account@appspot.gserviceaccount.com",
  "client_id": "client-id",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/..."
}
```

#### 5. Mailchimp API Key

Create a secret containing Mailchimp API key:

```bash
kubectl create secret generic mailchimp-credentials \
  --from-literal=mailchimp-api-key='your-mailchimp-api-key' \
  --namespace=<your-namespace>
```

Then set in `values.yaml`:
```yaml
landing:
  mailchimp:
    secretName: "mailchimp-credentials"
```

#### 6. Contentful Credentials

Create a secret containing Contentful space ID and access token:

```bash
kubectl create secret generic contentful-credentials \
  --from-literal=contentful-space-id='your-space-id' \
  --from-literal=contentful-token='your-access-token' \
  --namespace=<your-namespace>
```

Then set in `values.yaml`:
```yaml
landing:
  contentful:
    secretName: "contentful-credentials"
```

Or pass via Helm command:
```bash
helm install my-release reportportal/aggregator \
  --set landing.contentful.secretName=contentful-credentials
```

### Alternative: Using Inline Values (Not Recommended for Production)

While the chart supports inline values for development/testing, this is **not recommended for production**:

```yaml
landing:
  x:
    secretName: ""  # Empty = use inline values
    consumer: "your-consumer-key"
    consumerSecret: "your-consumer-secret"
    # ... etc
```

### Verifying Secrets

After creating secrets, verify they exist:

```bash
kubectl get secrets -n <your-namespace>
kubectl describe secret <secret-name> -n <your-namespace>
```

## Installing the Chart

Add the official ReportPortal Helm Chart repository:

```bash
helm repo add reportportal https://reportportal.io/aggregator && helm repo update aggregator
```

Install the chart:

```bash
helm install my-release reportportal/aggregator
```

## Uninstalling the Chart

```bash
helm uninstall my-release 
```

## Configuration

### Install from sources

For fetching chart dependencies, use the command:

```bash
helm install ./aggregator
```

### Install specific version

To search for available versions of a chart, use:

```bash
helm search repo aggregator --versions
```

To install a specific version of a chart, use:

```bash
helm install my-release reportportal/aggregator --version 1.46.1
```

## Documentation

* [General User Manual](https://reportportal.io/docs/)

## Community / Support

* [**Slack chat**](https://reportportal-slack-auto.herokuapp.com)
* [**Security Advisories**](https://github.com/reportportal/reportportal/blob/master/SECURITY_ADVISORIES.md)
* [GitHub Issues](https://github.com/reportportal/reportportal/issues)
* [Stackoverflow Questions](http://stackoverflow.com/questions/tagged/reportportal)
* [Twitter](http://twitter.com/ReportPortal_io)
* [Facebook](https://www.facebook.com/ReportPortal.io)
* [YouTube Channel](https://www.youtube.com/channel/UCsZxrHqLHPJcrkcgIGRG-cQ)

## License

Report Portal is [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0).