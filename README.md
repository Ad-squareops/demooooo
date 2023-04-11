# Demo Setup

Convert this doc to readme 
Handle the case where domain name needs to be changed . Use tfvars if needed 


# Prerequisites
Only for setting up a demo in a new AWS Account. This has already been configured in aws-skaf account us-east-1 region


1. AWS user with Admin Access
2. ECR Repository for application images.
3. DNS/Route53 Hosted zone(Update here https://github.com/sq-ia/skaf-demo/blob/d0be2a1e66f86a4eb8433737f1754b676545dd75/k8s/main.tf#L23 )
4. Application secrets in AWS Secret Manager 
5. Access to Squareops Terraform Cloud of Skaf Demo Project in squareops-technologies organization


# Estimated time for this activity 
Min : 2h 
Start the setup 6h before the demo . We will have significant data in logs and metrics services. Also we will have sufficient time to fix in case of any unexpected issues for Demo Setup

# Deployment with terraform cloud
Workspaces  are pre-configured in Terraform Cloud. 
* Terraform Cloud Organization:  squareops-technologies
* Project: Skaf Demo


Apply the 3 modules(workspaces) in following order


1. Skaf-demo-01-aws (https://app.terraform.io/app/squareops-technologies/workspaces/skaf-demo-01-aws)
This creates the AWS VPC , EKS cluster and bootstraps the cluster 
Navigate to module → Actions→ Start new run
  
NOTE: Steps(modules) 2 and 3 can be triggered in parallel as they are independent from each other[a]

2. Skaf-demo-02-k8s (https://app.terraform.io/app/squareops-technologies/workspaces/skaf-demo-02-k8s)
This execution deploys management services inside the EKS cluster
* Rancher
* Jenkins
* Sonarqube
* ArgoCD
* Grafana
* ELK Stack


Note the output values of K8s module. These values should be used to login into various service endpoints mentioned below.
1. Rancher: rancher.demo.skaf.squareops.in
2. Jenkins: jenkins.demo.skaf.squareops.in
3. ArgoCD: argo.demo.skaf.squareops.in
4. Kibana: kibana.demo.skaf.squareops.in
5. Grafana: grafana.demo.skaf.squareops.in
6. Sonarqube: sonar.demo.skaf.squareops.in


Configure Services using Manual Steps
Now that deployment is completed for all stacks , we need to configure certain services manually 
This includes
1. Adding jenkins webhook to sonarqube
2. Configure sonarqube connection in Jenkins ( The pipeline jobs configuration is restored from pre-defined backup as a part of terraform runs already ) 
3. Configurations in Kibana 


Sonarqube
Login to Sonarqube and perform following steps (http://sonar.demo.skaf.squareops.in)
1. Create token and save for later use:- 
go to profile > my account > security > Generate token
Token name: SonarqubeScanner 

2. Update Jenkins (http://jenkins.demo.skaf.squareops.in) Credential sonar with Sonarqube token created in 
Step 1.Go to  Manage Jenkins > Manage Credentials > Click on Sonar > Update Password

3. Create webhook

In main menu bar go to Administration>Configuration>webhooks>create new
Webhook name: SonarqubeScanner
  
Provide Webhook Name and URL as  jenkinsurl/sonarqube-webhook/. In this case https://jenkins.demo.skaf.squareops.in/sonarqube-webhook/ . Do not provide any secrets here.



# Jenkins
Login to Jenkins (http://jenkins.demo.skaf.squareops.in) and ensure Jenkins pipelines and Credentials are present. 
  
If pipelines and credentials are not present, delete Jenkins pod from  Rancher . The pod will be automatically re-created and pipelines will reflect in Jenkins dashboard after refreshing in dashboard. This happens when the restored backup is delayed and jenkins reload is needed.

Verify sonarqube configuration in Jenkins
1. Go to manage jenkins>configure system-> SonarQube Installations . verify the Sonarqube url is set correctly as per the demo urls: https://sonar.demo.skaf.squareops.in

2. Go to Manage Jenkins> Global Tool Configuration>sonarqube scanner and verify the correct version present.


  

# Grafana
Star important dashboards


1. Kubernetes / Compute Resources / Cluster
2. Kubernetes / Compute Resources / Node (Groups)
3. Kubernetes / Compute Resources / Workload
4. Kubernetes / Networking / Cluster
5. Kubernetes / Persistent Volumes
6. Kubernetes Nginx Ingress Prometheus NextGen
7. MongoDB Overview
8. MySQL Overview
9. Node Exporter / Nodes
10. Prometheus Blackbox Exporter
11. RabbitMQ-Overview
12. Redis Dashboard


Verify the data in starred dashboards. Disable auto-refresh and set the time duration to last 6h 



# Kibana
Follow ECK documentation for snapshot, dashboard and home page ECK https://docs.google.com/document/d/1jJziJN_FMunpY5t87z8D-Q3VMNV2FnN96AmyrxpN9FM/edit?usp=sharing 


# Application Deployment
3. skaf-demo-03-app (https://app.terraform.io/app/squareops-technologies/workspaces/skaf-demo-03-app)
This job deploys the sample app ( roboshop ) to running cluster. Predefined endpoint is [d]
Deploy the load test service also to demonstrate system under nominal load

Add DB Dump
MongoDB
1. Execute Shell in mongodb-0 pod using Rancher (http://rancher.demo.skaf.squareops.in)
  

Execute following commands in the shell:
2. Run command in mongo db shell

curl -o /tmp/catalogue.js https://raw.githubusercontent.com/instana/robot-shop/master/mongo/catalogue.js 

curl -o /tmp/users.js https://raw.githubusercontent.com/instana/robot-shop/master/mongo/users.js

mongo --host mongodb-headless.mongodb.svc.cluster.local -u root -p --authenticationDatabase admin < /tmp/catalogue.js

mongo --host mongodb-headless.mongodb.svc.cluster.local -u root -p --authenticationDatabase admin < /tmp/users.js

1. Enter password (get password from AWS Secrets Manager’s secret dev/squareops/mongodb)


I have no name!@mongodb-0:/$ mongo --host mongodb-headless.mongodb.svc.cluster.local -u root -p  neni5xFCLUBJ0Ui0pfor --authenticationDatabase admin < /tmp/users.js
MongoDB shell version v5.0.8
connecting to: mongodb://mongodb-headless.mongodb.svc.cluster.local:27017/?authSource=admin&compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("e161306c-0236-4ba9-9954-126cca6d69eb") }
MongoDB server version: 5.0.8
================
Warning: the "mongo" shell has been superseded by "mongosh",
which delivers improved usability and compatibility.The "mongo" shell has been deprecated and will be removed in
an upcoming release.
For installation instructions, see
https://docs.mongodb.com/mongodb-shell/install/
================
users
{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("643030cb121f565307243e3a"),
                ObjectId("643030cb121f565307243e3b"),
                ObjectId("643030cb121f565307243e3c")
        ]
}
{
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "createdCollectionAutomatically" : false,
        "commitQuorum" : "votingMembers",
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1680879819, 10),
                "signature" : {
                        "hash" : BinData(0,"+xm50kdz6gX2Rwfm7MCBydCrqN8="),
                        "keyId" : NumberLong("7219236637505486850")
                }
        },
        "operationTime" : Timestamp(1680879819, 10)
}
bye
I have no name!@mongodb-0:/$ mongo --host mongodb-headless.mongodb.svc.cluster.local -u root -p  neni5xFCLUBJ0Ui0pfor --authenticationDatabase admin < /tmp/catalogue.js 
MongoDB shell version v5.0.8
connecting to: mongodb://mongodb-headless.mongodb.svc.cluster.local:27017/?authSource=admin&compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("94ff4d86-93bb-4b2d-b73d-352c75066660") }
MongoDB server version: 5.0.8
================
Warning: the "mongo" shell has been superseded by "mongosh",
which delivers improved usability and compatibility.The "mongo" shell has been deprecated and will be removed in
an upcoming release.
For installation instructions, see
https://docs.mongodb.com/mongodb-shell/install/
================
catalogue
{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("643030d9f64443a6cfd8cfaa"),
                ObjectId("643030d9f64443a6cfd8cfab"),
                ObjectId("643030d9f64443a6cfd8cfac"),
                ObjectId("643030d9f64443a6cfd8cfad"),
                ObjectId("643030d9f64443a6cfd8cfae"),
                ObjectId("643030d9f64443a6cfd8cfaf"),
                ObjectId("643030d9f64443a6cfd8cfb0"),
                ObjectId("643030d9f64443a6cfd8cfb1"),
                ObjectId("643030d9f64443a6cfd8cfb2"),
                ObjectId("643030d9f64443a6cfd8cfb3"),
                ObjectId("643030d9f64443a6cfd8cfb4")
        ]
}
{
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "createdCollectionAutomatically" : false,
        "commitQuorum" : "votingMembers",
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1680879834, 3),
                "signature" : {
                        "hash" : BinData(0,"PJoSAmt6rubRuHYUnXOdxiRd/ZE="),
                        "keyId" : NumberLong("7219236637505486850")
                }
        },
        "operationTime" : Timestamp(1680879834, 3)
}
{
        "numIndexesBefore" : 2,
        "numIndexesAfter" : 3,
        "createdCollectionAutomatically" : false,
        "commitQuorum" : "votingMembers",
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1680879834, 9),
                "signature" : {
                        "hash" : BinData(0,"PJoSAmt6rubRuHYUnXOdxiRd/ZE="),
                        "keyId" : NumberLong("7219236637505486850")
                }
        },
        "operationTime" : Timestamp(1680879834, 9)
}
bye[g]






