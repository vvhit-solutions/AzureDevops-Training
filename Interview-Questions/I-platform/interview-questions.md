Interview questions 
Medium 
containerization /docker

1. how do convert a applciation to run in micro service environement ?

2 docker copy  vs docker add

3 how do you reduced docker attacking surface / how do you reduce vulenabilities

4 what is sourless vs sourcefull build in docker


 GCP
    *CLOUD ARCH

    5 exaplin the cloud architecture of your current working environments ?
    based on these arch ask 1 more question

    6 how do you connect two google cloud clients / other client who is hosted in other cloud

    7 what are the best practices you follow while designing  cloud infra in Azure  in terms of scaling, automation, security.

    8 how are you maintaining your client DR requirements in GCP.  caching mechanisms - I have a application where it will fetch large data its taking so much time when user is loading websites . what kind of measure will you take on infra level to reduce its app load speeds . 

autoscaling strategies : how do you handle peak traffic in black friday sales. 



   * IAM

    9 How do you provide lease privilege access for multi team environement.

    10 how do you provide CICD tool access run / provision resources in GCP

   11  how do you debug permission denied error in GCP


    *Kubernetes

    12 explain the Kubernetes architecture and explain what each component is responsible for what kind of action.

   13  write a deployment object with production ready best practices.
    (health checks, requests)

    14 how kubenertes allocate the deployment/pod based on requests or limits. what is the use of limits here.

    15 provide an example when to use init containers and write a simple example of init contaienrs.

    16 how do you upgrades kubenertes version as day 2 operations, what the process are you following.

    CRD vs Operator  what are all the restart policies available in k8s ? 


    *Deployment stratigies

    17 what are all differnt types of deployment stratgies you know, exaplain each one when to use in GCP.

    18 what are all Deployment stratgies you are using currently to deploy workfloads in k8s

    19 when to use canary deployments explain it with a scenario

    20 how do you deploy carary deployment using gke and istio




Terraform

21 write code for 2 vms where deployed in vent 1 sould have vnet other should not have internet access.

22 how do you work with mutilple clouds using terraform and maintain state for resources.

23 how do you design folder structure for you terraform code and how do you deploy them via automation / how do you test your PR.

24 i have multiple environments like dev, test, stg, prd how do you handle state for these application in google cloud

25 how do you authenticate your terraform workflows within AWS to provision the resources

26 write a for_each example in terraform with list object .

27 what are the all provisioners available in terraform.  how do you use provisiners in terraform.    monitoring & observability   1 . How do you monitor a Kubernetes cluster , what are all the imp metrics you look.   2. How to write queries for difference service like apim , app gw   3.  Lets say I have an app request which is taking more time than usual how do you troubleshoot using grafana.   scripting & automation :   1.  
2. What kind of automation you did in the past, explain the scenarios. 
3. Write a progrm to search for error in file  and count how many times it occurred. - python 
4. What all the libraries used and for which scenarios it is helpful . 
5. How to do you call a url from the powershelgl and get the result and response code ?    CI CD pipelins :    1. Explain the process of your current CICD approach , 


2. How are you maintaining different environment deployment via CICD pipelines, vars ? 
3. What kind of deployment strategies you are following in the CD . For k8s workloads. 
4. Environments vs deployment groups in AZURE. 

Cloud cost optimisation :   1. How do you optimise cost in cloud environment and what the measure will you take in order to save some costs. 




LLM/AI workloads. - any exposure on LLM / AI workloads.   what is LLM, how we are using it    stateless design applications vs stateful apps  ?  .    CI/CD   INFRASTRUCTURE AS CODE (IAC)  MONITORING & OBSERVABILITY  SCRIPTING & AUTOMATION  CLOUD SERVICES   KUBERNETES  AI workloads 


