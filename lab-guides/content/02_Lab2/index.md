## Lab 2: AWS Cloud Visibility 
### Introduction 
For intelligent **monitoring** of **services** and **infrastructure** running in **Amazon Cloud**, you can integrate Dynatrace with **Amazon Web Services** (**AWS**). The **AWS Cloudwatch integration**, combined with the **OneAgent**, helps you **stay on top of the dynamics** of your **data center** in the **cloud**. 
In this lab we will complete the following objectives:
* *Learn* how to **configure** **Dynatrace** to **integrate with** **AWS** **CloudWatch**
* *Learn* how **AWS CloudWatch** **metrics** can be configured as **metric events** for **anomaly detection** and **automation**
* *Review* how **Dynatrace** can perform **self-healing** tasks with a connection to **AWS** in **workflows**

### Lab Setup

For this lab you are going to set up an **EC2 instance** that runs a **sample application** that you can use to learn the **Dynatrace** **platform** and to review how Dynatrace brings **tremendous insights** all through the **Dynatrace OneAgent**.

#### Dynatrace API Token

1.	*Open* your **Dynatrace tenant** and *navigate* to **dashboards**.
2.	*Open* the **Dashboard** titled “**APItokenDashboard**”. This *contains* your **API token**, with all required **permissions**. *Copy* this **value** and *save* it for later.

#### AWS Account Creation
1. Click on the AWS training and certification workshop link to join the event 

2. Click on Join the event 

3. Input your email to have a code generated for you to login with a 1-time passcode. Copy that passcode into the box below and Sign In

![picture 0](../../assets/images/7e83cea7736031e647333e5b19e005b8775ff179af09ce883c3b55040e1b92f8.png)  

4.	Review the Terms and Conditions, Agree, then Join Event

![picture 1](../../assets/images/f7e19d15d48260f178ae0e04e688e71ef4f5825ef5d56a6869fe7efe326d684f.png)  

5.	In the lower left-hand corner Click on “Open AWS console (us-west-2)

![picture 2](../../assets/images/70d04877ff04cde33dc0bfa5bc3fbb2adc4c3017a01bd9afe50024fd6aed7870.png)  

#### AWS CLI Lab Setup

1.	*Open* up **CloudShell** in a **new tab**. To *open* the **CloudShell**, *right click* on the **CloudShell** icon at the top of the **AWS console**, and *open* in a **new tab**. This may take a minute to complete. 

In this lab, we will be using **AWS CloudShell**. **CloudShell** is a **browser-based shell** that makes it **easy to securely** **manage**, **explore**, and **interact** with your **AWS resources**.

To *open* the **CloudShell**, *click* on the **CloudShell** icon at the top of the **AWS console**. This may take a *minute* to *complete*.

![picture 0](../../assets/images/60ab8a246332230a35f52535bf621f7d10f7a6e02134476a4872fe95a7b16e05.png)  

*This may open up a slash page.*

![picture 1](../../assets/images/044bdf0820944ef9dce64832f5d3ea90a547c4ab155bf92970357b64bf57c91e.png)  

2.	After *closing* the **pop-up**, *wait* a minute for the **CloudShell** **to initialize.** When this is done, you will see the **command prompt** as shown below.

![picture 2](../../assets/images/2ba73b494c29b4692669bfce380db19bd87f9d4aa7b614b3a6ddd55cac9f1eb8.png)  

3.	*Clone* the **workshop scripts**.

Once you have the **CloudShell** open, you need to get some **scripts** that will **automate the** **workshop** setup. *Run* this **command**:
```
git clone https://github.com/Epeiswerth/aws-modernization-dt-orders-setup-saas.git
```
It should look like this:
 
