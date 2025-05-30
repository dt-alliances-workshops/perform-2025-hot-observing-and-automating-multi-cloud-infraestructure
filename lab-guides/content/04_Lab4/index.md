## Lab 4 – AWS Infra Logs via Firehose
### Introduction 

In this lab, we will learn how to integrate **Cloud platform events** with **Dynatrace**. **Platform events** are mostly generated in the form of **logs** when **services** are being instantiated. It's critical that we get these **logs** and **events** into a **centralized platform** so that the ops team can **troubleshoot with ease**. 

For the hands-on part of this lab, we will showcase the integration with **AWS services via CloudWatch** and **Data Firehose**. We will get various **logs from EKS** and at the end of the lab you will be able to see these logs within Dynatrace in near real-time.

Lab Prerequisites

* *Access* to **AWS account** with the relevant **permissions** to *create* an **EKS cluster** and **Kinesis Data Firehose**
* Able to *navigate* through **AWS console** with ease

### Lab Steps

We will follow instructions from **Dynatrace** **documentation** to *create* the **Firehose Delivery Pipeline**: [https://docs.dynatrace.com/docs/ingest-from/amazon-web-services/integrate-with-aws/aws-logs-ingest/lma-stream-logs-with-firehose](https://docs.dynatrace.com/docs/ingest-from/amazon-web-services/integrate-with-aws/aws-logs-ingest/lma-stream-logs-with-firehose)

***Tip***: To successfully *execute* the request below, you need an **access token** with ```logs.ingest``` scope.

To learn how to *create* **API** key, see this: [https://docs.dynatrace.com/docs/manage/identity-access-management/access-tokens-and-oauth-clients/access-tokens#create-api-token](https://docs.dynatrace.com/docs/manage/identity-access-management/access-tokens-and-oauth-clients/access-tokens#create-api-token )
 
***Step 1.*** *Execute* the below **commands** while in the **AWS console** (note the **live.dynatrace.com URI, and NO trailing “/”)

``` 
DYNATRACE_API_URL=<<your_API_URL>>
``` 
(e.g., DYNATRACE_API_URL=https://sur12345.live.dynatrace.com)
``` 
DYNATRACE_API_KEY=<<your_API_token>>
```
(e.g., DYNATRACE_API_KEY=dt0a01.sur12345.1211asdsdfaesdf56...asdfasdfdf) 
** Note: This api key is available in your dashboard under ApiTokenDashboard)
```
STACK_NAME=dynatrace-log-delivery-stream
```
 
``` 
wget -O dynatrace-firehose-log-stream.yaml https://assets.cloud.dynatrace.com/awslogstreaming/dynatrace-firehose-log-stream.yaml && \
aws cloudformation deploy --capabilities CAPABILITY_NAMED_IAM --template-file ./dynatrace-firehose-log-stream.yaml --stack-name $STACK_NAME --parameter-overrides DtApiUrl=$DYNATRACE_API_URL DtApiToken=$DYNATRACE_API_KEY 
```

Sample Output:

```
CloudWatchSubscriptionFilterRoleArn
arn:aws:iam::133220408703:role/dynatrace-log-delivery-stream-CloudWatchLogsRole-wgg3reyD7A73
FirehoseArn
arn:aws:firehose:ap-northeast-1:133220408703:deliverystream/dynatrace-log-delivery-stream	 
```

***Step 2.*** Once the **Firehose Stream** is in “**active**” state, test the connection between **AWS** and **Dynatrace** by *sending* **demo data**.
- *Open* **Data Firehose** in **Amazon Console**.
- *Click* the “**dynatrace-log-delivery-stream**" you created
- *Click* “**Test with demo data**”

![picture 0](../../assets/images/657ffc968c98eb54f1cb999e2a0e086f0101367b51e73303429cbb7969725e46.png)  

- Inside your **DT tenant**, *open* **Logs App**
- *Create* **logs filter** with the **ARN** for **data firehose**
- *Use* the **ARN of your resource to filter** the logs

![picture 1](../../assets/images/f6665fa51f2140612a34a5e5186d73cc6f93eac893c85d10ff6cc8889e3f9f07.png)  

![picture 2](../../assets/images/0019ab8ac3c910e81b8d9212608fa151531ec4a68edfebaddad1abf566e2d4d9.png)  

![picture 3](../../assets/images/eac288d32f8bedc3773940f922fe3f089160826ee2555b25f59e30595f2c09df.png)  

***Step 3.*** Now let’s onboard the logs from the **EKS cluster** into Dyntrace.
   
In the **EKS cluster** you created earlier, *navigate* to the “**Observability**” tab and *open* “**Manage logging**”

![picture 4](../../assets/images/9f988d6a568b622d64cdf74f87e6097a97a6ddf73f401cc034d5d28b7d950d12.png)  

*Select* the **sources** you’d like to **log**, and *save* your **changes**. 

In the below screenshot you see that all the **platform-level logs** are *selected*:

![picture 5](../../assets/images/ed5674a2c6fa3ea31af6ef211a057f5f48db0ee0ff4cc78a1a86a19acbcc25ee.png)  

***Step 4.*** Let’s *subscribe* to the **log group** where **EKS** *sends* those **platform logs**.
- *Open* **CloudWatch**
- *Navigate* to **Log Groups**

![picture 6](../../assets/images/5e91d57471ac04f0dbeecc76391ad46d8756480b3c6063890123c180acdaef4f.png)  

![picture 7](../../assets/images/1ccc775b5968647d32ee07ab43ecdbec5ee88e195e4a12da3cc7b4ad117c0d15.png)  

![picture 8](../../assets/images/bc4dc365a08de04d37827267216c593885b793ad5904a25d6be2871dd2780978.png)  

*Execute* the below commands on your **AWS Cloud shell**:
``` 
wget -O dynatrace-firehose-logs.sh https://assets.cloud.dynatrace.com/awslogstreaming/dynatrace-firehose-logs.sh && chmod +x dynatrace-firehose-logs.sh 
``` 

``` 
./dynatrace-firehose-logs.sh subscribe --log-groups /aws/eks/dynatrace-workshop/cluster 
``` 

![picture 9](../../assets/images/70daf7167dea9997b366b9d10588006810e2be6e0c3b6b1da7955cfdba902e99.png) 

Once you have **subscribed**, you can see those **logs** within your **Dynatrace tenant**:

![picture 10](../../assets/images/89989c8926f143a8b28d2257b8e4a447dd845b55fc45318767b10176a58d6b5e.png)  

<details>
<summary>Optional Appendix for Lab 4</summary>
Use this guide in the case where your **EKS cluster is not running** and you need to **create a new one** just to complete this lab.

***Step 1***. *Open* **AWS Cloudshell**

![picture 0](../../assets/images/1c3eeeee02688d530dfd2ed77d83c64390915f8b859c964c36c05555c954f258.png)  

***Step 2.*** The **eksctl CLI** is used to work with **EKS clusters**. It automates many individual tasks. 
 
The below commandline is to *install* **EKSCTL**. These are taken from: [https://eksctl.io/installation/](https://eksctl.io/installation/)

```
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum –check
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin
```

***Step 3.*** You can now go ahead and *create* a **cluster using the below command**. Remember to *change* the **name of the cluster and the region**.

```
eksctl create cluster --name suraj-hot-perform2025 --region ap-northeast-1 --fargate
``` 

***Step 4.*** You should be able to see the progress of the **EKS cluster creation** within **CloudFormation** -> **Stacks**.

![picture 1](../../assets/images/fb2aada6e9380609086c0d50b6b08a65242228ba27fd25027db0669569370dc8.png)  

***Step 5.*** *Clean up* **EKS**

```
eksctl delete cluster --name suraj-hot-perform2025 --region ap-northeast-1
```
</details>
