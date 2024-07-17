# Setup External DNS Ionos Webhook

> NOTE: Extracted from https://github.com/DOME-Marketplace/dome-gitops/tree/main/doc/external-dns-ionos-webhook

## 0. Prerequisite - Zone registration

> NOTE: Setup of External DNS Ionos Webhook, according to the README.md at URL:
https://github.com/ionos-cloud/external-dns-ionos-webhook

### 0.1. Create subdomains

- Access to Ionos Domain Management application (request can be submitted to souvik.sengupta@ionos.com)
  Where domain needs to be registered and needed subdomains need to be manually created.
  https://mein.ionos.de/subdomains/dome-marketplace-prd.org
- Access to Ionos DCD System (request can be submitted to souvik.sengupta@ionos.com)
  Credentials will be further used to generate token
  https://dcd.ionos.com/latest/

### 0.2. Token generation

Generate token using Ionos DCD System credentials via endpoint https://api.ionos.com/auth/v1/tokens/generate;

```bash
curl --location 'https://api.ionos.com/auth/v1/tokens/generate' \
--header 'Authorization: Basic DCD_BASIC_CREDENTIALS'
```

### 0.3. Create Zone

Create Zone for the domain(s) to be used in Ionos DNS API via:

```bash
curl --location --request POST 'https://dns.de-fra.ionos.com/zones' \
--header 'Authorization: Bearer tokenValue' \
--header 'Content-Type: application/json' \
--data-raw '{
"properties":{
"description": "desc",
"enabled": true,
"zoneName": "example-dome-marketplace.org"
}
}'
```

## 1. Add Chart

Add bitnami helm chart:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

## 2. Prepare Secret

Prepare Secret named ionos-credentials based on the token generated via endpoint
https://api.ionos.com/auth/v1/tokens/generate:

```bash
kubectl create secret generic ionos-credentials --from-literal=api-key='tokenWithoutBearer'
```    

## 3. Install Chart

```bash
helm install external-dns-ionos bitnami/external-dns -f values.yaml
```