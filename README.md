# terraform-aws-gitlab

### A quick proof of concept that leverages an [AWS ALB to authenticate users](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-authenticate-users.html "AWS Documentation") and [AWS IAM + SSM](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-getting-started-enable-ssh-connections.html "AWS Documentation") to allow for git ssh operations with no public port exposure with no extra infastructure (vpn/vpn-esque solutions). No custom code, no shims, just a little configuration and infrastructure glue.

#### Note: this example is just a proof of concept, there are many different applications for this pattern (i.e. non-public bastion hosts) and more scalable configurations (i.e. a single IDP using [temporary credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_request.html "AWS Documentation")). This seemed like a pretty simple and secure deployment maximizing the value of the gitlab and AWS teams, but I do not know everything, maybe I missed something.

---

## The Goals and the Flows

- **Goal**: a secure public endpoint for gitlab UI access that scales as the usage/users scale with little-none recurring infastructure management  
   **Solution**: AWS Cognito (or equivilent IDP) + AWS ALB  
   **Flow**:
  1. User intends to view gitlab web UI, heading to the some url via some web browser:
  1. User authenticates against the ALB who manages the auth flow with the IDP (cognito in this case)
  1. If successful, the User is redirected back to the gitlab IDP authentication endpoint (/users/auth/cognito/callback in our case) with the appropiate access token.
  1. gitlab is configured to use the IDP, negociates the token's validity, and allows the user access (creating an new gitlab account if needed)
  1. The User is now looking at the gilab web UI!
- **Goal**: a secure ssh solution that does not require instance/server security (i.e. fail2ban) ideally with no public exposure
  **Solution**: AWS SSH + AWS EC2  
   **Flow**:
  1. User intends to perform a git operation and has been indoctrinated into this system with a the appropiate ssh and aws configurations and has an active AWS token.
  1. User authenticates against AWS IAM and attempts to start a SSM session with the gitlab-hosting-ec2 instance
  1. If successful, user attempts to start an ssh session with gitlab
  1. If successful, the user has succeed in their git task!

---

## Prerequisits:

- [An AWS account](https://portal.aws.amazon.com/billing/signup "AWS Signup") - naturally we will need an AWS account to deploy into
- [Install terraform](https://learn.hashicorp.com/terraform/getting-started/install.html "Hashi Corp Guide") - nothing in life is perfect, but terraform is a great tool for this type of work
- [Set up your AWS User](https://blog.gruntwork.io/an-introduction-to-terraform-f17df9c6d180 "Good start up guide")
- [A Domain](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/registrar.html "AWS Documentation") - this example assumes the hosted zone for the domain is in the same AWS account we are deploying gitlab into
- Clone this repo :)

---

## Deploy

- core networking if needed
  - if you do not have a vpc you can run the network I provided. It will create:
    - A VPC
    - public subnet
    - private subnet
    - nat gatway
    - route tables/attachments
- gitlab + lb
  - creates:
    - public application load balancer
    - aws cognito user pool
    - acm cert for sub domain
    - target group
    - asg
    - encrypted volume
    -

---
