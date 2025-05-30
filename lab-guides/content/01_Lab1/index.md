## Lab 1 – Azure Cloud Visibility 

### Introduction

In this lab we will use multiple resources including:
* AKS Cluster
* Azure Monitor
* Logs 
* SLOs
* Clouds

### Creating a Notebook to Analyze Logs

1.	*Open* the **Notebooks app**. We will create a notebook to **analyze** the following; **logs coming from our Azure integration**, and **logs coming from our application deployed in AKS**.
2.	*Create* a **new notebook**.

![picture 0](../../assets/images/cd627fab51ed754524173018f3950af297a103ecb1d1d0d6b4e50dba9ef0f82c.png)  

3.	In our **new notebook**, let's *add* a **new section of DQL**.

![picture 1](../../assets/images/dcb321d46d6f21be8a5c0271e4144d4a0ccbbef830f37acd1170d7ca563c453a.png) 

4.	Inside the **DQL section** that we added, let’s *run* the **following query** that fetches all the records of azure using an azure subscription id as filter:

```
fetch logs
| filter azure.subscription == "7A91E8B3-EE09-4906-91A2-F357B77A61FD"
```

5.	We can use this query to look at all the **logs coming from our azure integration**. We can also *add* **filters** for resource group, resource id, resource name or type. 

```
fetch logs
| filter azure.subscription == "7A91E8B3-EE09-4906-91A2-F357B77A61FD"  and azure.resource.group == "SCW-GROUP-JOEBAILEY"
```

6.	Try to *add* a **new property to your query to filter your results**.
7.	We are also monitoring logs coming from our **AKS cluster**. Let's *add* **another DQL section** to our notebook to *check* these logs with the **next DQL query**.

```
fetch logs
| filter matchesPhrase(k8s.cluster.name, "perform") 
| sort timestamp desc
```

8.	This query is showing **logs** coming from our **AKS cluster** with the name "**perform**". In all these logs we can find useful information about **infrastructure performance** or **business-related** information.
9.	In the **AKS cluster** we have a **hipstershop application** running. This application writes logs that **contains business information** like successful transactions or revenue. Let's *add* another **DQL section** to our notebook, and *run* the **following query** to see the number of succussful transactions.

```
fetch logs
| filter contains(content, "payment") and k8s.namespace.name == "hipstershop"
| parse content, "json:j"
| fields timestamp, loglevel, content, message=j[message], traceid=j[dt.trace_id]
| summarize transactions=count()
```

10.	We have the information, but what about **adding some style** to the visualization? *Go to* the **options button** of our **DQL section** and *change* the **visualization to single value**.

![picture 2](../../assets/images/fb93a05319679f376c155b8d58b76c6859f436304ac3e6a31187043af3a11203.png)  

![picture 3](../../assets/images/6a63b69d823ca9b377b3a35374adc04bf330899d774967df22e9c9ba51e8d69c.png)  

Better, right?

11.	Now, what about **analyzing the revenue** of these transactions? *Add* another **DQL section** to your notebook and *paste* the **following DQL query**.

```
fetch logs
| filter contains(content, "payment") and k8s.namespace.name == "hipstershop"
| parse content, "json:j"
| fields timestamp, loglevel, content, message=j[message], traceid=j[dt.trace_id]
| parse message, "LD 'USD', double:Amount "
| summarize total_revenue=sum(Amount)
```

12.	Let's reconfigure the visualization to **single value**. Let's also go to the bottom of the **options menu** and *select* the **units and formats option**.

![picture 4](../../assets/images/924c1cbf5c55ad9ff2c0bb39b93dba977c85d0f85e2278f032d34742dbf37264.png)  

13.	*Add* an **override**:

![picture 5](../../assets/images/97f500c840eed7d67fa3d7b4773af5f3a4afdef64111330e1635254742894045.png)  

14.	*Change* the **unit to USD**. Try *typing* **USD in the search option**

![picture 6](../../assets/images/0972b5c23175a5448bfc44f76a5eead1c48beb7c32b1eb59d953cb37957370bc.png)  

That looks better.

15.	The idea of using notebooks is to enhance the collaboration in this kind of analysis. Let's *share* the **notebook with the rest of the users** in the environment. At the **top of your notebook**, first *change* the **name** of your notebook. Currently you'll see the title "**Untitled notebook**", *click* on **it** and *change* the **name** to **[your-first-name]-perfom2025**.

![picture 7](../../assets/images/421d8cfd4e5a1775a901f9c7350cb66c79c4ad56ace0e900614abb9a38564c35.png)  

16.	*Click* on the **share button**

![picture 8](../../assets/images/10da6296def3c59bc9d92eb181b602af798ae7f9f5c799c5e39ce84adab85ddd.png)  

17.	Once in **the share menu**, *click* on the **gear icon**

![picture 9](../../assets/images/18977e31597a04fd82425e4a3a782b783a8844f1de4d0e71be90bbd01e154e09.png)  

18.	Finally, *turn on* the **“Visible to anyone in your environment” toggle**.

![picture 10](../../assets/images/8a40eaca147eff8657deff1aea7cfcd3498d0938015e40b09a441311f77b01db.png)  

![picture 11](../../assets/images/c2ed58035d278386dda1f714b175ee8223f0128e08c629fca6928df2e0fe9a45.png)  

![picture 14](../../assets/images/18977e31597a04fd82425e4a3a782b783a8844f1de4d0e71be90bbd01e154e09.png)  