```
[CloudShell-user@ip-10-0-52-50 ~]$ git clone https://github.com/Epeiswerth/aws-modernization-dt-orders-setup-saas.git
 
Cloning into ‘aws-modernization-dt-orders-setup’…
 
remote: Enumerating objects: 161, done.
Remote: Counting objects: 100% (161/161), done.
Remote: Compressing objects: 100% (96/96), done.
Remote: Total 161 (delta 72), reused 143 (delta 60), pack-reused 0
Receiving objects: 100% (161/161), 19.82 MiB | 22.21 MiB/s, done.
Resolving deltas: 100% (72/72), done.
```
#### Provision VM
This step creates **two** **CloudFormation** **stacks** that do the following:
- *Add* **two** **EC2 instance** named: **dt-orders-monolith** and **dt-orders-services**
- At EC2 startup, it *installs* **Docker** and **Docker-Compose**
- At EC2 startup, it *installs* the **OneAgent** for your **Dynatrace** tenant
- Starts up the sample **application** by *running* **docker-compose up**

1.	*Type*:
```
 pwd
 ```
 You *should* be in the **directory**:
 ```
 /home/CloudShell-user
 ```
2.	*Copy* and *run* provisioning **script** **command** *exactly* as described below from the directory **/home/CloudShell-user**
```
cd ~/aws-modernization-dt-orders-setup-saas/provision-scripts && ./provision-workshop.sh https://<<tenantID>>.live.dynatrace.com <<API_Token>> <<Your email used to login to Dynatrace>>
```
 
Within the **AWS SSH shell**, *paste* the full **command** and you should see a **prompt to proceed** as shown below.

```
About to Provision Workshop for:
https://<tenantID>.live.dynatrace.com
SETUP_TYPE   = all
KEYPAIR_NAME = ws-default-keypair
Proceed? (y/n) : 
```
 
Once the **script is complete**, you should *see* **output** as shown below.
 
```
Done Setting up Workshop config
End: Thu Nov  4 01:45:06 UTC 2021
Create AWS resource: monolith-vm
{
    “StackId”: “arn:aws:cloudformation:us-west-2:838488672964:stack/monolith-vm-1635990306/d82cd2b0-3d10-11ec-a495-023df82ab493”
}
```
 
3.	*Verify* **CloudFormation** **Stacks**

The **CloudFormation** may take a few minutes, but you can *check* the **CloudFormation** **output** to ensure that all the **AWS** **resources** were provisioned successfully.

