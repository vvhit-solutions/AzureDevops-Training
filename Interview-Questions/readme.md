# interview-questions

## version numbers for reference 
jenkins -  2.32 
Docker - 20.10
Kubernetes - 1.24 
Terraform - 1.37
Git - 2.37 windows

## Git branching strategies

[Ref Link](https://www.flagship.io/git-branching-strategies/)

### Git Flow Branch Strategy

- Main
- Develop
- Feature
- Release
- Hotfix

![gitflow](images/gitflow.png)

### Github Flow Branch Strategy

- Master
- feature

![githubflow](images/githubflow.jpeg)

### gitlab flow

- master 
- development
- production

![gitlabflow](images/gitlab.png)

### trunk based flow

- developers will work on their shared trunk and merge the code. 

![trunk](images/trunk.png)

How to choose the best branching strategy for your team ? 

| Product type and its release method	| Team size	| Collaboration maturity |	Applicable mainstream branch mode|
| ------------	| ----------	| ---------- |	--------------|
| All	| Small team |	High	|Trunk-Based Development (TBD) |
| Products that support continuous deployment and release, such as SaaS products |	Middle |	Moderate | 	GitHub-Flow and TBD |
| Products with a definite release window and a periodic version release cadence, such as iOS apps | Middle | Moderate | Git-Flow and GitLab-Flow with release branch|
|Products that are demanding for product quality and support continuous deployment and release, such as basic platform products	| Middle | Moderate	 | GitLab-Flow |
| Products that are demanding for product quality and have a long maintenance cycle for released versions, such as 2B basic platform products	| Large	| Moderate	| Git-Flow |

## Git

1. devloper made 100 commits in branch develop, only 5 commits are need in release brach ? 
    - git cherry-pick <COMMITSHA>

2. 100 commins in develop brach, push all commits as single commit in remote branch ? dont want history 
    - git sqaush 
    - git rebase -i HEAD~3

    ```
git checkout master
git merge --squash bugfix
git commit
    ```

3. dev has uncommited changes , he need to attend some urgent work . ?
    - git stash , git stash list , git stash pop , git stash drop 



    
4. dev 1 created a module and pushed to artifacts , now dev 2 has direct dependency on the module . how dev 2 can consumer the package?
    - azure access to feed
    - copy feed url and add it in vs code editor 

5. dev created asp.net app , he want do deploy app end-end by azure devops and deploy it using IAC. ?

6. dev created asp.net microservice based app. PAC as code ? 

7. we have multiple projects inside xyz repo , each have deployment file. 
    how to run specifi project for build 
    paths :
        include : 
        - abc/*
8. one build is dependency on other how to acive SEQUENTIAL build runs ?
    - on triggers option to select , one build after another
9. how to restrict developer to push messy comits into repositories ?
    - brach policies

    exmample : 
    Require a minimum number of reviewers
    Check for linked work items
    Limit merge types
    Build Validation
    Status Checks
    Automatically included reviewers

10. i have multiple commanly used variable , how to avoid giving it for each pipelines ? 
    - variable group
11. multiple commanly used pipeline tasks ? 
    - task groups in azure devops 
12. types of processess in Azure, - basic, srum CMMI , agile . how to add custom filed ?
    - through inherited process
    - org-> processs - > 
13. work items - user stories are not visible in boards tab ? 
    - AREA Path is different 
    - allows you group work items 
14. how do we enabled Event-driven notifications in azure devops and send them across as notifications on other messages 
    - service hooks 
15. how do we enable policy to auto approve some group for PR ?
    - branch policy 
16. what is difference between environments and deployment groups under pipelines ? 
   
In effect, a deployment group is just another grouping of agents, much like an agent pool


An environment is a collection of resources that can be targeted by deployments from a pipeline. Environments can include Kubernetes clusters, Azure web apps, virtual machines, databases.

17. deployment stratageies in azure devops ? 
    - Runonce
    - Rolling
    - canary
18. what is maximum size of commits and how do we restirct ? 
    - 5 GB at a time 
    - repo policies
19. can we recover deleted projects in azure devops ?
    - Yes, first 28 days from deletion 
    - org owner can restore , project collection administrator group 
    - view recenltly deleted projects in projects secrion under org
20. Diff between azure devops services and azure devops server ? 
    - one is public and another one is private and self hosted.

21. difference between git pull and git fetch ?

    git pull = git fetch + git merge

    Git Fetch is the command that tells the local repository that there are changes available in the remote repository without bringing the changes into the local repository.

22. git log vs git reflog ?

    “git reflog” is very beneficial for recovering deleted branches and commits,

    The most significant distinction between git reflog and log is that the log is a public record of the repository’s commit history, whereas the reflog is private. After a push, fetch or pull, the git log is duplicated as part of the git repository. The git reflog, on the other hand, is not included.

23. git reset vs git revert

    git reset is used when we want to unstage a file and bring our changes back to the working directory.
    and git reset can also be used to remove commits from the local repository.

    git reset HEAD <filename>
    
    git reset HEAD~1


    git reset –soft HEAD~1 – 
    
    This command will remove the commit but would not unstage a file. Our changes still would be in the staging area.
    
    git reset –mixed HEAD~1 or git reset HEAD~1 – 
    
    This is the default command that we have used in the above example which removes the commit as well as unstages the file and our changes are stored in the working directory.
    
    git reset –hard HEAD~1 – 

    This command removes the commit as well as the changes from your working directory. This command can also be called destructive command as we would not be able to get back the changes so be careful while using this command.

    *git revert is used to remove the commits from the remote repository*

24.  How do you resolve conflicts in Git?
    
    Here are the steps that will help you resolve conflicts in Git:
    
    Identify the files responsible for the conflicts.
    Implement the desired changes to the files 
    Add the files using the git add command.
    The last step is to commit the changes in the file with the help of the git commit command.

25. What is Git Bisect and how do you use it?
    
    The Git Bisect command performs a binary search to detect the commit which introduced a bug or regression in the project’s history.

    Syntax: git bisect <subcommand> <options>

        git bisect start

        git bisect good sha

        git bisect bad sha

        git bisect bad

        git bisect good

        ending the process by git bisect reset

26. How do you find a list of files that has been changed in a particular commit?
    The command to get a list of files that has been changed in a particular commit is:

    git diff-tree –r {commit hash}


27. What is the use of the git config command?

    The git config command is used to set git configuration values on a global or local level. It alters the configuration options in your git installation. It is generally used to set your Git email, editor, and any aliases you want to use with the git command.

28. how to ignore ssl errors in git push or git clone ? 

ignore errors by below command 

`git config http.sslVerify false`

29. accidentally pushed a secret to remote repo, how to remove ?

BFG Repo-Cleaner  :: bfg --replace-text passwords.txt  my-repo.git # passwords.txt contains the expoesed passdword list replaced as **REMOVED**
git filter-repo   :: git filter-repo --replace-text replacements.txt --replace-refs delete-no-add

#### for file : 

bfg --delete-files <filename> repo.git

git filter-repo --invert-paths --path <path-to-sensitive-file> --replace-refs delete-no-add
    
30. how to avoid this in the future ?
    
    Enable push protection on github secret scanning.

Avoiding Future Oopsies
-----------------------

As they say, _“Prevention is better than a million facepalms”_. So, to avoid future secret key blunders, here are some of my tips.

*   `**.gitignore**`**:** This file is a lifesaver. By listing file names or directories in `.gitignore`, you're telling Git to ignore them during commits.
*   **Environment Variables:** Instead of storing secrets directly in your code, you can use environment variables. This way, your code only contains references to them, not the actual sensitive values.

## Terraform 

## terraform commands 
```
Main commands:
  init          Prepare your working directory for other commands
  validate      Check whether the configuration is valid
  plan          Show changes required by the current configuration
  apply         Create or update infrastructure
  destroy       Destroy previously-created infrastructure

All other commands:
  console       Try Terraform expressions at an interactive command prompt
  fmt           Reformat your configuration in the standard style
  force-unlock  Release a stuck lock on the current workspace
  get           Install or upgrade remote Terraform modules
  graph         Generate a Graphviz graph of the steps in an operation
  import        Associate existing infrastructure with a Terraform resource
  login         Obtain and save credentials for a remote host
  logout        Remove locally-stored credentials for a remote host
  output        Show output values from your root module
  providers     Show the providers required for this configuration
  refresh       Update the state to match remote systems
  show          Show the current state or a saved plan
  state         Advanced state management
  taint         Mark a resource instance as not fully functional
  test          Experimental support for module integration testing
  untaint       Remove the 'tainted' state from a resource instance
  version       Show the current Terraform version
  workspace     Workspace management

Global options (use these before the subcommand, if any):
  -chdir=DIR    Switch to a different working directory before executing the
                given subcommand.
  -help         Show this help output, or the help for a specified subcommand.
  -version      An alias for the "version" subcommand.

```
1. Terraform end - to end workflow
2. one resource we need to recreate / redeploy 
- terraform state list
- terraform taint "resource_name"
- terraform apply 
3. secure .tfstate file anking to available to other devs in the team.?
    - store it in azure files as backend store.
4. explain various meta-arguments in terraform ?
- depends_on
- count
- for_each
- provider
- lifecycle
5. who creates the terraform.tfstate.backup  file and which scenareio its created. ?
- when you initiate command terraform destory 

6. what are benefits of using modules in terraform ?

instead of writing multiple times of same code, you can moduleise the code so that you can import it. 

```tf

module "webserver_cluster" {
  source = "../../../modules/services/webserver-cluster"
  cluster_name           = "webservers-prod"
  db_remote_state_bucket = "(YOUR_BUCKET_NAME)"
  db_remote_state_key    = "prod/data-stores/mysql/terraform.tfstate"
}
```

7. what is private module registry ?

same like docker registry, terraform provides module registry to store your terraform files in registry.

8. what are all provisioners in terraform ?

with the help of provisioners , you can configure either local or remote server. 

some keywords are **connection**, local-exec , remote-exec ,  , file provisioner , 

```
resource "aws_instance" "web" {
  # ...

  # Copies the myapp.conf file to /etc/myapp.conf
  provisioner "file" {
    source      = "conf/myapp.conf"
    destination = "/etc/myapp.conf"
  }

```

9. what are data sources in terraform ? 

as name suggests, its used to get data outside of terraform using data source block. 

```
# Documentation Reference: https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/data-sources/kubernetes_service_versions
# Datasource to get Latest Azure AKS latest Version
data "azurerm_kubernetes_service_versions" "current" {
  location = azurerm_resource_group.aks_rg.location
  include_preview = false
}

```

10 . how can two people using the terraform cloud can create 2 different sets of infrastructre using same working directory ?

using workspace 

`terraform workspace list`

`terraform workspace create new-test`

`terraform workspace select default`


11. what are life cycle policy in terraform ?

lifecycle policy overrides default behaviour of terraform. 

some of lifycycle policy or *create_before_destroy* , *prevent_destory*, *ignore_changes*, *replace_triggeredby*

12. terraform refresh vs import ?

refresh will track the drift in real world  infrastructre , if someone modified changes in infra manullay we can track by refresh command 

import will import exisiting resources in terraform state file 
 - write respective terraform file  with empty details

`terraform import azurerm_resource_group.late-motiv-rg /subscriptions/8c247875-6978-4ff9-899b-2aab2f4h7443/resourceGroups/late-motiv-rg `


13. sample terraform example tf file ?

```
# Terraform Block
terraform {
  required_version = ">= 0.15"
  required_providers {
    azurerm = {
      source = "hashicorp/azurerm"
      version = ">= 2.0" 
    }
  }
}

# Provider Block
provider "azurerm" {
 features {}          
}
# Resource-1: Azure Resource Group
resource "azurerm_resource_group" "myrg" {
  for_each = {
    dc1apps = "eastus"
    dc2apps = "eastus2"
    dc3apps = "westus"
  }
  name = "${each.key}-rg"
  location = each.value
}

```

14. how do you handle secrets in terraform ?

1. environmnet variable 
example 

export TF_VAR_username = 'username'
export TF_VAR_pass = 'pass'

2. use azure keyvault 
3. use terraform vault 

15. what is resource graph in terraform ?

internal of terraform , dependency graph

16. terraform state rollback ? 

terraform by default does not have any, use some kind of version control system to roll back. 

17. can we use one project state file outputs in other project ?

yes, we can use data source as othe project state file and use in project.

18. terrafrom init failed with plugin outof date , provide unable to sync , how to trouble shoot ? 

`terraform init -upgrade`

19. you 've written terraform file but the syntax and format issue is there how do you resolve it. ?

`terraform fmt `
 
20. how to get more debug information for terraform for debug purpose and azure pipelines ? 

Terraform 

Terraform has detailed logs that you can enable by setting the **TF_LOG** environment variable to TRACE, DEBUG, INFO, WARN or ERROR. Enabling this setting causes detailed logs to appear on stderr.


Azure pipelines

for azure pipelines while running the pipeline, on pop up screen enable **Enable system diagnostics** or 

To configure verbose logs for all runs, you can add a **variable** named **system.debug** and set its value to true


## Docker scenareio 
    
```
C:\Users\2224741>docker

Usage:  docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default
                           "C:\\Users\\2224741\\.docker")
  -c, --context string     Name of the context to use to connect to the
                           daemon (overrides DOCKER_HOST env var and
                           default context set with "docker context use")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level
                           ("debug"|"info"|"warn"|"error"|"fatal")
                           (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default
                           "C:\\Users\\2224741\\.docker\\ca.pem")
      --tlscert string     Path to TLS certificate file (default
                           "C:\\Users\\2224741\\.docker\\cert.pem")
      --tlskey string      Path to TLS key file (default
                           "C:\\Users\\2224741\\.docker\\key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  builder     Manage builds
  buildx*     Docker Buildx (Docker Inc., v0.9.1)
  compose*    Docker Compose (Docker Inc., v2.10.2)
  config      Manage Docker configs
  container   Manage containers
  context     Manage contexts
  extension*  Manages Docker extensions (Docker Inc., v0.2.9)
  image       Manage images
  manifest    Manage Docker image manifests and manifest lists
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  sbom*       View the packaged-based Software Bill Of Materials (SBOM) for an image (Anchore Inc., 0.6.0)
  scan*       Docker Scan (Docker Inc., v0.19.0)
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.

To get more help with docker, check out our guides at https://docs.docker.com/go/guides/


```

1. difference between dockerfile and docker-compose ? 
    - docker file insruction to create an image.
    - docker-compose - used to run multi containers
2. container exiting  automatically 
    docker run -it ubuntu
    docker inspect
    docker run ubunut sleep 10 -- overide entry point / CMD
3. how do you run nginx on localhost:8088 using bridge network ? 
    - docker run -itd -p 8088:80 --name web_server nginx
4. explain how to make volume persistent ? and how do we mount the volume on the localhost "/root/nginx" directory ? Not in the default directory (/var/lib/volumes) ?
    - trough bind mount 
    - docker run -itd -p 8088:80 --name web_server nginx --mount type=bind,souce =,target=
5. company has restriction on pulling or pushing from docker or any cloud provider , how to setup local registry ? 
    - run local docker registry container.

6. how to export docker image as an archive?

`docker save -o export_name.tar image_name`

import by 

`docker load -i export_name.tar`

7. differnce between copy and add commands ?

copy will take copy files from local 

add has additional features to copy files from url or tar 

8. can container restart by itself ?

yes, by restart policy 

1. off
2. always
3. on-failure 
4. unless-stopped

`docker run -itd --restart [always] --name web1 nginx`

/var/lib/docker/volumes

9. commanly used instruction in dockerfile ? 

FROM 
WORKDIR
COPY 
EXPOSE
ENTRYPOINT 
CMD 
USER
RUN 

10. differnce between CMD and ENTRYPOINT  ? 


CMD and ENTRYPOINT are both used to specify the command that should be run when a container is started from the image, but they are used in slightly different ways.

CMD is used to specify the default command that should be run when a container is started from the image. If no command is provided when the container is started, the command specified in CMD will be used.

ENTRYPOINT is used to specify the command that should always be run when a container is started from the image. This command cannot be overridden when the container is started.

For example, if you have an image with an ENTRYPOINT of "python" and a CMD of "main.py", running the command "docker run my_image" will start a container and run the command "python main.py". However, running the command "docker run my_image arg1 arg2" will start a container and run the command "python main.py arg1 arg2", because CMD is overriden by the run command arguments.

In summary, CMD is used to specify default command and ENTRYPOINT is used to specify the command that should always be run when the container starts.

Ex

CMD ['sleep', 10]



ENTRYPOINT['sleep']
CMD 10

11. differnce beetween docker run and docker create ?

docker run create a container and container state will run 

docker create , creates a container in stopped state.

12. what is memory swap flag ?

when container used all its memory it can use some of the disk as memory 

13. different types of mounts in docker ? 

1. volume mount
2. bind mount
3. tmpfs mount

14. 

## Kubernetes
 
```
C:\Users\2224741>kubectl
kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/

Basic Commands (Beginner):
  create          Create a resource from a file or from stdin
  expose          Take a replication controller, service, deployment or pod and
expose it as a new Kubernetes service
  run             Run a particular image on the cluster
  set             Set specific features on objects

Basic Commands (Intermediate):
  explain         Get documentation for a resource
  get             Display one or many resources
  edit            Edit a resource on the server
  delete          Delete resources by file names, stdin, resources and names, or
by resources and label selector

Deploy Commands:
  rollout         Manage the rollout of a resource
  scale           Set a new size for a deployment, replica set, or replication
controller
  autoscale       Auto-scale a deployment, replica set, stateful set, or
replication controller

Cluster Management Commands:
  certificate     Modify certificate resources.
  cluster-info    Display cluster information
  top             Display resource (CPU/memory) usage
  cordon          Mark node as unschedulable
  uncordon        Mark node as schedulable
  drain           Drain node in preparation for maintenance
  taint           Update the taints on one or more nodes

Troubleshooting and Debugging Commands:
  describe        Show details of a specific resource or group of resources
  logs            Print the logs for a container in a pod
  attach          Attach to a running container
  exec            Execute a command in a container
  port-forward    Forward one or more local ports to a pod
  proxy           Run a proxy to the Kubernetes API server
  cp              Copy files and directories to and from containers
  auth            Inspect authorization
  debug           Create debugging sessions for troubleshooting workloads and
nodes

Advanced Commands:
  diff            Diff the live version against a would-be applied version
  apply           Apply a configuration to a resource by file name or stdin
  patch           Update fields of a resource
  replace         Replace a resource by file name or stdin
  wait            Experimental: Wait for a specific condition on one or many
resources
  kustomize       Build a kustomization target from a directory or URL.

Settings Commands:
  label           Update the labels on a resource
  annotate        Update the annotations on a resource
  completion      Output shell completion code for the specified shell (bash,
zsh, fish, or powershell)

Other Commands:
  alpha           Commands for features in alpha
  api-resources   Print the supported API resources on the server
  api-versions    Print the supported API versions on the server, in the form of
"group/version"
  config          Modify kubeconfig files
  plugin          Provides utilities for interacting with plugins
  version         Print the client and server version information

Usage:
  kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all
commands).

    
```

1. provide a command to run nginx pod ?

    az aks get-credentials --resource-group aks-rg1 --name aksdemo1
    
    kkubectl run my-first-pod --image nginx

2. explain kubernetes Components /Architecture  ? 

  k8 will have control-plane, node-plane 
  in control plane we have
    - **etcd**                  : key-value store for k8s to store all details   
 
  - **kube-api-server**       : API server validates and configures data for the api objects which include pods, services, replicationcontrollers, and others.   

  - **kube-controller**       : it has many control componenets, 
      *Node controller*: Responsible for noticing and responding when nodes go down.
      *Job controller*: Watches for Job objects that represent one-off tasks, then creates Pods to run those tasks to completion.
      *EndpointSlice controller*: Populates EndpointSlice objects (to provide a link between Services and Pods).
      *ServiceAccount controller*: Create default ServiceAccounts for new namespaces
 
  - **kube-schduler**        : watches for newly created Pods with no assigned node, and selects a node for them to run on.
  - **cloud-controller** :  for cloud platforms
  in Node-plane, we have 
  - **kubelet**          :   An agent that runs on each node in the cluster. It makes sure that containers are running in a Pod.
  - **kube-proxy**        :   maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.
  - **Container-runtime** :  container runtime is the software that is responsible for running containers. like docker, containerd, CRI-O 

2. you dont know the yaml syntax for a pod , deployment , use kubectl internal commands to create a yaml file for pod
 
    - Create an NGINX Pod

        `kubectl run nginx --image=nginx`

    - Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

        `kubectl run nginx --image=nginx --dry-run=client -o yaml`

    - Create a deployment

        `kubectl create deployment --image=nginx nginx`

    - Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

        `kubectl create deployment --image=nginx nginx --dry-run=client -o yaml`

    - Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)

        `kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml`

    Save it to a file, make necessary changes to the file (for example, adding more replicas) and then create the deployment.

    `kubectl create -f nginx-deployment.yaml`

    OR

    > In k8s version 1.19+, we can specify the --replicas option to create a deployment with 4 replicas.

    `kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml`

3. Difference between Deamonset and replicaset .

- replicaset will maintain number of replicas given irrespective nodes
- deamonset will maintain atleast one pod in all nodes, used for networking or monitoring

4. Difference between replicaset and replication-Controller ? 

main difference is **selector** support , replicaset provides set based selector example enviroment in (qa, dev)

###replication controller

```yml
apiVersion: v1 # String
kind: ReplicationController  # String
metadata: # Dictionary
  name: nginx       
spec:
  replicas: 3
  selector: 
    app: nginx
```

###ReplicaSet

```yml
apiVersion: apps/v1 # String
kind: ReplicationSet  # String
metadata: # Dictionary
  name: nginx       
spec:
  replicas: 3
  selector: 
    matchLables:
        tier: nginx
    or 
    matchExpressions:
        {key: app, operation: In, values: [nginx, frontend]}
```


5. write a simple pod defination and expose it outside? 

```yml
apiVersion: v1 # String
kind: Pod  # String
metadata: # Dictionary
  name: myapp-pod
  labels: # Dictionary 
    app: myapp         
spec:
  containers: # List
    - name: myapp
      image: stacksimplify/kubenginx:1.0.0
      ports:
        - containerPort: 80
```

## Create a LoadBalancer Service

```yml
apiVersion: v1
kind: Service
metadata:
  name: myapp-pod-loadbalancer-service  # Name of the Service
spec:
  type: LoadBalancer
  selector:
  # Loadbalance traffic across Pods matching this label selector
    app: myapp
  # Accept traffic sent to port 80    
  ports: 
    - name: http
      port: 80    # Service Port
      targetPort: 80 # Container Port
```

6. what are all type of services availble in K8s.?

in k8s, we have 
- clusterIp
- LoadBalancer
- Nodeport
- external

7. what are namespaces in k8s. ?

namespaces enables us to segregate some resources assign some access policies .
example : kube-system , kube-public

8. difference between lables   vs annotatinos vs selectors? 

lables and annotations are key value pairs

lables - used for identification purpose on kubernetes like app : myapp-pod , in service selector use this lable to pick the pod.

Annotations : these are for normal documentation like author, version 


selectors: 

environment = production
tier != frontend

environment in (production, qa)

kubectl get pods -l 'environment in (production),tier in (frontend)'


9. deployed a wrong version of application , i would revert my deployment what is the command to revert ?

`
###check history
kubectl rollout history deployment/my-first-deployment`


kubectl rollout undo deployment my-first-deployment



10. i would like to go to particular deployment revision in deployment/my-first deployment , what is the command to achive this ?

`kubectl rollout undo deployment my-first-deployment --to-revision=2`


11. set image without changing the yaml file ? 

`kubectl set image deployment/my-first-deployment nginx=nginx:v2 --record=true`

>to edit use 
>kubectl edit

12. i need to make multiple changes to the deployment by stopping it , what is the best way to do that ?

using pause and resume deployment

`kubectl rollout pause deployment my-first-deploymentment`

`kubectl set image deployment/my-first-deployment nginx=nginx:v2 --record=true`

`kubectl set resources deployment/my-first-deployment -c=kubenginx --limits=cpu=20m,memory=30Mi`

`kubectl rollout resume deployment/my-first-deployment`

13. provide the service types in k8s ?

    1. ClusterIp
    2. nodeport service
    3. LoadBalancer
    4. External

14. the application has a requirement to use azure file or azure disk service how do you achieve this in k8s ?

we can user PV and PVC to use either azure file or azure disk service. 


- storage classes 
Azure files, azure disks
- reclaim policy 

15. we want a pod to use specific node only in that Node, we need to deploy if in a case if that node is not available then it should take availble node to deploy

*preferredDuringSchedulingIgnoredDuringExecution* Node Affifnity policy 

to select particular node use NodeSelector 

16. i want a pod to schedule a special hardware  ? 

use taints and tolarations 

`kubectl taint nodes node1 key1=value1:NoSchedule`

remove taint 

`kubectl taint nodes node1 key1=value1:NoSchedule-`

```yml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
```

17. what are static pods ?

we can schedule pods without control planin using kubelet  on node plane. 

keep the pod definition file in the directory of */etc/kubernetes/manifests* 

and restart kubelet  pod will be created on the node.

`systemctl restart kubelet`

18. check the cpu and memory of nodes ?

`kubectl top nodes`

`kubectl top pods`

19 . difference between configmaps and secrets ?

configmaps are used to store non-confidential data like sql schema creation initial statge of details.

secrets encoded in base64 format used for secret storing but , its recommended to use keyvault or terraform vault for secret store in k8s.

20. how do use environment variables in k8s ?

these are key-value pairs .

env: 
    - name: APP_COLOR
      value: blue

env: 
    - name: APP_COLOR
      valuefrom:
        configMapKeyRef:

21.difference between docker and kubernetes ?

Docker is container platform
k8s is container orcestration platoform

22. differnce between docker swarm and kubernetes ?

both are container orchestration platforms 

k8s has lot of features when compare to docker swarm advanced load balacing, k8s apps are tightly integrate with cloud platform.

23. Differnce between deployment , statefulset, deamonset ?

deployment

deployment will consisit of replicaset and pods.

stateful set - where app need to maintain its state 
    when app is down , the new app should come up with same data

daemonset - ensures pods are deployed on all nodes, when new nodes comes it deploy pod automatically

24. what is ingress in k8s ? 

it enables us to define traffic rules and advanced context based routing 

25. we are doing a maintenance on kubernetes node, how do plan and what are all the steps involved. ?

we need to drain the node using 

`kubectl drain node node01`
`kubectl uncordon node01`

26. what are init containers and when do you use it ?

init containers are which starts before app containers are started.

example, DB service need to up then after app container need to UP.

```yml

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```


27. roles, rolebindings, clusterRole, clusterRole bindings ?

Role : A Role always sets permissions within a particular namespace
CluserRole : ClusterRole, by contrast, is a non-namespaced resource. 

```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

```yml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
  #
  # at the HTTP level, the name of the resource for accessing Secret
  # objects is "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```


A ==role binding== grants the permissions defined in a role to a user or set of users.

```yml

apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```
A ==RoleBinding may== reference any Role in the same namespace. Alternatively, a RoleBinding can reference a ClusterRole and bind that ClusterRole to the namespace of the RoleBinding


```yml
apiVersion: rbac.authorization.k8s.io/v1
# This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io

```

28. service accounts ?

    service accounts are used for dashboard pupose or monitoring purpose 

    it has some token , the token stored in secret object.

29. image security ?

30. how do to get current context ?

`kubectl config view`
`kubectl config current-Context`
`kubectl config use-context othername`

31. difference between liveness and readiness probe ?

*liveness probe* 

liveness probes to know when to restart a container. 

For example, liveness probes could catch a deadlock, where an application is running, but unable to make progress. Restarting a container in such a state can help to make the application more available despite bugs.
```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```


*readiness probes* to know when a container is ready to start accepting traffic

For example, an application might need to load large data or configuration files during startup, or depend on external services after startup.

>Note: Readiness probes runs on the container during its whole lifecycle.

>Caution: Liveness probes do not wait for readiness probes to succeed. If you want to wait before executing a liveness probe you should use initialDelaySeconds or a startupProbe.

```yml
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

Readiness and liveness probes can be used in parallel for the same container. Using both can ensure that traffic does not reach a container that is not ready for it, and that containers are restarted when they fail


32. how to integrate azure keyvault with azure kubernetes service ? 

portal option : 

[*] Enable secret store CSI driver


**Step 1**. First, enable the add-on for Azure Key Vault. This will install the Secrets Store CSI Driver for Key Vault.

`az aks enable-addons --addons azure-keyvault-secrets-provider --name name_of_your_aks_cluster --resource-group resource_group_name`

**Step 2** . Verify that the Secrets Store CSI Driver is running by reviewing the Pods that are created for CSI

`kubectl get pods -n kube-system -l 'app in (secrets-store-csi-driver, secrets-store-provider-azure)'`

**step 3**. Enable system managed user identiy 

`az aks update -g <RGName> -n <AKSName> --enable-managed-identity`

In the output, you'll see an identityProfile {} block. The identity profile block contains the clientId which you'll need for the next step.

**step 4**. grant permissions that provide read access from AKS to Azure Key Vault.
```
az keyvault set-policy -n <keyvault-name> --key-permissions get --spn <client-id>
az keyvault set-policy -n <keyvault-name> --secret-permissions get --spn <client-id>
az keyvault set-policy -n <keyvault-name> --certificate-permissions get --spn <client-id>
```

**step 5** The last step is to create a **SecretProviderClass** that uses a system-assigned identity to access Key Vault. Save the following YAML as *secret.yaml*:


```yml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-keyvault-system-identity
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"
    useVMManagedIdentity: "true"
    userAssignedIdentityID: "" # defaults to use the system assigned identity on the AKS cluster
    keyvaultName: <key-vault-name>
    objects:  |
      array:
        - |
          objectName: secret1 # Name of the secret in your Key Vault that you want to use in AKS
          objectType: secret
    tenantId: <tenant-id> # Tenant ID of the Key Vault you created
```

`kubectl apply -f secret.yaml`

33. how to integrate azure container registry with AKS ?

we use --attch-acr command and 

az aks update -n myAKSCluster -g myResourceGroup --attach-acr <acr-name>

34. how to integrate azure app gateway with AKS ?

we need to Enable AGIC addon (Azure Gateway ingress controller) on K8s cluster.

35 . what is open service mesh ? 

link : https://learn.microsoft.com/en-us/azure/aks/open-service-mesh-about


enable add on using portal or CLI or Bicep. 

feature :

Encrypt communications between service endpoints deployed in the cluster.
Enable traffic authorization of both HTTP/HTTPS and TCP traffic.
Configure weighted traffic controls between two or more services for A/B testing or canary deployments.
Collect and view KPIs from application traffic.


### JENKINS groovy

write a sample jenkins groovy 

```
pipeline {
    agent any

    stages {
        stage('hello message') {
            steps {
                sh 'echo hello'   
            }
        }
    }
}
```

Azure pipelines yaml example 

```
trigger :
 - master

pool:
  vmImage: 'ubuntu-latest'

steps : 
- script: echo "Hello World"
  displayName: 'Print Hello World'
```


## Azure Concept questions

1. hub and spoke architecture
2. managed identity vs service pricincipal
3. loadbalancer, traffic manager , Application gateway, Azure Front door
4. Explain CI CD flow for the application
5. explain IAC - (ARM )strategy deployment using Azure DevOps
6. how to troubleshoot pipeline to get more info when pipeline failed.

## terraform

null resource
local exec 
remote exec 
comman patterns in terraform
for each set -- select specific value 
run , cmd command 
multi staging in docker 
ns <--> ns connection 
cordon



# Helm

| Command                                            | Scenario                                          | Example                                                    |
| -------------------------------------------------- | ------------------------------------------------- | ---------------------------------------------------------- |
| `helm repo add <name> <url>`                       | Add a Helm chart repo (e.g., Bitnami, Prometheus) | `helm repo add bitnami https://charts.bitnami.com/bitnami` |
| `helm repo update`                                 | Refresh local repo cache after adding repos       | `helm repo update`                                         |
| `helm search repo <keyword>`                       | Find charts inside added repos                    | `helm search repo nginx`                                   |
| `helm install <release> <chart>`                   | Install a new Helm release                        | `helm install myapp bitnami/nginx`                         |
| `helm install <release> <chart> -f values.yaml`    | Install using custom values                       | `helm install myapp bitnami/nginx -f values.yaml`          |
| `helm install <release> <chart> --set key=value`   | Override chart values inline                      | `helm install myapp bitnami/nginx --set replicaCount=3`    |
| `helm list`                                        | List installed releases in current namespace      | `helm list`                                                |
| `helm list -A`                                     | List all releases across namespaces               | `helm list -A`                                             |
| `helm status <release>`                            | Check status/details of a release                 | `helm status myapp`                                        |
| `helm history <release>`                           | View all revisions of a release (for rollback)    | `helm history myapp`                                       |
| `helm upgrade <release> <chart> --set key=value`   | Upgrade an app with new config                    | `helm upgrade myapp bitnami/nginx --set image.tag=latest`  |
| `helm rollback <release> <revision>`               | Roll back to previous working version             | `helm rollback myapp 2`                                    |
| `helm uninstall <release>`                         | Uninstall a Helm release                          | `helm uninstall myapp`                                     |
| `helm template <release> <chart>`                  | Render manifests without applying                 | `helm template myapp bitnami/nginx`                        |
| `helm install <release> <chart> --dry-run --debug` | Test install without deploying                    | `helm install testapp bitnami/nginx --dry-run --debug`     |
| `helm pull <chart> --untar`                        | Download a chart locally for inspection           | `helm pull bitnami/nginx --untar`                          |
| `helm show values <chart>`                         | Show default values of a chart                    | `helm show values bitnami/nginx`                           |
| `helm get values <release>`                        | Show effective values of a deployed release       | `helm get values myapp`                                    |
| `helm dependency update`                           | Download chart dependencies (for umbrella charts) | `helm dependency update ./mychart`                         |