db = db.getSiblingDB('catalogue');
db.products.insertMany([
   {sku: 'Watson', name: 'Watson', description: 'Probably the smartest AI on the planet', price: 2001, instock: 2, categories: ['Artificial Intelligence']},
   {sku: 'Ewooid', name: 'Ewooid', description: 'Fully sentient assistant', price: 200, instock: 0, categories: ['Artificial Intelligence']},
   {sku: 'HPTD', name: 'High-Powered Travel Droid', description: 'Traveling to the far reaches of the Galaxy? You need this for protection. Comes in handy when you are lost in space', price: 1200, instock: 12, categories: ['Robot']},
   {sku: 'UHJ', name: 'Ultimate Harvesting Juggernaut', description: 'Extraterrestrial vegetation harvester', price: 5000, instock: 10, categories: ['Robot']},
   {sku: 'EPE', name: 'Extreme Probe Emulator', description: 'Versatile interface adapter for hacking into systems', price: 953, instock: 1, categories: ['Robot']},
   {sku: 'EMM', name: 'Exceptional Medical Machine', description: 'Fully automatic surgery droid with exceptional bedside manner', price: 1024, instock: 1, categories: ['Robot']},
   {sku: 'SHCE', name: 'Strategic Human Control Emulator', description: 'Diplomatic protocol assistant', price: 300, instock: 12, categories: ['Robot']},
   {sku: 'RED', name: 'Responsive Enforcer Droid', description: 'Security detail, will gaurd anything', price: 700, instock: 5, categories: ['Robot']},
   {sku: 'RMC', name: 'Robotic Mining Cyborg', description: 'Excellent tunneling capability to get those rare minerals', price: 42, instock: 48, categories: ['Robot']},
   {sku: 'STAN-1', name: 'Stan', description: 'Observability guru', price: 67, instock: 1000, categories: ['Robot', 'Artificial Intelligence']},
   {sku: 'CNA', name: 'Cybernated Neutralization Android', description: 'Is your spaceship a bit whiffy? This little fellow will bring a breath of fresh air', price: 1000, instock: 0, categories: ['Robot']}
]);