*Monitor* **CloudFormation** stack status *within* the **AWS** **console**. *Navigate* to the **CloudFormation** page or just navigate to:
* [https://console.aws.amazon.com/cloudformation/home](https://console.aws.amazon.com/cloudformation/home)

When it is *complete*, it will show a **CREATE_COMPLETE** status as shown below.

![picture 3](../../assets/images/ac90c928218eefb1f026f237bf97ddb536378c5a6e77ae39ce4162939b01b0b7.png)  

#### Create an AWS OIDC Identity Provider for AWS Connector
The **AWS Connector** actions use **OpenID Connect** (**OIDC**) to authenticate with AWS, allowing them to access **AWS resources**. To *configure* **AWS IAM**, we will do the following
1.	On the left side menu, *Add* a new **Identity Provider**.
2.	*Add* an **AWS** role.
      -	*Open* **IAM** in the **AWS** Console > *Click* on **Identity Providers** > **Create Identity Provider**
      -	*Choose* “**OpenID Connect**”
      -	*Enter* in the following details
 
Provider URL: 
```
https://token.dynatrace.com
```
 
Audience: 
 
```
<<TenantID>>.apps.dynatrace.com/app-id/dynatrace.aws.connector
```
 
It should look like this:

![picture 4](../../assets/images/6462762fe9c396e04fe4efb81b891bf866d29480fceb476f8e6676df20d0d5bb.png)  

3.	*Click* “**Add Provider**”
4.	*Click* on the “**Assign role**” button in the upper right hand corner, and *Create* a **new role**

![picture 5](../../assets/images/9b33199f8d51525b666abc5527ea9df1c67edbda1d95a39e994cac906b28c679.png)  

5.	*Select* **Trusted entity**
7.	*Choose* “**Web Identity**”
8.	**Identity Provider** should be *set to* “**token.dynatrace.com**”
9. **Audience** should be  

``` 
<<TenantID>>.apps.dynatrace.com/app-id/dynatrace.aws.connector 
``` 
10.	*Click* **Next**
11.	*Choose* these **policies**

```AmazonEC2FullAcess```

```eks-autoscaler-policy```

![picture 6](../../assets/images/c0cdbc52a09327ad705388d0b5c286d4a59ce41f4bb81c6a3d74f4d8ae35c2ea.png)  

12.	*Click* **Next**
13.	*Give* the **Role** a **Name**:
```
Dynatrace-AWSConnectAdmin
```
14.	It should look *similar* to this 

![picture 7](../../assets/images/ac666e7378248d6394626eef5f4fcea8b2f6bb878c9c400a119e9c6dda5aab5d.png)  

15.	*Create* the **role**, then *view* the **role**
16.	*Copy* the **ARN** of the **role**, and *save* **it** in a **notepad**

![picture 8](../../assets/images/7f102074b771c01fb2504c1f4fab61129282277d15a93d9e217aa3f9f896d534.png)  

Now we need to allow your **tenant** to connect to **endpoints** in the **allow-list**. In the **Dynatrace UI**:
1.	*Open* **Settings App (New)**
2.	*Select* **Connections**
3.	*Select* the **Connector** **AWS**
4.	*Add* a new **Connection**
    * Name: 
```
PerformAWSConnect
```
**Role ARN** – *Paste* in the **ARN** you just *saved* from the **AWS console**

5.	*Click* **Create**

![picture 9](../../assets/images/55c2436de8165188a315d5c1d8baa47f3768a4df8e4a57f0d9f20b69ed843ea4.png)  

We need to *allow* your **tenant** to *connect* to **endpoints** in the **allow-list**.
 
6.	In the **settings page**, *select* “**Limit outbound connections**”
 
7.	*Add* to the **allow-list**
```
*.amazonaws.com
```
8.	*Add* another **allow-list** endpoint
```
<<TenantId>>.live.dynatrace.com 
``` 
 
9.	*Save* **Changes**

![picture 10](../../assets/images/9ce2a6113924fb7bd73882d48616c7cf9e654f9f52011e241cd90ca3bd960ff3.png)  

### Components for Lab 2
Referring to the picture below, here are the **components** for lab 2.

![picture 11](../../assets/images/f7e41b9310564ff2fff0f27fc44111df541e9ca9c6624418161dec4d347f4077.png)  

#### 1. Sample Application

A sample app representing a simple architecture of a **frontend** and **backend** implemented as **Docker containers** that we will review in this lab.

#### 2. Dynatrace monitoring

Alongside an **AWS cloudwatch integration** (which we will revisit in a next step), the **Dynatrace OneAgent** has been installed by the **workshop** **provisioning** **scripts** onto the **EC2 instance** and is communicating to your Dynatrace Tenant.

##### TECHNICAL NOTE

Learn more about the various ways the **OneAgent** can be installed, in the [Dynatrace documentation](https://docs.dynatrace.com/docs/ingest-from)

#### 3. Load generator process

A JMeter process sends **simulated user traffic** to the **sample app** running within a Docker container. You will not need to interact with this container, it just runs in the background.

##### TECHNICAL NOTE:

A real-world scenario would often start with the application components running on a **physical** or **virtualized host** **on-prem** and not “**Dockerized**”. To simplify the workshop, we **“Dockerized”** the application into a **front-end** and **back-end**. In **Dynatrace**, these Docker containers all show up as “**processes**” on a **host** just like a “**non-Dockerized**” **application** will.

#### Sample app

The sample application is called **Dynatrace Orders**. A more **detailed overview** & **source code** can be found on [GitHub](https://github.com/dt-orders/overview). 

#### Get the Public IP to the frontend of the Sample Application

To get the **Public IP**, *open* the **EC2 instances** page in the **AWS console**. On the newly created **host** **dt-orders-monolith** *find* the **Public IP** as shown below.

![picture 12](../../assets/images/9ef19c308bece21b0b6c0dfe116b5a72bf3b194b627e99f3aa8ba584409d0d61.png)  

#### View the Sample app in a Browser

To view the application, *paste* the **public IP** using **HTTP**, NOT HTTPS, into a **browser** that will look like this:

![picture 13](../../assets/images/5f1105efffa42eef00af1c80c083ebbeaa2bcba8e7e509ec41e64659acdebca9.png)  

*Use* the **menu** on the **home page** to *navigate* around the **application**. *Note* the **URL** for key functionality. You will see these **URLs** later as we *analyze* the **application**.

* **Customer List** = customer/list.html
* **Customer Detail** - Each customer has a unique page = customer/5.html
* **Catalog List** = catalog/list.html
* **Catalog Search Form** = catalog/searchForm.html
* **Order List** = order/list.html
* **Order Form** = order/form.html

### OneAgent Data 

*Review* the **Infrastructure and Operations App** to *view* the **Monolith VM**.

![picture 14](../../assets/images/a18b74c0ed249e76ef6e8d84492a12a27d87731ee6cac2226e4528c791efc80b.png)  

![picture 15](../../assets/images/b70b64d38ac8c1954432561682dfdafcb5bf025fd9ca3f6df4d22863602f8ed0.png)  

![picture 16](../../assets/images/b2e33953dce1cf6fc34c576f031a8076ae10cd1f352208971a73245a0cbd4b44.png)  

![picture 17](../../assets/images/4d8309ce41bf5ea333e96cce8a08a8fe3c8900c184f9f0f969a23e4a40009c31.png)  

### Clouds App

In addition to monitoring your **AWS workloads** using **OneAgent**, Dynatrace provides integration with **AWS CloudWatch** which adds **infrastructure monitoring** to gain insight even into all popular **AWS services** (e.g. serverless application scenarios).

#### How this helps
The **AWS** **Cloudwatch** **integration** brings **logs** and additional **metrics** for **cloud infrastructure**, **load balancers**, **API Management Services**, and **more** into the Dynatrace platform. Dynatrace brings value by **enriching the data in context** with **all observability signal**s. 

The **Cloudwatch metrics** are managed by **Dynatrace’s AI engine** **automatically** and this extended observability **improves operations**, **reduces MTTR** and **increases innovation**.

Here is an example, **filtered** for the **AWS account** of this lab:

![picture 18](../../assets/images/c6a73a459db444c7095b038d124017e9fed7103d4b9606c4115e3470174f8281.png)  

![picture 20](../../assets/images/149909954e0f5f1f089d696fab0a65b6ae641c78a961358ebc7000fb5c4270e2.png) 

The **Clouds** **App** gives you a **unified perspective** among your **multi-cloud inventories** and allows you to drilldown to your relevant set of **cloud services**. For example you can filter by:
* Region
* Service Category (e.g. Databases)
* Service Type (e.g. AWS Lambda)
* Service Name
* Account ID (Environment)
* Tags

<br>
<details>
  <summary>OPTIONAL - Hands-on Exercise – AWS Cloudwatch metric integration</summary>

The steps below were already completed with the Cloudformation template. We will not perform the steps to set up an (additional) integration with AWS cloudwatch to ingest metrics to Dynatrace, but they are here for your reference. 

1.	From the **Clouds App** > **Configure** > **AWS** > **Connect new Instance**
2.	*Add* the **following information**:
- **Connection Name**: 
```
Dynatrace Integration
```

- **Auth method**: Role Based authentication
- **IAM Role**: 
```
Dynatrace_monitoring_role
```
- Your **AWS** **account ID**: found on the **AWS console** **top right-hand dropdown**.
- **Monitor all resources**

*Click* **Save**

![picture 19](../../assets/images/123595159bcc266f22a4229f80f0931cc7700e0c34303c89a079a80c923af5c4.png)  

3.	Once the **connection** is successfully *verified* and *saved*, your **AWS account** will be listed in the **Clouds** app.
</details>


#### Review Collected Metrics

Let’s *navigate* to the **AWS dashboards** that were automatically created as soon as you hooked up the integration with **AWS Cloudwatch**:

![picture 21](../../assets/images/d6ab236797b7d815825456b92f956d620882ea4844d90abc71ef5afca177663f.png)  

#### Why is this important? 

The **AWS integration** is a central way to get a **picture** and **metrics** for the **AWS resources** running against your **accounts** as you migrate.

Read more about the **latest AWS integrations** in our [blog](https://www.dynatrace.com/news/blog/new-integrations-announced-at-aws-reinvent-enhance-cloud-performance-security-and-automation/) and review all details about our AWS integrations in the [Dynatrace documentation](https://docs.dynatrace.com/docs/observe/infrastructure-monitoring/cloud-platform-monitoring/aws-monitoring). 

#### Custom Metric Events

Dynatrace Davis **automatically analyzes** abnormal situations within your **IT infrastructure** and attempts to **identify any relevant** **impact** and **root causes**. 

Davis relies on a wide spectrum of information sources, such as a **transactional** **view** of your **services** and **applications**, as well as **all events raised** on individual nodes within your **Smartscape topology**.

 There are two main **sources** for single events in Dynatrace:
1.	**Metric-based events** (events that are triggered by a series of measurements)
2.	**Events** that are independent of any **metric** (for example, **process crashes**, **deployment changes**, and **VM motion events**)

**Custom metric events** are configured in the **global settings** of your **environment**, and are *visible* to all **Dynatrace** users in your **environment**.

### Hands-on Custom Anomaly Detection

**Davis Anomaly Detection** allows you to create **anomaly detectors**, set up **customized alerts**, and **transform** **metric events** configuration. You can also save time and create an **anomaly detector** in **Notebooks** while using the **app**. We will be using the **metrics** *captured* from **AWS**.

1.	*Go* to the app **Davis Anomaly Detection**.
2.	*Select* the **“Settings”** dropdown in the upper right-hand corner, *select* **Authorization Settings**, and *select* **All permissions**.
3.	*Select* **Anomaly Detector** > **Create your own Anomaly Detector**.
4.	*Give* your configuration a meaningful **Title**, like: 

```
High EC2 CPU Usage %
```  

5.	*Expand* Configure your query and *provide* the **DQL query** to *fetch* your **data**.
6.	**DQL Query**: 
```
timeseries max(dt.host.cpu.usage), by: {dt.entity.ec2_instance, dt.source_entity}
| lookup [fetch `dt.entity.EC2_INSTANCE`
  | fields id, entity.name, tags, awsInstanceId], sourceField:dt.entity.ec2_instance, lookupField:id
| fieldsAdd EC2Tags = lookup.tags, awsInstanceId = lookup.awsInstanceId
| expand EC2Tags
| filter EC2Tags == "[AWS]Name:dt-orders-monolith"
```
7.	**Actor**: Yourself
##### Customize Parameters
1. **Analyzers:** Static threshold anomaly detection
2. **Threshold**: 70
3.	*Click* on “**Show Advanced Properties**”. *Change* the **values** to:
      - **Violating Samples**: 1  
      - **Sliding Window**: 3 
      - **Dealerting samples**: 3

![picture 22](../../assets/images/db270a83c38436b483edf9026828fa94d243787bd33ccfa6e6dfeb5a2af7feee.png)  

4.	**Alert condition**: Alert if Metric is Above
5.  *Create* an **Event Template**
- **Event Name**: 
```
High EC2 CPU Usage %
```
- Event Description: 
```
The EC2 CPU Usage on {dims:dt.source_entity} exceeded {threshold} with AWS tags {dims:EC2Tags}
```
-	*Configure* your **Event Properties** to look like this

![picture 23](../../assets/images/36767a1322ce80cab613a793b19f0d74692bebb6efd5253c05c9b7437c71e4ed.png)  

   - Add the key value pairs 
```
AWS_Tags : {dims:EC2Tags}
AWS_Instance_ID : {dims:lookup.awsInstanceId}
```
9.	*Click* **Create**.

### AWS Connect for Workflows

*Open* the **AWS CLI**.

#### Trigger Workflow with Problem Card

We are going to create a **self-healing** **workflow** that gets triggered from the **anomaly detection** **event** you just created for the **EC2** **CPU usage**. This workflow will take advantage of the **AWS Connector integration**. **AWS Connector** enables your Dynatrace environment to interact with different **AWS Services** based on **events** and **schedules** defined in a dedicated **workflow**.

In a **workflow**, you can **query** and *manipulate* **AWS** resources, such as **EC2 instances** or **S3 buckets**. **AWS Connector** provides an extensive set of **workflows** actions, which *offer* a familiar interface very closely aligned with **AWS CLI** so that you can use *actions* and *concepts* that are already well known to you.

#### Create EC2 Reboot workflow

1.	To authorize the service account running this workflow, in the **upper right hand corner**, *click* on “**Settings**” then *select* “**Authorization settings**”

![picture 24](../../assets/images/5a15d7d47ca2a6886e28364649885a7af29f6b3e032f83356e91c8028eb67966.png)  

2.	*Grant* **all** **Permissions** and *Close* the **popup window**. The below message indicates that there are **some** **permissions** that could not be granted to the **workflow service account** because the group you are a part of does not have them granted in the **policy**. They are not needed for this lab.

![picture 25](../../assets/images/f70ef9caaac48cd85e0bc3adae57af6a0822dac40407ad7186b0a5daea687876.png)  

3.	*Click* on the **Workflows App** > *click* on “**+Workflow**”
4.	*Select* Trigger: **On demand trigger**
5.	*Click* on add **task** right below trigger box
6.	*Search* for **actions**, and *select*: 
```
EC2: Describe instances
```
7.	In the **Connection** space, *find* the **Connection** you just *built* “**PerformAWSConnect**”
8.	In the **region** section, *click* on the code box “**Add Expression**” on the right hand side of the region field. *Delete* all the curly brackets and type: 
```
us-west-2
```
9.	In **Filters** *enter* the following
```
tag:Name
```
 
```
dt-orders-monolith
```
10.	*Click* **Save**, then **Run**
11.	Once it runs successfully, *click* on the **action** in the **workflow** and it should show the result as shown below. You can *expand* the result to see the list of **VM instances** and their **IDs**. We will *parse* this result to pull out the **instance ID** for the next step.

![picture 26](../../assets/images/5e1cb0ec181088fde94fead1b8b5e6d65d7ef6d320d3aa1754b5d97e1f436e61.png)  

12.	*Add* another task, “**EC2 Reboot Instances**”
13.	In the **region section**, *click* on the code box “**Add Expression**” on the right hand side of the region field. *Delete* all the curly brackets and type: 
```
us-west-2
 ```
14.	*Add* this **parameter** in the “**instanceIds**” field, and *Save* the **Workflow**
``` 
{{result("ec2_describe_instances_1").Reservations[0].Instances[0].InstanceId}}
```

![picture 27](../../assets/images/b39a63ba1f98dc8bdc150a93b1e69bfb697a23b58830c19f8bb6eea5a6e94c1a.png) 

15. *Save* your **workflow** 
15.	*Change* the **trigger** type to “**Davis problem trigger**”
16.	*Describe* the **trigger** as seen below to recognize the **custom event** type from your **anomaly detection rule**.

![picture 28](../../assets/images/db3ddc56740ab0ab30a5b1849e8052883c7d35ca9eb8245c0e2f801a35b7be90.png)  

17.	*Filter* on **tags** with the **key:value pair**:
 ```
 [AWS]Name
 ```
 ```
 dt-orders-monolith
 ```
18.	*Save* the **Workflow**
19.	*Change* the name of the **workflow** to “**Reboot Monolith VM**” by *clicking* on the **title**. Then *Save* the **Workflow** again.

#### Trigger Custom Problem Alerts
##### SSH to monolith host
 
We are going to manually trigger a **CPU problem** on the **monolith VM**. This will **generate a problem** from your **custom anomaly detection rule** and will trigger the **EC2 reboot workflow**. 

To connect to the host, simply use **EC2 Instance Connect**. To do this, *navigate* to the **EC2 instances** page in the **AWS console**.

From the list, *pick* the **dt-orders-monolith** (1) and then the **Connect button** (2) as shown below.

![picture 29](../../assets/images/0eeb0389b9ae9ce9d1d69c8ada750992629bd494a0f236268da28af4cb1bc6a6.png)  

Then on the next page, *choose* the **EC2 Instance Connect** option and then the **connect button**.

![picture 30](../../assets/images/ff824d32093238757b6dd9ec9b5997f2d6a6c77fc071e4105ca1774c2174d9cf.png)  

Once you're connected, you will see the terminal prompt like below.
```
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-1045-aws x86_64)
...
...
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
ubuntu@ip-10-0-0-118:~$ 
```
 
1. Using a **unix utility yes**, we can generate **CPU stress** just by *running* the **yes command** a few times.
In the terminal, *copy* **all these lines** and *run* **them**:
```
yes > /dev/null &
yes > /dev/null &
yes > /dev/null &
```
To verify, *run* this **command**:
```
ps -ef | grep yes
```
The output should look like this:
```
ubuntu    5802  5438 99 20:48 pts/0    00:00:05 yes
ubuntu    5805  5438 89 20:48 pts/0    00:00:04 yes
ubuntu    5806  5438 97 20:48 pts/0    00:00:03 yes
ubuntu    5818  5438  0 20:48 pts/0    00:00:00 grep --color=auto yes
```
2.	Back in Dynatrace within the **Infrastructure and Operations view**, the CPU should now be high, as shown below:

![picture 31](../../assets/images/889db60334e2f0cbac868cd6e79a0589d5fd82b57624551499848efb97ebfd6e.png)  

3.	It may take a minute or so, but you will get a **problem card** shown below. #1 is the alert from the **severity = RESOURCE** where Davis AI was invoked, and #2 is the alert from **severity = CUSTOM ALERT**.

![picture 32](../../assets/images/948c733b39c7f6a7c2108a2068a751544d9a54b1d836b161090d2d2c950faa7c.png)  

4.	*Navigate* back to the **Workflows App** and find the **workflow you just created**
5.	*Click* on the **ellipsis button** to view the execution history

![picture 33](../../assets/images/cdbfadd48347e2fcd83f5ac2c57fd6a21a46e7f0d221039339e8342f8b49bc87.png)  

6.	It should show a successful run that was triggered by the Davis AI detected **custom alert problem**:

![picture 34](../../assets/images/c1efee416d2c15bf5db3baa2f04bdbb6135bc4568c5564bd53c7bb63f61068fc.png)  

##### *(Optional – If the workflow does not correctly reboot your VM)*
To stop the problem, you need to *kill* the **processes**. To do this:
1.	Back in **CloudShell**, *run* this **command** to get the **process IDs** 

``` 
ps -ef | grep yes 
```

2.	For each **process**, *copy* the **process ID** and *run* **kill** <PID\>

For example:
##### If the output is this...
 ```
ubuntu@ip-10-0-0-118:~$ ps -ef | grep yes
ubuntu    5802  5438 99 20:48 pts/0    00:00:05 yes
ubuntu    5805  5438 89 20:48 pts/0    00:00:04 yes
ubuntu    5806  5438 97 20:48 pts/0    00:00:03 yes
 ```
##### Then run...
 ```
kill 5802
kill 5805
kill 5806
```

Or better to use the below **command** to *kill* all the **PID’s** at once

```
kill $(ps -ef | grep yes | awk '{print $2}' | sed '$d')
```
Or even more effective is:

```
pkill yes
```

3.	*Verify* they are gone by *running* this again: 

```
ps -ef | grep yes
```

4.	*Verify* that **CPU** in Dynatrace goes to normal and the **problems** will eventually *automatically* close.
5.	*Exit* the **SSH**: Simply type **“exit”** to return to the **CloudShell**.

### Lab 2 Conclusion
In this section, you have completed the following:
- Review how Dynatrace integrates with **AWS CloudWatch**
- Review how **Infrastructure metrics** can be configured as **Metric events** for alerts
- Review how Dynatrace can perform **self-healing** tasks with a connection to **AWS** in **workflows**
