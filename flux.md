**Flux** enables application deployment (CD) through automatic reconciliation and provides alerting, notifications. It can even scan image repos and update container images automatically before pushing/commiting them back to Git repo.

**What is Flux?**

It is a tool for maintaining k8s cluster in sync with configurations defined on source repos like git, helm, container image, s3

It continuosly monitors running applications, comparing their live state(k8s) to the desire state (git)

It reports the deviation eo help devs manually or automatically synchronize the live state with the desired state

**How does it work?**

It follows the GitOps pattern by using git, s3, helm, OCI repos as the source of truth for the desired state of apps and infra. Based the resources it is going to use helm controller or kustomize controller to deploy and manage applications. 

**Basic Terminologies**

**Bootstrap** : process of installing the flux components withinthe k8s cluster in a GitOps manner    

**Agent**: It is a cluster running process that performs tasks of a user, these are knows as flux controllers are software agents that implement a control loop 

**Controller**: These are implemented based on k8s controller pattern. The pattern uses control loop along with CRDs to manage an aspect of cluster state

**Sources**: Git, Helm, OCI repository, S3 - containing the desired state of the cluster and the requirements to obtain it.

**Kustomization**: kustomization controller represents a local set of kustomize overlay resources that flux is supposed to reconcile in the cluster. It can work with both overlay files and plain k8s manifest 

**Reconcilation**: Ensuring that given states matches with desired state

**Suspend**: suspends the reconcilation of a resource

**Resume**: resume a suspended resource

**Prune**: It's a garbage collection process, this will help in removing the resources automatically, if a resource is deleted from desired state, and if prune is false, then it'll not delete that resource in k8s

**Image**: flux image command works with image automation objects to update a Git repo when new container images are available.

-------------

Let's assume we want to make use of flux cd to deploy an application to k8s using the plain deployment yaml manifest stored in Git repo.

As part of flux installation, a flux-system ns is created and 5 flux controllers are deployed in it. In order to deploy the application, flux needs to fetch the deployement YAML manifest from the git repo. 

We create source obj through source controller, whose primary duty is to provide a standard interface for artifact acqusation. Source controller can connect and fetch artifacts from git, helm, ocr repos and s3 buckets. After fetching, next step is to deploy the application. Based on the type of artifact fetched we either use kustomize or helm controller. In our case, the fetched artifacts are plain YAML manifest. So we can create a kustomization object. which gets the artifact from the source controller, build kustomize overlays and deploy the application to the k8s cluster. Any changes to the plain deployment yaml manifest are automatically reconciled through the source and kustomized controllers. Same process for any kustomization overlay files stored in a repo

Now let's deploy helm charts

Helm charts can be stored in a git, helm or oca repository. The source controller can connect with all these repos and fetch the helm charts. But how does flux do helm release to deploy thea app? In flux-system ns, we have helm controller, which allows us to manage helm chart releases with k8s manifests through the helm relaese k8s CRDs, the desired state of a release is specified. Helm controller performs automated helm actions including helm release, test, rollback and uninstalls. 

So lets say devs have made some code changes, and merged them to a git repo. So this usually trigger CI pipeline, this will push the image to container registery. Now the question is how do we update the image version in the deployment manifest. Generally without flux we either do it manually by editing the deployment.yaml file or extend the same CI pipeline to have another stage to pull, update and commit to merge the changes. Within flux we make use of the image automation using image controller.

Image controller has 2 parts, Image reflector controller, Image automation controller. which works together to update  a GIT repo when new container images like version or tags are available. So the first thing is we make use of image reflector controller, which is going to scan the container image repos and fetch the image metadata. In our case, it's going to fetch the v1 and v2 versions and based on the image policy object, flux gets the latest image tag. The image automation controller updates YAML manifest based on the image policy and commits the changes to the Git repos. How does flux get to know about the changes in git repos? So flux has notification controller, which is desgined to handle both the inbound and outbound events. The controller handles events coming from git, gitlab etc and notifies flux controllers about source changes. The source controller then pulls the new changes about the desired state and the kustomize controller reconciles the application to the latest version. This cycle repeats whenever there is change in the source repo. Flux can also detect image changes by using a notification controller webhook configured on the container image registries or repos. The notification controller can also handle events generated by flux controller such as source, kustomize, helm and send them to external systems like slack, teams etc

