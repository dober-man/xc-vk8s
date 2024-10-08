# Workloads, Deployments, Stateful Sets or Daemon Sets?
All are valid methods to deploy the application with each offering it's own unique capabilities. 

 **Workloads** are a non-native Kubernetes construct and are an abstract (XC) definition of groups of objects to be deployed. Workloads are similar to deployments but one logical layer above them.....or an umbrella of vK8s objects, making it easier for users to interact with Kubernetes resources in a more simplified, higher-level manner. You would use the XC API or Console UI to deploy and manage workloads. More info [here](https://docs.cloud.f5.com/docs-v2/api/views-workload) and [here](https://docs.cloud.f5.com/docs-v2/platform/reference/api-ref/ves-io-schema-views-workload-api-replace).

 
**Deployments** are a native Kubernetes construct and best used when managing stateless applications. You could simply paste your deployment definition YAML into XC Console or leverage kubectl to imperatively create the deployment. You do not deploy workloads with kubectl. These will be covered in detail in the section 3. 

<img width="1124" alt="image" src="https://github.com/user-attachments/assets/ac1ea09b-1811-487f-a8ec-af229ff5ba66">
<br>
<br>

**StatefulSets and DaemonSets are covered in section 4 & 5 of this guide.**

> **Note:** The test container/image used in this setup is a public nginx container that runs unpriviledged. This is necessary per the restrictions listed above in the diagram. The container natively starts on a high unpriviledged port (8080) that does not require root to bind to. 

## Workloads
Starting with the most commonly used method to deploy vK8s services, we will use a workload to 
define and create our entire sample nginx application which includes the deployment, pods, service, replication sets,  and service mesh with optional volumes, load balancer and origin pool.

Distributed Apps -> Applications -> Virtual K8s -> "Click on your vK8s name". 

From the Workloads tab, click "Add vK8s workload" and use the screenshot to configure the initial form. Choose **Service**. 

<img width="957" alt="image" src="https://github.com/user-attachments/assets/57886aee-84df-40cd-979b-e322b0060650">

Click the blue "Configure" under "Service". 

Containers -> Add Item
Image Name: ```ghcr.io/nginxinc/nginx-unprivileged:1.27.1-bookworm-perl```
<img width="745" alt="image" src="https://github.com/user-attachments/assets/fb24118b-09e4-4d38-8d6f-3cdfab89f835">

Deploy Options: 
Choose: "Regional Edge Virtual Site"
Click the blue "Configure" under "Regional Edge Virtual Sites"
<img width="845" alt="image" src="https://github.com/user-attachments/assets/b141c359-068f-4b37-937a-977b8e845803">

Use the USA based regional edge virtual site: 

<img width="336" alt="image" src="https://github.com/user-attachments/assets/29061427-6e81-4401-9502-6c2ae33ca6c0">

#### Advertise Options - Advertise on Internet (AoI) vs Advertise in Cluster (AiC)

**Advertise on Internet (AoI)** 

AoI does exactly what it sounds like. AoI preconfigures a service to expose the pods in vK8s, an origin pool referencing the service, and finally a public, globally-redundant load balancer. 

AoI objects are created and managed by the workload. For example, you can not directly modify the load balancer or change or add anything outside of what is offered in the AoI config definition form. This is similar to the iApp feature on BIG-IP. When we choose AoI, we limit the amount of steps we need to take to deploy and publish an app but we also are limited in terms of adding any additional functionality to the load balancer in support of the app.

**Advertise in Cluster (AiC)**

This setting is similar to AoI and while this automatically creates a vK8s service, it does not create an origin pool or load balancer. It will be up to the admin to define the origin pool, create the load balancer, add security policy and deploy it to the Virtual Site appropriate to provide access to the service. For example, a load balancer running on a CE, publishing services to other clusters or to internal clients. 

The benefit of this model is the entire suite of XC security services can be configured on the load balancer that weren't exposed in the AoI model. You can add WAAP, API Discovery, Bot mitigation and a whole bunch of other XC security services to protect the app. You can modify the load balancer and origin pool at whim. There are some restrictions around which security features are available when running the load balancer on the CE, but those are outside the scope of this lab which is 100% Regional Edge Deployment. 

Either method (AoI or AiC) could be used to publish the app or service but which method you use will determine which post-deployment features are available for the service. We will explore the configuration and outcome of each method now. 

**AoI Method**

For Advertise Options, select "Advertise on Internet" from the dropdown and click the blue "Configure". 

<img width="514" alt="image" src="https://github.com/user-attachments/assets/e4d70a63-7316-4e2a-87e1-b7dfb10e9f2a">


Define the port information and then click the blue "Configure" under HTTP/HTTPS Load Balancer.

<img width="836" alt="image" src="https://github.com/user-attachments/assets/25423fbb-5416-4517-a8a6-08014c8131fb">

Configure the load balancer as shown below. 
<img width="831" alt="image" src="https://github.com/user-attachments/assets/0dcc11b8-815c-4244-8feb-e10add310602">

Click Apply, Apply, Apply, and finally Save and Exit. 

Within a few moments you should see your workload status 

<img width="1121" alt="image" src="https://github.com/user-attachments/assets/d1d1c004-ec91-4ae1-b775-3a022aaa7831">

Click on the "Deployments" tab and review the deployment created as part of the workload. Click the "3 dots" under the Actions column to review the actions you can take.  

<img width="1110" alt="image" src="https://github.com/user-attachments/assets/ee122d06-4db3-4164-8723-63d8ad8c7cca">

Click "Edit" and review the YAML. Change the number of replicas from 1 to 2 and click "Save".

<img width="926" alt="image" src="https://github.com/user-attachments/assets/f2530b38-c8bd-42cc-8b39-71c1757e6c12">

Over the next 30 seconds or so you will see the number of pods double. There are now 10 pods across 5 sites. 

<img width="552" alt="image" src="https://github.com/user-attachments/assets/fd43db43-ed9e-481e-8494-70cafb7fa4c1">

Click on the "Services" tab and review the services deployed as part of the workload. Notice the default kubernetes service which allows 443->6443 access to the kube-api. 

<img width="954" alt="image" src="https://github.com/user-attachments/assets/9baae205-b236-4560-b6d8-2138fedd002d">

Click on the "Replica Sets" tab and review the Replica Sets deployed as part of the workload. 

<img width="1124" alt="image" src="https://github.com/user-attachments/assets/fe850d59-0c42-4cf8-b5b2-b7cb6fe308ca">

Click on the "Pods" tab and review the Pods deployed as part of the workload then click the "3 dots" for one of the pods under the Actions column to review the actions you can take. 

<img width="1117" alt="image" src="https://github.com/user-attachments/assets/5ecdd4d2-a4de-4b23-811e-61ae13ce81e0">

From the left menu go to Manage -> Load Balancers -> HTTP Load Balancers

You will see the load balancer that was automatically created as part of the workload definitions. You can also take a look under "Origin Pools" to see a similar object. 

<img width="1122" alt="image" src="https://github.com/user-attachments/assets/0a142113-0862-4feb-b7e2-20df3f1286f2">

Click the "3 dots" under the Actions column and notice that there is no option to manage or edit the load balancer or origin pool directly. To modify any aspect of either, it would need to be initiated from the workload definitions. 

<img width="1109" alt="image" src="https://github.com/user-attachments/assets/e6a93474-893d-40c4-a040-fac247fe9dc1">

### Testing Access to the Service
Click the little down arrow next to the Load Balancer.

<img width="1102" alt="image" src="https://github.com/user-attachments/assets/445ebb78-3fb4-4422-a8c7-724162c1c7c5">

Scroll down to around line 95 and find your IP address. 

<img width="657" alt="image" src="https://github.com/user-attachments/assets/6217ee74-02e8-4ec1-8808-935b9ba85880">

Make a host file entry on your local machine to point nginx.example.com to the IP address in your config.

<img width="771" alt="image" src="https://github.com/user-attachments/assets/b66a9b93-2e54-4772-a6a3-fd6fc8ece100">

### Reviewing Service Mesh
When deploying apps in vK8s, a service mesh is automatically created. Now is a good time to review that functionality. 

On the local host run a curl loop to generate some traffic. 
```
while true; do
  curl http://nginx.example.com/
  sleep 0.5
```
Wait a few moments and then on the left Nav menu under "Mesh" click on "Service Mesh" and then click "default".
<img width="487" alt="image" src="https://github.com/user-attachments/assets/b6a8474a-944c-4a44-839d-04aa16f0418a">

Click on the icon for the service and click "Health". 

<img width="1129" alt="image" src="https://github.com/user-attachments/assets/3c508686-8d2f-4f95-bcd3-e881a9a9b3db">

Review the stats and click close. Click the "Requests" tab at the top and review the requests. 

<img width="1135" alt="image" src="https://github.com/user-attachments/assets/e005c4c5-a6b8-47ee-9094-96cedbcce930">

Click back on the "Service Graph" tab but this time **double-click** on the icon for the service. This is a view of all the RE sites defined in the virtual site that is hosting this vK8s app. 

<img width="790" alt="image" src="https://github.com/user-attachments/assets/d225690c-c86e-4997-8441-bf30c7331e2a">


**AiC Method**

Testing the AiC method simply involves making a change to the workload and manually building an origin pool and load balancer. The real benefit of this method will be clear momentarily. 

Navigate to Home -> Distributed Apps -> Applications -> Virtual K8s -> click on your vK8s name, click the "Workloads" tab and then click the actions button on the far right under your workload name. Choose "Manage Configuration". 

<img width="1119" alt="image" src="https://github.com/user-attachments/assets/6ca64a5d-5ef4-4198-9e3e-b25f832244e5">

Edit Configuration

<img width="1174" alt="image" src="https://github.com/user-attachments/assets/d91da46a-6b8c-41ea-b656-270e80a474b5">

Edit Service Configuration

<img width="583" alt="image" src="https://github.com/user-attachments/assets/4f9b8a39-c5c5-4066-b7c9-c481fd74bd82">

Advertise Options: Advertise in Cluster - click the blue "Configure"

<img width="501" alt="image" src="https://github.com/user-attachments/assets/0e9fe677-e746-4b34-8ae1-79e94caa5cf3">

Advertise in cluster config

<img width="839" alt="image" src="https://github.com/user-attachments/assets/03495427-7cca-418f-8f2e-5218b287c768">

Apply, Apply, Save & Exit. 

Test access to the site again. 

<img width="745" alt="image" src="https://github.com/user-attachments/assets/f4f68964-c831-4af3-b1e7-b820d980cfe9">

Why the 404? Unlike AoI, AiC deploys the workload but does not automatically create the Origin Pool and Load Balancer. While this is an extra step for the admin, it allows for a much more feature-rich front-end as all the normal capabilties are available for the load balancer. 

From the left menu go to Manage -> Load Balancers -> Origin Pools and configure the pool exactly as shown in the screenshot. 

<img width="846" alt="image" src="https://github.com/user-attachments/assets/3127a05e-1361-458b-a8f4-ba76da287174">
<img width="840" alt="image" src="https://github.com/user-attachments/assets/1ea7b815-bfdc-4359-9502-b9c56d63aae6">

Click Save and Exit.

From the left menu go to Manage -> Load Balancers -> HTTP Load Balancers and configure the load balancer exactly as shown below, then click Save and Exit. 

<img width="844" alt="image" src="https://github.com/user-attachments/assets/82ef492b-853b-482b-b7dd-9d1d544b383e">
<img width="839" alt="image" src="https://github.com/user-attachments/assets/ba065d53-ff7e-44bc-b46e-31cce45cf17f">

> **Note:** Everything past "Origins" is optional, so take all the defaults. However, note the plethora of options and security features available. This is the tradeoff between AoI and AiC. AoI is fast and easy but provides no security. AiC involves two extra steps to create the origin pool and load balancer, but this could easily be automated and offers a lot more in terms of security and function.

Test access by browsing to: http://nginx.example.com

<img width="686" alt="image" src="https://github.com/user-attachments/assets/0ecd2c51-2f49-4338-8529-64f2952ac282">

**This concludes the Workloads learning section. Continue to 3-deployments.**
