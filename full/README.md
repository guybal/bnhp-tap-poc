# TAP Installation Prerequisites for Client 

This user guide specifies the required steps by the client, for a successful TAP installation.

<!-- TOC -->
* [TAP Installation Prerequisites for Client](#tap-installation-prerequisites-for-client)
* [Steps](#steps)
  * [1. Image Registry Configuration](#1-image-registry-configuration)
    * [a. Create Image Registry Projects](#a-create-image-registry-projects)
    * [b. Create Image Registry Robot Account or Applicative User](#b-create-image-registry-robot-account-or-applicative-user)
    * [c. Set Permissions for Robot Account](#c-set-permissions-for-robot-account)
  * [2. Git Configuration](#2-git-configuration)
    * [a. Create Git projects](#a-create-git-projects)
    * [b. Create Git token](#b-create-git-token)
        * [**Github**](#github)
        * [**GitLab**](#gitlab)
    * [c. Create OAuth App in Organization level](#c-create-oauth-app-in-organization-level)
        * [**GitHub**](#github)
        * [**GitLab**](#gitlab)
  * [3. Push TAP GUI Blank Catalog Content](#3-push-tap-gui-blank-catalog-content)
    * [1. Download `tap-gui-blank-catalog.tgz`](#1-download-tap-gui-blank-catalogtgz)
    * [2. Extract `.tgz` file](#2-extract-tgz-file)
    * [3. Push Content to `tap-catalog` Git Project](#3-push-content-to-tap-catalog-git-project)
  * [4. Relocate TAP Packages](#4-relocate-tap-packages)
    * [a. Prerequisites](#a-prerequisites)
        * [1. Login to Your Private Image Registry](#1-login-to-your-private-image-registry)
        * [2. Login to VMware's Image Registry](#2-login-to-vmwares-image-registry)
    * [b. Import TAP Packages Overlay Images](#b-import-tap-packages-overlay-images)
        * [1. Relocate Packages](#1-relocate-packages)
  * [5. Install and configure `postgresSQL` database tor TAP GUI](#5-install-and-configure-postgressql-database-tor-tap-gui)
    * [a. Install `helm`](#a-install-helm)
    * [b. Create `psql-values.yaml` values file](#b-create-psql-valuesyaml-values-file)
    * [c. Install `Postgres` helm chart](#c-install-postgres-helm-chart)
    * [d. Validate](#d-validate)
  * [6. Create `tap-values.yaml` file](#6-create-tap-valuesyaml-file)
  * [7. Install](#7-install)
<!-- TOC -->

# Steps
## 1. Image Registry Configuration
### a. Create Image Registry Projects

TAP will use 3 dedicated projects/repos in the image registry.

Create the following projects under your image registry:
- `tap`: will be used for hosting TAP packages
- `tap-workloads`: will be used for hosting workload images built during supply chain.
- `tap-source-code`: will be used for hosting source code image bundles developers create during inner loop iteration.

### b. Create Image Registry Robot Account or Applicative User
- `tap-system-user`: will be used by TAP to pull/push TAP packages, workload images and source code image bundles.

### c. Set Permissions for Robot Account
`tap-system-user` robot account needs to have the following permissions on `tap`, `tap-workloads` and `tap-source-code` projects:

```bash
List Repository
Pull Repository
Push Repository
Delete Repository
Read Artifact
List Artifact
Delete Artifact
Create Artifact label
Delete Artifact label
Create Tag
Delete Tag
List Tag 
```
![img.png](images/05-01-robot-account-permissions.png)

* This is an example for Harbor configuration. 
---

## 2. Git Configuration
### a. Create Git projects

TAP will use 3 dedicated projects/repos under your Git provider.

Create 3 Git repositories under your Git Organization account:
- `tap-catalog` - will be used by TAP GUI service catalog.
- `tap-gitops` - will be used by TAP's supply chain as destination to push manifests to.
- `tap-namespace-provisioner` - will be used to configure auto generated resources in developers' namespaces.

### b. Create Git token

##### **Github**

This token will be used to auto-generate repositories from Application Accelerator and to pull and push manifests.
1. In your GitHub Organization account go to: `Account > Settings > Developer Settings > Personal Access Tokens > Tokens (Classic)`
2. Click on `Generate New Token`
3. Select Scopes [**repo**, **user**, **project**, **admin:org**, **workflow**, **gists**].
4. Copy the content of the access token as it will be used to configure `tap-values.yaml` file.
  - **repo**:

   ![img.png](images/05-01-github-token-repo-permissions.png)

  - **user**:

   ![img.png](images/05-01-github-token-user-permissions.png)

  - **project**:

   ![img.png](images/05-01-github-token-project-permissions.png)

  - **admin:org**:

   ![img.png](images/05-01-github-token-org-permissions.png)

  - **workflow**:

   ![img.png](images/05-01-github-token-workflow-permissions.png)

  - **gists**:

   ![img.png](images/05-01-github-token-gists-permissions.png)

##### **GitLab**

This token will be used to auto-generate repositories from Application Accelerator and to pull and push manifests.
1. In your GitHub Organization account go to: `Profile > Edit Profile > Access Tokens`
2. Select Scopes [**api**, **read_user**, **read_repository**, **write_repository**].
3. Click on `Create personal access token`
4. Copy the content of the access token as it will be used to configure `tap-values.yaml` file.

![img.png](images/05-02-gitlab-token-create.png)

Copy the content of the access token:
![img.png](images/05-02-gitlab-token-create-01.png)

![img.png](images/05-02-gitlab-token.png)

### c. Create OAuth App in Organization level
Will be used as TAP's identity under your Git Org account.

##### **GitHub**

1. In your GitHub Organization account go to: `Account > Settings > Developer Settings > OAuth Apps`
2. Click on `New OAuth App`.
3. Fill the requested properties:
  - `Application name`: TAP-Backstage
  - `Homepage URL`: TAP-GUI homepage URL
  - `Authorization callback URL`: add the suffix `/api/auth/github/handler/frame` to TAP GUI URL, for example: `https://tap-gui.guy-tap.terasky.demo/api/auth/github/handler/frame`
4. Generate a client secret.
5. Copy `clientId` and `clientSecret` as both will be used to configure `tap-values.yaml` file.

##### **GitLab**

Enable OIDC/OAuth in GitLab:
1. Sign in to GitLab as an administrator.
2. Select `Profile` > `Preferences` > `Applications`.
3. Fill in the details for your client application, including the `name`, `redirect URI`, and `allowed scopes`.
   - `Name`: TAP-Backstage
   - `Homepage URL`: TAP-GUI homepage URL for example `https://tap-gui.VIEW_CLUSTER_FQDN`
   - `Scopes`: Select scopes [**api**, **read_user**, **read_repository**, **write_repository**, **openid**, **profile**]
   - `Authorization callback URL`: add the suffix `/api/auth/github/handler/frame` to TAP GUI URL. For example: `https://tap-gui.VIEW_CLUSTER_FQDN/api/auth/github/handler/frame`

4. Make sure the `openid` scope is enabled.
5. Select `Save application` to create the new OAuth application.

![img.png](images/05-02-gitlab-oath-app.png)

---

## 3. Push TAP GUI Blank Catalog Content
### 1. Download `tap-gui-blank-catalog.tgz`
   - For TAP v1.5: Go to `https://network.tanzu.vmware.com/products/tanzu-application-platform/#/releases/1317341/file_groups/6091` and download `tap-gui-blank-catalog.tgz`.
   - For TAP v1.4: Go to `https://network.tanzu.vmware.com/products/tanzu-application-platform/#/releases/1317261/file_groups/6091` and download `tap-gui-blank-catalog.tgz`.

**Alternative link:**
  - Go to `https://network.tanzu.vmware.com/products/tanzu-application-platform/` > `tap-gui-catalog-latest` > download `tap-gui-blank-catalog.tgz`.

### 2. Extract `.tgz` file

Copy the `tap-gui-blank-catalog.tgz` into an empty folder called `tap-catalog`
```bash
cd tap-catalog

# extract
tar -xvf tap-gui-blank-catalog.tgz
```

Folder structure should look like this:
```bash
tap-catalog/
  blank/
    .git/
    components/
    docs/
    domains/
    groups/
    systems/
    catalog-info.yaml
    mkdocs.yml
    INSTALLATION.md
```

### 3. Push Content to `tap-catalog` Git Project
```bash
cd blank

git init 
git add .
git commit -m "first commit"
git branch -M main
git remote add origin <GIT_REPO_HTTPS_CLONE_URL> # for example: https://github.com/guybal/tap-catalog.git
git push -u origin main
```

Where:
- `GIT_REPO_HTTPS_CLONE_URL` is the `tap-catalog` Git repository created in step **2.a** of this user guide.

---

## 4. Relocate TAP Packages

### a. Prerequisites

##### 1. Login to Your Private Image Registry
First, login into your local image registry:
```bash
docker login ${INSTALL_REGISTRY_HOSTNAME}
```

##### 2. Login to VMware's Image Registry
Second, login into VMware's registry:
```bash
docker login registry.tanzu.vmware.com
```

### b. Import TAP Packages Overlay Images

##### 1. Relocate Packages
```bash
# package repositories
export INSTALL_REGISTRY_USERNAME=tap-system-user # For Harbor: robot$tap-system-user
export INSTALL_REGISTRY_PASSWORD=<ROBOT_ACCOUNT_PASSWORD>
export INSTALL_REGISTRY_HOSTNAME=<PRIVATE_IMAGE_REGISTRY_FQDN>
export INSTALL_REPO=tap

# (Option 1) For TAP latest version:
export TAP_VERSION=$(imgpkg tag list -i registry.tanzu.vmware.com/tanzu-application-platform/tap-packages | grep -v sha | sort -V | tail -1)
export TAP_BS_VERSION=$(imgpkg tag list -i registry.tanzu.vmware.com/tanzu-application-platform/full-tbs-deps-package-repo | grep -v sha | sort -V | tail -1)
export SCG_VERSION=$(imgpkg tag list -i registry.tanzu.vmware.com/spring-cloud-gateway-for-kubernetes/scg-package-repository | grep -v sha | sort -V | tail -1)

# (Option 2) For TAP fixed version:
export TAP_VERSION=1.5.2    
export TAP_BS_VERSION=1.10.10
export SCG_VERSION=2.0.4

# Relocate to local image registry
imgpkg copy -b registry.tanzu.vmware.com/tanzu-application-platform/tap-packages:${TAP_VERSION} --to-repo ${INSTALL_REGISTRY_HOSTNAME}/${INSTALL_REPO}/tap-packages
imgpkg copy -b registry.tanzu.vmware.com/tanzu-application-platform/full-tbs-deps-package-repo:${TAP_BS_VERSION} --to-repo ${INSTALL_REGISTRY_HOSTNAME}/${INSTALL_REPO}/tbs-full-deps
imgpkg copy -b registry.tanzu.vmware.com/tanzu-application-platform/scg-package-repository:${SCG_VERSION} --to-repo ${INSTALL_REGISTRY_HOSTNAME}/${INSTALL_REPO}/scg-package-repository

# Relocate overlay images
docker pull ghcr.io/vrabbi/docker:dind-rootless
docker pull spotify/techdocs:v1.1.2

docker tag ${INSTALL_REGISTRY_HOSTNAME}/${INSTALL_REPO}/vrabbi/docker:dind-rootless
docker tag ${INSTALL_REGISTRY_HOSTNAME}/${INSTALL_REPO}/techdocs:v1.1.2

docker push ${INSTALL_REGISTRY_HOSTNAME}/${INSTALL_REPO}/vrabbi/docker:dind-rootless
docker push ${INSTALL_REGISTRY_HOSTNAME}/${INSTALL_REPO}/techdocs:v1.1.2
```

---
## 5. Install and configure `postgresSQL` database tor TAP GUI

### a. Install `helm`
```bash
wget https://get.helm.sh/helm-v3.11.1-linux-amd64.tar.gz
tar -zxvf helm-v3.11.1-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
```
### b. Create `psql-values.yaml` values file
```yaml
auth:
  postgresPassword: FILL_ME_IN
architecture: replication
replication:
  synchronousCommit: remote_apply
  numSynchronousReplicas: 1
readReplicas:
  replicaCount: 1
image:
  registry: docker.io
  repository: bitnami/postgresql
  tag: 15.3.0-debian-11-r0
```

### c. Install `Postgres` helm chart
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami

kubectl create ns postgres-tap-gui

helm install postgres-tap-gui bitnami/postgresql -n postgres-tap-gui --values psql-values.yaml
# Read more about the installation in the PostgreSQL packaged by Bitnami Chart Github repository
```
**output**
```bash
NAME: postgres-tap-gui
LAST DEPLOYED: Sun Feb 19 15:29:55 2023
NAMESPACE: postgres-tap-gui
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: postgresql
CHART VERSION: 12.2.1
APP VERSION: 15.2.0

** Please be patient while the chart is being deployed **

PostgreSQL can be accessed via port 5432 on the following DNS names from within your cluster:

    postgres-tap-gui-postgresql.postgres-tap-gui.svc.cluster.local - Read/Write connection

To get the password for "postgres" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace postgres-tap-gui postgres-tap-gui-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

To connect to your database run the following command:

    kubectl run postgres-tap-gui-postgresql-client --rm --tty -i --restart='Never' --namespace postgres-tap-gui --image docker.io/bitnami/postgresql:15.2.0-debian-11-r2 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host postgres-tap-gui-postgresql -U postgres -d postgres -p 5432

    > NOTE: If you access the container using bash, make sure that you execute "/opt/bitnami/scripts/postgresql/entrypoint.sh /bin/bash" in order to avoid the error "psql: local user with ID 1001} does not exist"

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace postgres-tap-gui svc/postgres-tap-gui-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432

WARNING: The configured password will be ignored on new installation in case when previous Posgresql release was deleted through the helm command. In that case, old PVC will have an old password, and setting it through helm won't take effect. Deleting persistent volumes (PVs) will solve the issue.
```
### d. Validate
```bash
kubectl get po -n postgres-tap-gui
```
**output**
```bash
NAME                            READY   STATUS    RESTARTS   AGE
postgres-tap-gui-postgresql-0   1/1     Running   0          63s
```

---
## 6. Create `tap-values.yaml` file
```yaml
profile: full

ceip_policy_disclosed: true

shared:
  ingress_domain: guy-tkg-tap1-cls.terasky.demo                      # @TODO: change
  ingress_issuer: tap-ingress-selfsigned
  image_registry:
    project_path: "harbor.vrabbi.cloud/tap/workloads"
    username: "robot$tap-system-user"
    password: ${ROBOT_ACCOUNT_PASSWORD}

  # @TODO: change
  ca_cert_data: |
    -----BEGIN CERTIFICATE-----
    MIIDdzCCAl+gAwIBAgIQH4wNdThKCJpOv8K1LzP7sDANBgkqhkiG9w0BAQsFADBO
    MRUwEwYKCZImiZPy...
    -----END CERTIFICATE-----

supply_chain: testing_scanning # Can take basic, testing, testing_scanning.

ootb_delivery_basic:
  service_account: default

ootb_supply_chain_basic:
  service_account: default
  registry:
    server: harbor.vrabbi.cloud
    repository: tap/supply-chain/workloads

ootb_supply_chain_testing:
  service_account: default
  registry:
    server: harbor.vrabbi.cloud
    repository: tap/supply-chain/workloads

ootb_supply_chain_testing_scanning:
  service_account: default
  scanning:
    image:
      policy: scan-policy
    source:
      policy: scan-policy
  registry:
    server: harbor.vrabbi.cloud
    repository: tap/supply-chain/workloads
  # maven:
  #     repository:
  #        url: https://MAVEN-URL
  #        secret_name: "maven-creds"
#  gitops:
#   server_address: https://github.com/
#   repository_owner: guybal
#   repository_name: tap-gitops
#   branch: dev
#   ssh_secret: git-creds

policy:
  tuf_enabled: true

buildservice:
  exclude_dependencies: true

tap_gui:
  service_type: ClusterIP

  app_config:
#    proxy:
#      /metadata-store:
#        target: https://metadata-store-app.metadata-store:8443/api/v1
#        changeOrigin: true
#        secure: false
#        headers:
#          # @TODO: change
#          Authorization: "Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IlUwZ2dQZjJveWsxMUJCZHA2V0ZydEFrQmJvdDJMVmlMU1IxbldOQUR2TDgifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJtZXRhZGF0YS1zdG9yZSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJtZXRhZGF0YS1zdG9yZS1yZWFkLXdyaXRlLWNsaWVudCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJtZXRhZGF0YS1zdG9yZS1yZWFkLXdyaXRlLWNsaWVudCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjM4M2Q2Mjc2LTIyZmEtNGI2ZS04NTdiLWY2OGIwNDAyYmMzMyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDptZXRhZGF0YS1zdG9yZTptZXRhZGF0YS1zdG9yZS1yZWFkLXdyaXRlLWNsaWVudCJ9.AQyL-E33AFZH0oneJ920_KcFQFDPFZACCjNu2i_lyr8MmTipxlzJ0TqFelEFi8sWkahEt4OlxB8xgvTPr-PZ-24ccz2pKkylYU4sjoCm6HE1-r8VssehsLhN668CS2zf0e70Y3drXyMRIGSJcVx-XpBJvg-crGw3aY7K0VhD-XvagJs-4A7vYQIiEcDofdo7Mi0bMn8iXmjW5jZNthqR299aE66r3_4-1fK-TC86TooODBAuaj02SsJr9URoq21CSHTtrZej0msJpXCBlS9LwfEYprzv4ehKgA53-vXKZWbFhI6op7ZZZ-LDIkMNBh305mAmM24CUaMAqycR22uk3A"
#          X-Custom-Source: project-star

    # integrations allow Backstage to read or publish data using external providers such as GitHub, GitLab, Bitbucket, LDAP, or cloud providers.
    integrations:
      github:
        - host: github.com
          token: ${GITHUB_TOKEN}

      # For GitHub Apps (No SSH communication):
      #      github:
      #        - appId: ${GITHUB_APP_ID}
      #          clientId: ${GITHUB_APP_CLIENT_ID}
      #          clientSecret: ${GITHUB_APP_CLIENT_SECRET}
      #          allowedInstallationOwners: ['${GITHUB_ORG_NAME}']

#      gitlab:
#        - host: gitlab.com
#          token: ${GITLAB_TOKEN} # @TODO: change 
          # If `host` ISN'T `gitlab.com` apiBaseUrl: https://<GITLAB_SERVER_FQDN>/api/v4
    techdocs:
      generator:
        dockerImage: spotify/techdocs:v1.1.2
        pullImage: false
        runIn: docker
    catalog:
      locations:
        - type: url
          target: ${TAP_GUI_BLANK_CATALOG_URL}/-/blob/main/catalog-info.yaml 
#    backend:
#      database:
#        client: pg
#        connection:
#          host: postgres-tap-gui-postgresql.postgres-tap-gui.svc.cluster.local  # @TODO: change
#          port: 5432
#          user: postgres
#          password: ${POSTGRES_DB_PASSWORD}          # @TODO: change
#          ssl: false
    auth:
      allowGuestAccess: true
      environment: tap_gui
      providers:
        # GitHub authentication
        github:
          tap_gui:
            clientId: ${GITHUB_OAUTH_APP_CLIENT_ID}
            clientSecret: ${GITHUB_OAUTH_APP_CLIENT_SECRET}
            callbackUrl: https://tap-gui.${SHARED_INGRESS_DOMAIN}/api/auth/github/handler/frame
        # AzureAD authentication
#        microsoft:
#          tap_gui:
#            clientId: FILL_ME_IN
#            clientSecret: FILL_ME_IN
#            tenantId: FILL_ME_IN
#        gitlab:
#          tap_gui:
#            clientId: ${OAUTH_GITLAB_CLIENT_ID}
#            clientSecret: #${OAUTH_GITLAB_CLIENT_SECRET}
#            callbackUrl: https://tap-gui.${SHARED_INGRESS_DOMAIN}/api/auth/gitlab/handler/frame
            ## uncomment if using self-hosted GitLab
            # audience: https://gitlab.company.com
            ## uncomment if using a custom redirect URI
            # callbackUrl: https://tap-gui.${SHARED_INGRESS_DOMAIN}/api/auth/gitlab/handler/frame
      loginPage:
        github:
          title: GitHub Login
          message: Sign in using your GitHub account

metadata_store:
  app_service_type: ClusterIP
  ns_for_export_app_cert: '*'

grype:
  namespace: "default"
  targetImagePullSecret: "tap-registry"

excluded_packages:
#  - contour.tanzu.vmware.com
#  - cert-manager.tanzu.vmware.com

package_overlays:
  - name: tap-gui
    secrets:
      - name: techdocs-overlay
#  - name: ootb-templates
#    secrets:
#      - name: scan-stamping-labels-overlay

namespace_provisioner:
  additional_sources:
#    # Add templated tekton testing pipeline for Java applications
#    #    - git:
#    #        ref: origin/main
#    #        subPath: namespace-provisioner-gitops-examples/custom-resources/testing-scanning-supplychain
#    #        url: https://github.com/terasky-int/application-accelerator-samples.git
#    #      path: _ytt_lib/testingscanning   # this user-generated path must always begin with "_ytt_lib/"
#
#    # Add workload service-account patched with cosign and registeries secrets.
#    - git:
#        ref: origin/main
#        subPath: namespace-provisioner-gitops-examples/custom-resources/workload-sa
#        url: https://github.com/terasky-int/application-accelerator-samples.git
#      path: _ytt_lib/workload-sa
#
#    # Add templated tekton testing pipelines for Python, Angular and Golang applications
#    - git:
#        ref: origin/main
#        subPath: namespace-provisioner-gitops-examples/custom-resources/tekton-pipelines
#        url: https://github.com/terasky-int/application-accelerator-samples.git
#      path: _ytt_lib/tektonpipelines   # this user-generated path must always begin with "_ytt_lib/"
#
#    # Add templated Grype scan policy to enforce ["Critical", "High"] CVE's
#    # - git:
#    #     ref: origin/main
#    #     subPath: namespace-provisioner-gitops-examples/custom-resources/scanpolicies/grype
#    #     url: https://github.com/guybal/application-accelerator-samples.git # https://github.com/vrabbi-tap/application-accelerator-samples.git
#    #   path: _ytt_lib/grypepolicy   # this user-generated path must always begin with "_ytt_lib/"
#
    - git:
        ref: origin/main
        subPath: namespace-provisioner-gitops-examples/default-resources-overrides/overlays
        url: https://github.com/terasky-int/application-accelerator-samples.git # https://github.com/vrabbi-tap/application-accelerator-samples.git
      path: _ytt_lib/customize   # this path must always be exactly "_ytt_lib/customize"
```

---
## 7. Install
```bash
tanzu package install tap -p tap.tanzu.vmware.com -v $TAP_VERSION  --values-file tap-values.yaml -n tap-install

tanzu package installed update tap -p tap.tanzu.vmware.com -v $TAP_VERSION  --values-file tap-values.yaml -n tap-install
```