db.products.createIndex({
   name: "text",
   description: "text"
});


db.products.createIndex(
   { sku: 1 },
   { unique: true }
);


db = db.getSiblingDB('users');
db.users.insertMany([
   {name: 'user', password: 'password', email: 'user@me.com'},
   {name: 'stan', password: 'bigbrain', email: 'stan@instana.com'},
   {name: 'partner-57', password: 'worktogether', email: 'howdy@partner.com'}
]);


db.users.createIndex(
   {name: 1},
   {unique: true}
);
	

MySQL


Get the root password from secret manager secret dev/squareops/mysql 


Exec into mysql pod and run : 
curl -o /tmp/10-dump.sq.gz https://github.com/instana/robot-shop/blob/master/mysql/scripts/10-dump.sql.gz?raw=true
curl -o /tmp/20-ratings.sql  https://raw.githubusercontent.com/instana/robot-shop/master/mysql/scripts/20-ratings.sql




mysql -uroot -p63hnfYGhj2ssnAdiVp4q < /tmp/20-ratings.sql
mysql -uroot -p63hnfYGhj2ssnAdiVp4q -e “create database cities”  >> double quotes are interpreted differently on shell when copied from doc . Copy to text editor first 
zcat /tmp/10-dump.sql.gz | mysql -uroot -p63hnfYGhj2ssnAdiVp4q cities 








1. In a terminal (create image)
kubectl run mysql-client --image=mysql:5.7 -it --rm --restart=Never -- /bin/bash
	2. In another Terminal, copy dump. The Dump file is stored here in AWS SKAF account
