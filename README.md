# 3scale Configuration - GitOps Tutorial

The following tutorial provide steps on leveraging GitOps to configure 3scale using 
3scale CRs (Product, Backend CRs etc.)

## Prerequisites
- 3scale operator being installed (from operatorhub) either should be cluster wide operator(introduced in 3scale 2.12) or if it is namespace specific then the operator should be installed in the namespace where the 3scale CRs are to be applied using GitOps
- Install 3scale using APIManager CR (CR is not included). Please follow [Provision OpenShift Data Foundation](https://github.com/3scale-demos/ossm-3scale-wasm#provision-openshift-data-foundation) and [Provision 3scale](https://github.com/3scale-demos/ossm-3scale-wasm#provision-3scale) to install 3scale

## Install RH OpenShift GitOps
Install Red Hat OpenShift GitOps operator from the OperatorHub in the OCP webconsole

![](images/gitops-operator.png)

This will automatically create 
- `openshift-gitops` namespace 
- `ArgoCD` CR with the name `openshift-gitops` in openshift-gitops namespace.
 `ArgoCD` CR creates a bunch of deployments. These deployments together make ArgoCD application
- `AppProject` CR with the name `default` in openshift-gitops namespace.

## Login to OpenShift GitOps
Go to the login page using the route URL it creates as `openshift-gitops-server` in `openshift-gitops` namespace.

You can login to `OpenShift GitOps` using the `admin` user that comes with ArgoCD deployment OR using
OpenShift as oauth provider by clicking the button `LOG IN VIA OPENSHIFT`.

Find the password for the admin in `openshift-gitops-cluster` secret in `openshift-gitops` namespace.

## Create password secret for tenant creation

```
oc project 3scale
```
```
oc apply -f 3scale/tenants/tenant-password-secret.yaml
```
```
oc apply -f 3scale/tenants/tenant-password-secret.yaml
```

## Create Tenants in different namespace

### Development Tenant

```
oc apply -f 3scale/namespaces/development-namespace.yaml
```
```
oc apply -f 3scale/tenants/tenant-development.yaml
```
### Testing Tenant
```
oc apply -f 3scale/namespaces/testing-namespace.yaml
```
```
oc apply -f 3scale/tenants/tenant-testing.yaml
```

### Production Tenant
```
oc apply -f 3scale/namespaces/production-namespace.yaml
```
```
oc apply -f 3scale/tenants/tenant-production.yaml
```


## Enabling RBAC
Create cluster role to create, update, delete 3scale CRs (Need to have OCP admin access for this)

```
oc apply -f rbac/ClusterRole_gitops-threescale-access.yaml
```
Assign the cluster role to sa `openshift-gitops-argocd-application-controller`

```
oc adm policy add-role-to-user gitops-threescale-access system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n threescale-development
```
```
oc adm policy add-role-to-user gitops-threescale-access system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n threescale-testing
```
```
oc adm policy add-role-to-user gitops-threescale-access system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n threescale-production
```

## Connect the git repository

Configure the repositories to be connected by the ArgoCD application 

Click `Manage your repositories, projects, settings` icon on the left panel of the ArgoCD console, Click 
`Repositories` and Click either `Connect repo using SSH` OR `Connect repo using HTTPS` and fill in the form as shown below and click `CONNECT`. Make sure it is SUCCESSFUL.

![](images/gitops-connectrepo.png)

## Create ArgoCD Application
 
Create the ArgoCD Application for all the three environments/tenants. 

```
oc apply -f gitops/Application_threescale-dev.yaml -n openshift-gitops
```
```
oc apply -f gitops/Application_threescale-test.yaml -n openshift-gitops
```
```
oc apply -f gitops/Application_threescale-prod.yaml -n openshift-gitops
```
Three ArgoCD application `threescale-dev` , `threescale-test` and `threescale-prod`are created.

## 3scale CRs
**Please note that the directory structure used for 3scale CRs are for the tutorial purpose only. Please adjust based on the needs**

3scale CRs required for this tutorial are 3scale/backend-echo-api.yaml and 3scale/product-echo-api.yaml

## Manual Synch of 3scale CRs

The three GitOps applications configured to synch manually. But, it can be changed to synch automatically i.e. changes committed to git repo are automatically applied to 3scale else go to the GitOps console using the route URL it creates as `openshift-gitops-server` in `openshift-gitops` namespace.

Click `Manage Application` icon on the left panel of the ArgoCD console. You will then see `threescale-dev` application as shown below
![](images/gitops-application.png)

Click `SYNC` and `SYNCHRONIZE` as shown below to synch the 3scale CRs 
![](images/gitops-synch.png)

Once synched then the application should look as below
![](images/gitops-synched.png)

Go to 3scale Admin console and observe that the product `customer` and backend `customer-order` are configured as shows below
![](images/3scale-synch.png)