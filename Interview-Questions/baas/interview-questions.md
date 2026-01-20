# L3 SysEng Interview questions

```
Networking Fundamentals & OSI Model
1. Can you walk me through the OSI model and explain the purpose of each layer in simple
terms?
2. At which OSI layers do switches, routers, and firewalls primarily operate?
3. What is the difference between Layer 3 routing and Layer 2 switching?
4. Explain the difference between TCP and UDP with real examples.
5. What happens when you type in a browser? Walk me through the flow.
Light Technical
6. What is a subnet mask, and how does subnetting work?
7. Can you explain CIDR notation? What does /24 mean? /30?
8. What is the difference between private vs public IP ranges?
9. What is NAT, and where is it typically used?
Routing
```
10. What is the purpose of a routing table?

11. How does a router decide which path to send a packet through?

12. What is the difference between static routing and dynamic routing?

```
Troubleshooting
```
15. If a VM cannot reach an external site but can ping internal resources, what routing issues
might you investigate?
16. What does traceroute tell you? How do you interpret it?

17. Why might traceroute show * * * for some hops?

```
Firewall & Security
```
18. What is the difference between a stateful and stateless firewall?

19. Explain what an ingress rule vs egress rule is.
20. What is a security group vs a network ACL (cloud context)?

21. How does port forwarding work?

```
http://www.example.com
```

```
Practical
```
22. A server is reachable via ping but the application is not loading on port 8080 — what firewall
checks would you do?
23. How do you debug if traffic is blocked: tcpdump command examples?

# VPN & IPSec

24. Explain what a VPN is and why companies use it.
25. What is the difference between site-to-site and client-to-site VPNs?
26. What is IPSec, and what problem does it solve?

27. What are the key components of IPSec (IKE, ESP, AH)?

28. What is Phase 1 vs Phase 2 negotiation in IPSec?

```
Scenario-Based
```
29. Your site-to-site IPSec VPN keeps dropping every few hours — what standard things do you
check?

30. If a VPN tunnel is up but traffic is not flowing, what could be the reason? (Hint: Phase
selectors, routes, firewall rules)

# Linux Troubleshooting Tools (ping, traceroute, tcpdump)

31. How would you use ping to diagnose packet loss?

32. If ping works but traceroute doesnʼt, what does that tell you?
33. Show me how youʼd capture traffic on port 443 using tcpdump.

34. What does this tcpdump output tell you? (Give a sample you can print)
35. How can you check DNS issues using dig or nslookup?

# Public Cloud Networking (Nice to Have)

36. What is a VNet/VPC?

37. What is a subnet inside a VNet/VPC?
38. What is a Network Security Group (NSG) and how is it different from a firewall?
39. What is peering? Can two peered VNets share a transitive network path?

40. What is a private endpoint, and when do you use it?

41. What is the difference between a load balancer and an application gateway?

42. Explain what Azure VPN Gateway or AWS VPN does at a high level.


## CI/CD + Git + GitHub Actions

Git SCM (Version Control Fundamentals)

1. What is the difference between Git and GitHub?

2. Can you explain how branching works in Git and why we use it?

3. What is the purpose of git rebase vs git merge? When would you use each?

4. What is a pull request (PR), and what steps usually happen before merging code?

5. How do you resolve a merge conflict? Walk me through the steps.

6. What is .gitignore? What type of files usually go into it?

7. Explain what Git tags are used for. What is the difference between lightweight and annotated tags?

CI/CD Concepts

8. What is CI/CD? Can you explain the difference between Continuous Integration, Delivery, and Deployment?

9. Why do teams use CI/CD pipelines instead of deploying manually?

10. Name common checks or steps you would expect in a CI pipeline.

E.g., linting, unit tests, build, security checks.

11. What is artifact management? What tools have you used or heard of?

GitHub Actions (Hands-On Focus)

12. What is a GitHub Actions workflow? What are jobs, steps, and runners?

13. How do you trigger a workflow? Can you name different trigger types (on: keywords)?

Expect: push, pull_request, schedule, workflow_dispatch, paths.

14. Explain the difference between self-hosted runners and GitHub-hosted runners. When would you use each?

15. Imagine your GitHub Actions pipeline fails at one step — how do you debug it? Whatʼs your approach?


IAC Terraform

Terraform Basics & Core Concepts (Beginner Level)

1. What is Terraform, and why do we use it instead of manual provisioning?

2. What is the difference between Terraform and tools like Ansible or CloudFormation?

3. Explain what “Infrastructure as Code (IaC)” means.

4. What are providers in Terraform? Can you name a few?

5. What happens when you run terraform init?

6. What is the purpose of terraform plan?

7. What is the difference between terraform apply and terraform destroy?

Variables, State, and Configuration (Intermediate Basics)

8. What is a Terraform state file? Why is it important?

9. What problems can occur if multiple people modify the same state file?

10. What is the purpose of terraform.tfvars and .auto.tfvars?

11. How do you define and use variables and outputs in Terraform?

12. What is a data source in Terraform? When would you use one?

## Modules & Structure (Average Level)

13. What is a Terraform module? Why should we use modules?

14. What is the difference between a root module and a child module?

15. Imagine you applied Terraform but the resource didnʼt change when you edited the code. What could be the
reason?

Evaluates troubleshooting skills:

```
Drift
Missing arguments
Lifecycle settings
Ignored changes
```

