# DEPRECATED

There are much better ways to do this in modern times, and this repo has potential security issues, so I'm gonna archive it!

I'm not a jerk, so I won't leave you totally hanging- if you were thinking about using this solution, I'd suggest 
giving https://cloud.google.com/secret-manager/docs/secret-manager-managed-csi-component a look. It's currently supported and much easier :D 

# Readme

This repo demonstrates how to inject versioned secrets into GKE pods at runtime using a custom [MutatingAdmissionWebhook](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#what-are-they) and [Pod Workload Identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity).

It's based on https://github.com/GoogleCloudPlatform/berglas/tree/master/examples/kubernetes but extends this demo to use workload identity instead of the GKE service account.

# Setup berglas and secret store

https://github.com/GoogleCloudPlatform/berglas
`berglas` is a CLI for storing and retrieving secrets in GCP. It works both with secrets encrypted with Cloud KMS/stored in GCS and is compatible with Secret Manager.
We chose to base this integration on berglas because it could be integrated into Quibi services with minimal modification. When Secret Manager becomes more mature and the GKE
integrations become more full-featured, berglas has a mode to migrate all of your stored secrets into Secrets Manager.

Follow along with https://github.com/GoogleCloudPlatform/berglas#setup steps 1 thru 6.
These steps will use beglas to bootstrap your secret store bucket, KMS keyring and key.
The following instructions will refer to variables and resources set and created in those steps.

# Create and configure Google Service Account
Create a Google Service Account (GSA) that GKE will use to access and decrypt the secrets.

`gcloud iam service-accounts create "berglas-reader-gke" --display-name="Berglas reader GKE"`

Note: it is possible to use different Google service accounts for each service and assign each service a path under which all its secrets would live, but is beyond the scope of this demo.

Grant the service account read access to the secret store bucket:

`gsutil iam ch "serviceAccount:berglas-reader-gke@${PROJECT_NAME}.iam.gserviceaccount.com:objectViewer" "${SECRETS_BUCKET}"`

Bind iam policies to the service account:

```
gcloud kms keys add-iam-policy-binding ${KMS_KEY} \
  --member "serviceAccount:berglas-reader-gke@${PROJECT_NAME}.iam.gserviceaccount.com" \
  --role roles/cloudkms.cryptoKeyDecrypter
```

# Configure GKE for Workload identity
Workload Identity must be enabled in the GKE cluster https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity?hl=en_US&_ga=2.220667864.-1552821829.1576603081#enable_workload_identity_on_a_new_cluster


Create a Kubernetes service account in a namespace.
`kubectl create serviceaccount "berglas-reader-gke"`

Create the workloadIdentity bindings for this service account. These define the relationship between a namespace/ksa and a google service account.
When a pod in this namespace is running under this service account, it will assume all permissions of the google service account when interacting with
Google APIs. This is exposed to the Pod via application default credentials.

```
gcloud iam service-accounts add-iam-policy-binding \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:${PROJECT_NAME}.svc.id.goog[default/berglas-reader-gke]" berglas-reader-gke@$PROJECT_NAME}.iam.gserviceaccount.com

kubectl annotate serviceaccount \
  --namespace "${K8S_NAMESPACE}" \
  --overwrite \
  berglas-reader-gke \
  "iam.gke.io/gcp-service-account=berglas-reader-gke@${PROJECT_NAME}.iam.gserviceaccount.com"
```

# Install AdmissionController Webhook
This webhook watches for Pod creation events. When it sees one, it mutates the PodSpec. When it finds the `berglas://` syntax, it
does some magic to substitute those references to decrypted values in the Pod.

Cd to `kubernetes` in this repo.

Make sure cloud functions API is enabled:
```
gcloud services enable --project ${PROJECT_ID} \
  cloudfunctions.googleapis.com
```

Deploy the mutation webhook cloud function and register the webhook with the cloud function URL:
```
gcloud functions deploy berglas-secrets-webhook \
  --project ${PROJECT_ID} \
  --runtime go111 \
  --entry-point F \
  --trigger-http
ENDPOINT=$(gcloud functions describe berglas-secrets-webhook --project ${PROJECT_ID} --format 'value(httpsTrigger.url)')
sed "s|REPLACE_WITH_YOUR_URL|$ENDPOINT|" deploy/webhook.yaml | kubectl apply -f -
```


# Test time!

Create a secret, note the generation ID:
```
berglas create berglas://pso-quibi-qsecrets/my-secret1 "this value is secret. first edition" \
  --key projects/pso-quibi/locations/global/keyRings/berglas/cryptoKeys/berglas-key
Successfully created secret [my-secret1] with generation [1583534284391004]

berglas access berglas://pso-quibi-qsecrets/my-secret1#1583534284391004
this value is secret. first edition
```

Update the secret
```
berglas update berglas://pso-quibi-qsecrets/my-secret1 "this is a secret value. second edition"
  --key projects/pso-quibi/locations/global/keyRings/berglas/cryptoKeys/berglas-key
Successfully updated secret [my-secret1] to generation [1583534416742067]

berglas access berglas://pso-quibi-qsecrets/my-secret1#1583534284391004
this value is secret. first edition

berglas access berglas://pso-quibi-qsecrets/my-secret1#1583534416742067
this is a secret value. second edition
```

Create an app to read the secret (note the syntax for secret references):
`kubectl apply -f kubernetes/wi/deployment.yaml`

Check the value:
```
(in one terminal)
kubectl port-forward deployment/app1 8080:8080
(in a second)
curl localhost:8080 | jq .
```
The decypted values will be included in this output.
