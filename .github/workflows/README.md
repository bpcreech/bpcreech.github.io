# GCS setup notes

From:
https://github.com/google-github-actions/auth?tab=readme-ov-file#direct-wif

this should all be one-shot setup.

## initialize:

```
export PROJECT_ID=bpcreech-test-1
```

## create the workload identity pool:

```
gcloud iam workload-identity-pools create "github"   --project="${PROJECT_ID}"   --location="global"   --display-name="GitHub Actions Pool"
```

## get the workload identity pool ID:

```
$ gcloud iam workload-identity-pools describe "github"   --project="${PROJECT_ID}"   --location="global"   --format="value(name)"
```
-> `projects/433482736901/locations/global/workloadIdentityPools/github`

## create a workload identity *provider*:

```
gcloud iam workload-identity-pools providers create-oidc "blog"   --project="${PROJECT_ID}"   --location="global"   --workload-identity-pool="github"   --display-name="My GitHub repo Provider"   --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository"   --issuer-uri="https://token.actions.githubusercontent.com"
```

## get the workload identity pool *provider* name:

```
$ gcloud iam workload-identity-pools providers describe "blog"   --project="${PROJECT_ID}"   --location="global"   --workload-identity-pool="github"   --format="value(name)"
```

-> projects/433482736901/locations/global/workloadIdentityPools/github/providers/blog

## set variables:

```
export REPO=bpcreech/blog
export WORKLOAD_IDENTITY_POOL_ID=projects/433482736901/locations/global/workloadIdentityPools/github
```

## add an IAM binding so the workload identity pool can mess with GCS:

```
gcloud projects add-iam-policy-binding "${PROJECT_ID}"   --member="principalSet://iam.googleapis.com/${WORKLOAD_IDENTITY_POOL_ID}/attribute.repository/${REPO}" --role=roles/storage.admin
```

## add to ./github/workflows/publish.yaml:

```
- uses: 'google-github-actions/auth@v2'
  with:
    project_id: 'bpcreech-test-1'
    workload_identity_provider: 'projects/433482736901/locations/global/workloadIdentityPools/github/providers/blog'
```
