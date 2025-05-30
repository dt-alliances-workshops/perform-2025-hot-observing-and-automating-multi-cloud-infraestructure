## Lab 3 – Kubernetes Observability and EKS Autoscaling Workflow 

### Introduction 

In this lab, we will learn how to observe and **troubleshoot Kubernetes clusters** with Dynatrace. We will use the **out-of-the-box health assessment** provided by Dynatrace to detect and analyze a reliability issue. Additionally, we will build a workflow to **automate a self-healing action** for this use case.  

For the hands-on part of this lab, we will showcase the integration with **AWS EKS** via Dynatrace Operator (on top of the Cloudwatch integration). We will create a workflow and you will see the workflow execution **triggered by Dynatrace Davis AI**. 

#### Lab Prerequisites

The following assets / scripts have already been triggered in Lab 2: 
- Create CloudFormation Stacks by executing provisionWorkshop.sh script 
- Create an AWS OIDC Identity Provider for AWS Connector  
- Create AWS connection and allow outbound connection to AWS APIs (“AWS Connect for Workflows”) 
- Authorization Settings in Workflow App for Service User are granted 
 
### Lab Setup 

For this lab you are going to use an **updated version of the order application** in Kubernetes. This makes it easy for you to see the **transformation of the Sample Application** in Lab 2 and the value of running Kubernetes on AWS without needing to stand up or maintain your own Kubernetes control plane. 

#### Verify CloudFormation Stacks

We have automated the EKS setup for this lab by leveraging CloudFormation stacks. As this has been already included in the initial workshop provisioning script triggered in Lab 2, we should already have an EKS cluster with 2 nodes as well as an integration with Dynatrace up and running. Please check the **CloudFormation output** to ensure that all the **AWS resources were provisioned successfully**. 

