# OpenShift

## Prepare cluster installation according to instruction  from https://okd.io

Additionally configure DNS to resolve sub-domains of *okd* to cluster host machine.

## Set up cluster and spin up cluster with capabilities to restart it.

```bash 
oc cluster up --routing-suffix=okd --base-dir=~/okd/_cluster_config_dir --public-hostname=master.okd
```

# Spinnaker

## Configure Spinnaker deployment using Halyard.

```bash
 hal config provider kubernetes account add my-k8s-v2-account \
    --provider-version v2 \
    --context $(kubectl config current-context) \
    --environment dev \
    --only-spinnaker-managed=true \
    --namespaces=spin
```

```bash
hal config features edit --artifacts true
```

```bash
hal config deploy edit --type distributed --account-name my-k8s-v2-account
```

## Spinnaker for configuration storage requires S3 or S3 like storage

```bash
docker run -d -p 9001:9000 --name minio1 \
    -v /opt/minio/data:/data \
    -v /opt/minio/config:/root/.minio \
    -e "MINIO_ACCESS_KEY=AKIAIOSFODNN7EXAMPLEHKJHK" \
    -e "MINIO_SECRET_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLKJLEKEY" \
    minio/minio server /data
```

```bash
hal config storage s3 edit --endpoint http://192.168.86.77:9001/ --access-key-id AKIAIOSFODNN7EXAMPLEHKJHK --secret-access-ke
hal config storage edit --type s3
```

## Deploy Spinnaker to Openshift cluster

```bash
hal deploy apply
```

## Soften security restrictions to be able to run Spinnaker

### Make developer an admin in spinnaker project

```bash
oc adm policy add-role-to-user admin developer -n spinnaker
```

### Allow to run containers within spinnaker project from any user

```bash
oc -n spinnaker edit scc restricted
```

```yaml
requiredDropCapabilities:
...
  runAsUser:
    type: RunAsAny
  seLinuxContext:
...
```

## Fix routing to Spinnaker Gate from Spinnaker Deck by modifying secret `spin-deck-files`.  Content is similar to what we have in `spinnaker/deck-settings.js` folder

```js
var gateHost = 'http://spin-gate-spinnaker.okd';
```

## Configure origin to allow request from http://*-spinnaker.okd. To do this we can define env variable for Spinnaker Gate pod.

```bash
CORS_ALLOWEDORIGINSPATTERN = .*(\.okd)
```