### SLOs
Now we will *create* **Service Level Objectives** (**SLOs**) to *track* the performance of our newly deployed **Azure Cloud**. **Service Level Objectives** will *ensure* that we are maintaining our **standards** and **best practices**.
1.	*Open* up the **Service-level Objectives** (**SLO**) app. 
2.	*Create* a new **SLO**.

![picture 15](../../assets/images/0ae6a9a73c6ba3d309261791c098326dc5030a9b99063acc8c755423eab3d90d.png)  

3.	While Dynatrace offers **preset SLO options** to help you get you started, we will be creating **custom SLOs**. *Select* the **custom SLO** option.

![picture 16](../../assets/images/9491869d2b47a780c51dfadc09c8db5833eb3f5f1c24728d08a3aa4be87c1fd6.png)  

4.	Here we can *add* our **custom DQL**.
5.	For our first example, we will be *tracking* the number of **Successful EventHub requests**. To do this, we will need to *utilize* **several metrics** that come built-in with the **Azure Event Hub Integration**. 
    * **dt.cloud.azure.event_hub.requests.successful**  - *Tracks* the number of **successful EventHub requests**
    * **dt.cloud.azure.event_hub.errors.server** – *Tracks* the number of **EventHub server errors**
    * **dt.cloud.azure.event_hub.errors.user**  - *Tracks* the number of **EventHub User errors**
    * **dt.cloud.azure.event_hub.errors.quota_exceeded** – *Tracks* the number of **Quota Exceeded errors**
    * **dt.cloud.azure.event_hub.requests.throttled** – *Tracks* **throttled EventHub requests**
6.	*Add* the **following DQL** to *build* your **SLO**: 

```
 timeseries {Successful = sum(dt.cloud.azure.event_hub.requests.successful),
ServerErrors = sum(dt.cloud.azure.event_hub.errors.server),UserErrors = sum(dt.cloud.azure.event_hub.errors.user),QuotaExceededErrors = sum(dt.cloud.azure.event_hub.errors.quota_exceeded),Throttled = sum(dt.cloud.azure.event_hub.requests.throttled)}
| fieldsAdd sli = (((Successful[] - ServerErrors[]- UserErrors[] - QuotaExceededErrors[] - Throttled[]) / Successful[]) * (100))
| fieldsRemove Successful, ServerErrors, UserErrors, QuotaExceededErrors, Throttled
```

7.	*Refresh* the **preview** to ensure that the **SLO** is *reporting* results correctly.

![picture 17](../../assets/images/d7deda0069a386f0c9541f5ff0421061d5fdc6d0b1abd41fcd0feb83eed8678c.png)  

8.	*Hit* ‘**Next**’. Now, we will *set* our **Target threshold**, **warning threshold**, and **evaluation period**. The **target value** can be a standard set by **your company** or a standard set by **the industry**. We will *use* **98%**. *Toggle* the show **warning** option:

![picture 18](../../assets/images/0dfd0a02f8bf8f1ada54c1f4f4f5feeb3b84b243187ca562c6f3be5d9d928e6e.png)  

9. *Set* the **target** to **99%**. Finally, *choose* your **evaluation period**. For this example, we can *use* the **default**, **7 days**.
10.	*Hit* ‘**Next**’. *Give* your **SLO** a **name**, **description**, and any relevant **tags**, and *hit* ‘**Save**’. 
11.	Your first **SLO** is now complete! We will *repeat* this process **twice** more. 
12.	Our next **SLO** will *track* the number of **Azure MySQL** **active connections** versus the **number of aborted connections**. *Repeat* the **steps** from the 1st **SLO** but use this **DQL**: 
```
timeseries { ActiveConnections = sum(cloud.azure.microsoft_dbformysql.flexibleservers.active_connections), AbortedConnections = sum(cloud.azure.microsoft_dbformysql.flexibleservers.aborted_connections) }
| fieldsAdd sli = (((ActiveConnections[] - AbortedConnections[])/ ActiveConnections[]) * (100))
| fieldsRemove ActiveConnections, AbortedConnections
```

13.	We will *repeat* the same **process** for our third **SLO**. The last **SLO** will track our **Azure App Service** and track the % of requests that are *less* than **500ms**. *Use* the **following** **DQL**: 

```
timeseries rsp_time = avg(dt.cloud.azure.app_service.response.avg), default:0
| fieldsAdd high=iCollectArray(if(rsp_time[]> (1000 * 500), rsp_time[]))
| fieldsAdd low=iCollectArray(if(rsp_time[]<= (1000 * 500), rsp_time[]))
| fieldsAdd highRespTimes=iCollectArray(if(isNull(high[]),0,else:1))
| fieldsAdd lowRespTimes=iCollectArray(if(isNull(low[]),0,else:1))
| fieldsAdd sli=100*(lowRespTimes[]/(lowRespTimes[]+highRespTimes[]))
| fieldsRemove rsp_time, high, low, highRespTimes, lowRespTimes
```
14.	We now have 3 **SLOs** set up to *track* various performance objectives across our **Azure** deployment. 
15.	Now, *add* your **SLOs** to a **dashboard** to *display* with other metrics!

![picture 19](../../assets/images/86011d72f2ab316f39143821e1bedf0a2a2425433a037beb945d6d14a1e97c5b.png)  

### Lab 1 Conclusion
In this **lab** we have successfully *created* a **notebook** to *analyze* **logs** and then **SLOs** to *track* our **Azure** deployment performance.