kubectl cp 10-dump.sql.gz mysql-client:/
	3. exec to mysql and login
  

mysql -u root -p
	   1. Enter password (get password from AWS Secrets)
4. Run bellow commands
CREATE DATABASE ratings
DEFAULT CHARACTER SET 'utf8';
USE ratings;
CREATE TABLE ratings (
   sku varchar(80) NOT NULL,
   avg_rating DECIMAL(3, 2) NOT NULL,
   rating_count INT NOT NULL,
   PRIMARY KEY (sku)
) ENGINE=InnoDB;

create database cities;
	


5. Now go to (create image) terminal of point 1 and run
zcat 10-dump.sql.gz | mysql -h  mysqldb-primary.mysqldb.svc.cluster.local -u root -p cities
	Jenkins webhook Configuration in Gitlab
Open gitlab and navigate to Reference Architecture-> msa-ref-roboshop(https://gitlab.com/sq-ia/ref/msa-app)
1. Add jenkins webhook  in each microservice application
   1. Go to application> settings> integration > jenkins
      1. Add Jenkins server url: https://jenkins.demo.skaf.squareops.in
      2. Add Project name (same for pipeline): web
      3. Add Jenkins Username: admin
      4. Add Jenkins Password: Get Jenkins Password from K8s module outputs on Terraform Cloud
  





ArgoCD
Login to Argocd (get credentials and URL from K8s Output)
1. Create Repository
1. Browse to Settings on the left side bar, click on Repositories, Click CONNECT REPO USING HTTPS
i. Add the following configuration:
URL: https://gitlab.com/sq-ia/ref/msa-app/helm.git
Username: gitlab+deploy-token-1401129
Password: NySyabsEnjANE5ynp7-y
  

Now, click on Connect and ensure connection status is successful.


2. Create application
   1. Go to Applications from left sidebar and Click on New App
      1. Give Application Name: roboshop
      2. Project Name: default
      3. SYNC POLICY: Automatic
         1. Select checkbox
            1. PRUNE RESOURCES
            2. SELF HEAL
            3. AUTO-CREATE NAMESPACE
  

  
  
  



      4. In Source
         1. Give Repository URL from dropdown
         2. Revision: qa
         3. Helm Values Path: .
      5. In Destination
         1. Select cluster URL from dropdown list
         2. Namespace: roboshop
      6. In Helm, 
1. Values Files: env/demo/values.yaml (Press enter after entering this values for ArgoCD to confirm selection)




Update AWS secrets Manager with database credentials with below corresponding values: 
demo/user=> mongodb
demo/catalogue => mongodb
demo/payment  => rabbit
demo/dispatch  => rabbit
demo/shipping =>  mysql 
demo/ratings => mysql
demo/web and demo/cart require no secret


Access Robot-shop Application: https://roboshop.demo.skaf.squareops.in/


Check CICD implementation
   1. Push/Make changes web application git repository that will trigger the pipeline automatically and send a notification on slack for each success and failure along with this, it will trigger Argocd to update its application with latest image.




Cleanup
  

SKAF Demo Checklist
   * Base Setup
   * Network Setup
   * EKS setup
   * K8s Setup
   * Jenkins
   * Add credentials
   * Gitlab
   * AWS        
   * Sonarqube
   * Slack
   * Check integration
   * Sonarqube
   * Slack
   * Webhook with Gitlab
   * Argocd
   * Check Deployment is running properly
   * Kibana
   * Perform Manual Steps mentioned in below doc
   * Grafana
   * Alerts
   * Check all dashboards
   * Check slack integration
   * Rancher
   * Sonarqube
   * Create token
   * Create webhook for Jenkins
   * Data Module
   * MongoDB
   * Insert Dummy Data
   * MySQL
   * Insert Dummy Data
   * Redis
   * RabbitMQ
   * Check Jenkins Webhook integration in GitLab Roboshop Application
   * Check CICD by manually changing in any microservices


[a]Module app is dependent on module k8s. Domain mapping of Route53 records to Load balancer process takes place in k8s module.
[b]I don't think this is needed
[c]Pending to review . Not sure what exactly to be shown from Demo purpose. Even ELK may not be ready for demo .
[d]Hold this step for later, once management services are configured
[e]how does this make sure that we are logging into primary always ? does this headless service routes to primary instance only ?
[f]This endpoint resolves to pods mongo master and slave pods. However, when connecting to this endpoint, connection is passed over to master
[g]Check with rohit if db loaded successfully
