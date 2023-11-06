It is a modern cloud native approach to continuos deployment that leverages the git version control in Github, Bitbucket or Azure DevOps.

GitOps is a framework where the entire code delivery process is controlled via a git repository

GitOps is an extension of DevOps methodologies

It requires the desired state of the sys to be stored as version control

Any updates to git, will trigger the CI/CD pipeline then to k8s. So GitOps is an automated process, which ensures that the desired state in the Git repo and the actual state in k8s env always matches. Using git, any updates are controlled through PRs and the git history provides a sequence of updates allowing us to recover state from any snapshot

**GitOps Principles**

GitOps has 4 principles

1. It mandates infra and app code to be declared in a declarative state. It discourages the use of the imperate approach as it makes reconciliation difficult because imperative approach doesn't store any state

2. To make use of Git. All the declarative files, also knows as desired state, are stored in git/bitbucket/az repos.

3. Once we stored the desired state, GitOps opeartors, also knows as agents, automatically pull the desired state from Git and apply them in one or more k8s clusters. Popular gitops agents are ArgoCD and FluxCD.

4. Fourth principle is about reconciliation, The opeartor has a continuous loop to observe the git repository for any changes in the desired state. It compares the difference between the actual state to the desired state and acts automatically to reconcile both the states


**GitOps vs DevOps**

GitOps is focused on Delivery but the scope of DevOps is much broader as it can include CI, CD, visibility and monitoring.

One major diff btw them wrt to app deployment is.. 

We know in DevOps approach, Dev writes code > commits to git > trigger pipeline in Jenkins > Unit Testing > Building and Creating Docker Image > Push to Container Registry > Deploy to kubernetes.

Here the problem is in Jenkins it'll store the k8s cluster creds and leads to a compliance issue. The CI system has read and write access to k8s.

Within the GitOps approach everything is same till the Docker image push to container registry. Here we have 2 options

    Install a gitops agent on top of k8s and this agent scan and pull the container registries for new images
    [or]
    We can extend the pipeline to clone manifest files/repo, Update the k8s manifest, and raise PR. Once it's approved, the GitOps agent can pull updated manifests and apply them on the k8s cluster. 

    In both these cases, Jenkins doesn't need access to the k8s cluster, and hence eliminates the need to store k8s cluster credentials on it. 