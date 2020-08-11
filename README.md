# Deployment of the phonebook application

___________________________________________________

1. **Context**
   
   * Deployment of the phonebook application through the CI / CD
   
   * Implementation of the security aspect
   
   * Notification system
   
   * Monitoring of the application and servers in real time

2. **Used tools**
   
       * Amazon EC2                                       
       * Docker
       * Jenkins 
       * Ansible                                         
       * Jfrog artifactory
       * Zabbix                                           
       * Clair                                           
       * Gauntlt

3. **Infrastructure**
   
   ![image infra](C:\Users\Fujitsu\Desktop\infra.png)

All servers are deployed on **AWS**.
We have on our infrastructure:

- **zabbix server:**
  it will allow you to supervise the build, pre-production and production servers in order to prevent possible failures and thus guarantee the availability of the application.

- **Ansible and Jenkins server:**
  
  This server is the main one. Thanks to Ansible, the application will be deployed automatically on the servers and thanks to Jenkins, we will be able to orchestrate this deployment.

- **private register:**

The goal here is to store and share our docker images. we used **artifactory** which is very interesting. In addition to storing images, it allows to share several types of artifacts and to store artifacts by project called *Repository Key*.

- **build, preprod and production servers:**

They will be used respectively for the build of images, for deployments in test and production environments.

4. **Workflow**

![Workflow](C:\Users\Fujitsu\Desktop\workflow.png)

It consists of two environments: recet and procuction
In order to fully understand this workfow, let's take the following scenario:

- Developer makes a modification to the code and pushes it on github

- Thanks to the webhook, the modification is received on the jenkins server and the build of the project can begin

- Syntax checks will be done (unit tests)

- We will have the build of the docker images and pushed on the private **Jfrog artifactory** registry. follow this link to deploy artifactory:
  [https://github.com/franck-art/artifactory](https://github.com/franck-art/artifactory)

- Deployment in a test environment (pre-production) can begin by pulling images from the private register and a notification is sent to slack.

- The modification validated by the development team, a **PR** will be carried out in order to share the modification to the Operational (Ops)

- After validation by the whole team, the Ops manager can make the **merge request** in order to pass the modification on the master branch.

- Deployment in production environment will then be activated and a notification is sent to slack

- Once the application is deployed, a user will be able to connect and consume the application.

- In order to prevent possible breakdowns and always guarantee the availability of servers, a monitoring solution has been deployed using **zabbix**. follow this link to deploy zabbix: [https://github.com/franck-art/zabbix-server-playbook] (https://github.com/franck-art/zabbix-server-playbook)

## Keywords

```
zabbix-server, ansible, jenkins, AWS, Jfrog artifactory, Jmeter, Clair, Gauntlt, Slack, Pull Request, Merge Request
```

## References

* https://github.com/franck-art/zabbix-server-playbook 

* https://github.com/franck-art/artifactory

* https://www.zabbix.com/documentation/4.0/fr/manual

* https://jfrog.com/download-jfrog-platform/