*Navigate* to the **CloudFormation page** to check the status in the AWS console: 
- [https://console.aws.amazon.com/cloudformation/home](https://console.aws.amazon.com/cloudformation/home) 

When it is complete, it will show a **CREATE_COMPLETE** status as shown below. 

![picture 0](../../assets/images/0ccf50643253ca9cd2730bed737464cf81e6c058be29e2fad2f96d2a75a96141.png)  

#### Verify Cluster within AWS Console

With the AWS Console, *search* for the “**Elastic Kubernetes Service**” or *click* on the **link below**.
- [https://console.aws.amazon.com/eks/home#/clusters](https://console.aws.amazon.com/eks/home#/clusters)

On the **cluster page**, *click* on cluster “**dynatrace-workshop**”. 

Next *click* on the **compute** tab. You will see **2 Kubernetes nodes** managed by one node group. 

*Click* on the **NodeGroup** and *review* the **details**. The node group is managed by an **EC2 Autoscaling group** with a lower (Minimum size) and upper (Maximum size) boundary.

![picture 1](../../assets/images/724ad746591462dda3ab44a7ff6cb7f52c77954ae504738f544059e2b2f0f31e.png)  

### Deploy App on EKS

For this lab, another version of the application, used in Lab 2, exists that breaks out each of these **backend services** into **separate services**. By putting these services into container images, we gain the ability to deploy the service into **containerized workloads in EKS**.

![picture 2](../../assets/images/fec982a5e64af73e5592aeaf0e3bab811ca1e0e70d6a8da497062f13b963b1dc.png)  

#### Deploy sample app on EKS

*Copy* & *paste* the **following command** in your **AWS cloud shell** to *deploy* the **sample app**

```
 cd ~/aws-modernization-dt-orders-setup-saas/app-scripts
./start-k8.sh
```

The output should look similar to this:

![picture 3](../../assets/images/38b65b0d8c7c6b4b7d52c07c7dd8b32d57cd548ed110ad2793c2f419bf22f6ad.png)  

Note the status **“Pending”** of the frontend pod. We will revisit that in a later step. 

### Dynatrace on EKS - Dynatrace Operator

One key Dynatrace advantage is **ease of activation**. OneAgent technology **simplifies deployment** across large enterprises and **relieves engineers** of the burden of instrumenting their applications by hand. As Kubernetes adoption continues to grow, it becomes more important than ever to **simplify the activation of observability** across workloads without sacrificing the deployment automation that Kubernetes provides. 

The Dynatrace Operator **automates the deployment** of required Dynatrace components within Kubernetes clusters. Observability should be **as cloud-native as Kubernetes** itself. In our workshop, we have already pre-installed the Dynatrace Operator via helm in the initial provisioning script.
 
*Verify* Dynatrace Deployment in EKS using **kubectl**
1.	*Open* **CloudShell**
2.	(Optional – as already installed: *install* **kubectl**)
3.	*Run* the **following commands**: 
```
 kubectl get dynakube -n dynatrace
```
```
 kubectl get pod -n dynatrace
```
 
*Verify* that the DynaKube (Dynatrace Custom Resource) and all pods in the Dynatrace namespace are **ready & running**. 

```
[cloudshell-user@ip-10-142-77-211 app-scripts]$ kubectl get dynakube -n dynatrace
NAME               APIURL                                    STATUS    AGE
workshop-cluster   https://tqp85835.live.dynatrace.com/api   Running   55m
[cloudshell-user@ip-10-142-77-211 app-scripts]$ kubectl get po -n dynatrace
NAME                                  READY   STATUS    RESTARTS   AGE
dynatrace-oneagent-csi-driver-9ttw2   4/4     Running   0          56m
dynatrace-oneagent-csi-driver-bcqcl   4/4     Running   0          56m
dynatrace-operator-58bd4995bf-r2t7b   1/1     Running   0          56m
dynatrace-webhook-d6f748f58-5wg2k     1/1     Running   0          56m
workshop-cluster-activegate-0         1/1     Running   0          56m
workshop-cluster-oneagent-7q2md       1/1     Running   0          56m
workshop-cluster-oneagent-gj6gg       1/1     Running   0          56m
```


<details>
  <summary>Optional - Deploy Dynatrace in a Kubernetes cluster</summary>

*Open* the **Kubernetes App** in your Dynatrace tenant and *click* on “**Add cluster**” in the **upper right corner**. 

The **Kubernetes App** guides you through **onboarding a new K8s cluster to Dynatrace via helm**. More information about the different deployment options can be found in our [documentation](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s).

![picture 7](../../assets/images/533cbcaa0fa8e83754ce3daa2235958eb8f0baa28bc5c889f07ca6b6c777bf2f.png)
</details>

### Kubernetes Observability with Dynatrace

#### Dynatrace Kubernetes App

*Open* the **Kubernetes app** in the **Dynatrace platform**. 

You will immediately get a first overview of the **health state** of the most important Kubernetes objects within your K8s clusters. Notice the links to **ready-made** dashboards for various Kubernetes object types (e.g. Node - Pods).

![picture 5](../../assets/images/c93c620e2a58be5bc14c48661b31b3537cb7d5e6dcbceaee068317f49588a3cb.png)  

Note the **info message highlighting** that there are some out-of-the-box alerts are inactive. You can investigate the list of health alerts that are available out of the box by either *clicking* on **“show”** in this info message or by *switching* to the **“Recommendations”** tab in the K8s App.

**Out-of-the-box health alerts**

![picture 6](../../assets/images/8abd1eef2341fdd9f6c69ad021f79a0fa55a8ddc57b2d2037f6cc8de7f12c956.png)  

*Expand* the workload alerts section. You can see that most of the **workload alerts** are activated out of the box. *Click on “Activate”* to view or change the settings for workload alerts. You will get redirected to the Classic Settings app. 

To speed things up for our Hands-on-session, we’ve adapted the default configuration for detecting anomalies for pending pods. Expand the configuration for the “Detect pods stuck in pending”anomaly detection. The anomaly detector is configured to alert if there has been a pod stuck in pending for at least 2 minutes within the last 3 minutes as shown below.

![picture 0](../../assets/images/c17916d77b3594f5892f77edef4cc51837fbaae61a10a887c8ed7b715844452a.png)  

#### Kubernetes Explorer
- *Navigate* to the **Kubernetes Explorer**
- *Explore* the **unhealthy workloads** by directly *clicking* on the **number of unhealthy workloads** in the **health status** above the cluster table.

![picture 8](../../assets/images/90b498b162ec3f8a88e491a2906198c69830ab7a52ff8ba6011ffe4a7c929c0c.png)  

![picture 9](../../assets/images/a1c2d6991374195fb1a2c51fafdd68f074e4ccefa4c9a15672c7e23ccafa9338.png)  

*Analyze* the unhealthy workload **“frontend”** in namespace **“orders”**.

Dynatrace provides all **Observability signals** in context (metrics, logs, events, traces) of this workload and shows you why it is unhealthy.

![picture 10](../../assets/images/582c7d803b2910eb4c476549a8a42e9cb434a6704f0d155f36841f4efd920e15.png)  

![picture 11](../../assets/images/0d69502f43f3a97c746776d8a79d689bd71c49c1d96b1c662b952ee939e32935.png)  

In this case, the pod can’t be scheduled as there are **not sufficient resources available** on any node.

![picture 12](../../assets/images/585d35f6806e321b88fd172e8c318789764b56c375d7ed1e1a30dd8ca8c43ed1.png)  

Do you remember the initial EKS setup? Didn’t we have a **node autoscaler** in place? 
As it turns out, the **maximum Size** of the Autoscaling Group was already reached (2 nodes). Can we prevent such a problem in the future?

### K8s/AWS Workflows with AWS EC2 APIs

Knowing what Observability data is available within Dynatrace and how to manipulate it using **DQL** is a great first step. The next step is to understand **how to automate processes** within our organization using that information. Dynatrace allows users to **trigger actions from workflows** and apply **self-healing actions** so you don’t need to wake up SREs or Clouds Ops teams on weekends or during nights. 

We do not have enough time to create another workflow from scratch in this lab, so let’s **import a workflow from a template**. 

1.	*Download* the workflow template from [the GitHub repository](https://github.com/Epeiswerth/aws-modernization-dt-orders-setup-saas/blob/main/wftpl_manage-eks-autoscaling-group-size-workflow.yaml)
2.   *Open* **Workflow App** in Dynatrace
3.	*Select* **“Upload”** in the **upper right corner**
4.	*Select* and *open* the **manage-eks-autoscaling-group-size-workflow.yaml file** in the Github repository
5.	*Check* the **required apps** and *click* **“Next”**

![picture 13](../../assets/images/60864fdaad584194af25e4437de981d18029078855ff857b7dd2eb7eac3b257e.png)  

6.	*Select* the AWS connection **“PerformAWSConnect”** created in Lab 2 and *Import* Workflow

![picture 14](../../assets/images/fed1dcb35f26221c1f3e674ee0ed68d588092579aa79652dc3501d8c47ce3dac.png)  

Analyze each step in the workflow and *make sure* the **Davis Problem Trigger is enabled**.

![picture 15](../../assets/images/0b90d1a970e7f2b33cea3c7bfc6a5511ef695b55a4dbbbfc08120a7a7bb611bf.png)  

- **Davis Problem Trigger**
     - The workflow gets triggered **when Davis AI detects a reliability problem** in with a pod stuck in pending. For our example, we scoped the workflow action to only trigger this workflow if the problem occurs for the namespace of one of our most important apps called **“orders”**. 
- **Analyze Pending Pods**
     - In this initial step, we double-check whether there is a pending pod that is **not already marked for deletion**.  
- **Fetch node group labels**
     - Next, we determine the node group that is **most likely the one that lead to the pending pod** due to **insufficient resource coverage**. For this purpose, we’re analyzing the utilization of the nodes and take the node group including the node with the highest utilization. 
     - ***Note***: We took a shortcut in this example to keep the complexity of the workflow and the amount of instrunctions for this HoT session at a managebel level. To be 100% precise, we’d need to check the **taints** and **tolerations** of the pending pod as well as matching node groups. You can setup & connect [Dynatrace EdgeConnect](https://docs.dynatrace.com/docs/ingest-from/edgeconnect) for this purpose to retrieve the full yaml from the pod and nodes via Kubernetes API and use it in a workflow action. 
     - Moreover, we have some **exciting updates** in our roadmap section.
- **Describe Auto Scaling Group (ASG)**
     - We *use* the EC2 Action “Describe auto scaling group” to get the name of the EC2 autoscaling group based on the **EKS nodegroup name**. This action sends a request to the **AWS EC2 Autoscaling API**.
- **Analyze Auto Scaling Capacity**
     - This action executes **javascript code** where we analyze the current, desired and maximum capacity of the Autoscaling Group. In case the desired capacity is already at the **maximum level**, we *increase* the **maximum ASG size by 2**. 
     - ***Note***: If you don’t have a cluster autoscaler in place, you can update the desired capacity with this Dynatrace workflow, too. 
- **Upgrade Auto Scaling Group Max Size**
     - We *use* the EC2 Action “Update auto scaling group” to send a request to the **EC2 autoscaling API** and update the **ASG with the parameters defined** in the previous step. 

**Workflow Execution**

*Check* whether the workflow has been already **successfully executed**. If not, *select* **Davis problem trigger**, *click* on “**query past events**,” and then *run* the **workflow manually**.

![picture 16](../../assets/images/93fab182b507f0fb029a0285283d50ae3f383b5a0716653c6a2fb9a2c6b878ef.png)  

Check the number of Kubernetes nodes in the Dynatrace Kubernetes App (or AWS console). You should now see **3 Kubernetes nodes** and **no longer any pod stuck in pending**!

***Note***: The workflow has increased the maximum Size in the Autoscaling Group from 2 to 4 instances. It takes quite some time (~10 minutes) until this change is reflected in the EKS node group details displayed in the AWS Console. While the Autoscaling group information is updated ad-hoc, unfortunately you don’t have sufficient permissions with your workshop participant AWS user to access it.
