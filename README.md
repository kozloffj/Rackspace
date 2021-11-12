# Azure Cloud Architect Assessment

# Requirements

- Virtual Network with a private subnet and all required dependent infrastructure
- Azure Load Balancer with web server instances in the backend pool
  - Include a simple health probe to make sure the web servers in the backend pool are responding and healthy
  - The health probe should automatically replace instances if they are unhealthy, and the instances should come back into service on their own
- Virtual Machine Scale Set that launches Virtual Machines and registers them to the backend pool
  - Establish a VMSS minimum and desired server count meeting Highly Available and zone redundant requirement
  - Establish a VMSS maximum
  - Establish an Autoscale rule that scales up/down based on a metric of your choice (and be able to demonstrate a scaling event)
- Network Security Group allowing HTTP traffic to load balancer from anywhere (not directly to the instance(s))
- Network Security Group allowing only HTTP traffic from the load balancer to the instance(s)
- Remote management ports such as SSH and RDP must not be open to the world
- Some kind of automation or scripting that achieves the following:
  - Install a web server (your choice – Apache, Nginx, and IIS are common examples)
  - Deploys a simple &quot;hello world&quot; page for the web server to serve up
  - May be written in the language of your choice (HTML, PHP, etc)
  - May be sourced from the location of your choice (Git, cookbook file/ template, etc)
  - Must include the server&#39;s hostname in the &quot;hello world&quot; presented to the user
- All Azure resources must be created using Terraform or Azure Resource Manager
- No resources may be created or managed by hand in the portal, the Azure CLI, or with Powershell

# Architecture

![Application Architecture](https://github.com/kozloffj/Rackspace/blob/main/Architecture.png?raw=true)

Characteristics of the solution architecture

**Single Resource Group**

A single resource group was selected as the application is limited to a single tier and has minimal network security. With a single-tier application, the application administrators can be limited to DevOps, and network architecture utilizes NSGs and Bastion, a variety of access is not required.

**Single Virtual Network and Two Subnets**

A single VNet with two Subnets is deployed. The subnets are scoped to their specific objective: Bastion Host and VM Scale Set. Both Subnet&#39;s are associated with dedicated NSGs with hardened inbound and outbound security rules.

**Virtual Machine Scale Set**

A virtual machine scale set is defined as a Windows 2019 Second Generation server. The Virtual Machine Scale Set (VMSS) has the following characteristics

- Zone Redundant between Zones 2 and 3. Zones are limited to Zone 2 and 3 due to restrictions based on VM Size
- Auto-Scaling, minimum instances = 2, with Diagnostic Settings enables for tracking/monitoring
  - If CPU \&gt; 50%, scale up by 1
  - If CPU \&lt; 30% scale down by 1
- Overprovisioning is enabled if, during spin-up, an instance fails to increase the chance of successfully provisioning during peak usage.
- Configured Upgrade Policy with Health Monitoring (using Load Balancer health probe) that will automatically deploy new OS images every month to maintain compliance
- VM insights enabled, including VMInsights Solution deployment to Log Analytics Workspace

**Log Analytics Workspace**

Used to collect logs from a variety of sources.

**Storage Account (Classic)**

Log Analytics can store data for an extended retention period but has a high cost. To mitigate the cost of keeping only in Log Analytics, data in Log Analytics should be reserved for a maximum of 90 days. For long-term retention, typically required for audits, logs, and activities can additionally be forwarded to a Storage Account v1 (also known as Classic).

**Bastion Host**

A bastion host was selected as a method of access for DevOps engineers.

**Load Balancer and Public IPs**

The Load Balancer and Public IPs honor the concern of Zone Redundancy to minimize the risk of datacenter failure as a single point of failure.

# Method of Implementation

To automate deployment, Azure Resource Manager (ARM) Templates were developed. For optimal navigation and management, a combination of nested and linked templates was utilized. All files have been pushed to a public GitHub Repository (https://github.com/kozloffj/Rackspace).

The solution has also been deployed to Azure Subscription:

- **Subscription Name** : kozloff
- **Subscription ID** : 4edb1653-c1f5-41fb-9e99-92a72cec12d1
- **Resource Group** : rgSampleTest

# Access and Testing

To access the Virtual Machines, utilize the Bastion connection for each VMSS Instance. Login details are:

- **User Name:** SampleVMSSWebAdmin
- **Password:** JeffSampleVMssWeb!!

To view the site, go to: http://samplevmssweb2021.eastus2.cloudapp.azure.com/MyApp. The expected output should be as shown below, where [samplevms00007] will change per which machine is serving the request (tip: hold Ctrl and hit F5 for increased chance to have a request sent to a different VM).

![Sample of App Running](https://github.com/kozloffj/Rackspace/blob/main/SampleAppResult.png?raw=true)

To force spin-up of additional Virtual Machines, an increase of CPU utilization on the VM is required. This is performed by connecting to one or more VMSS Instances using Bastion, then run the PowerShell code found in this article - https://www.wellarchitectedlabs.com/performance-efficiency/100_labs/100_monitoring_windows_ec2_cloudwatch/5_generating_load/

# ARM Template Package

The ARM Template package is found here - https://1drv.ms/u/s!Aitr2fxTo4xjjvko\_xcwvZlAlXncug?e=W29ww6 (OneDrive)

NOTE: the template **ParentTemplate.json** is the root template, and all linked templates map to GitHub (https://github.com/kozloffj/Rackspace). Update ParentTemplate.json template links to run locally.

To run the IaC on a different subscription and use the public storage, use the following PowerShell command:

```
New-AzDeployment -Name "Initial-Deploy-1" -Location "East US 2" `
-TemplateUri "https://raw.githubusercontent.com/kozloffj/Rackspace/main/IaCWebApplicationDemo/ParentTemplate.json" `
-TemplateParameterUri "https://raw.githubusercontent.com/kozloffj/Rackspace/main/IaCWebApplicationDemo/ParentTemplate.paramaters.json" `
-DeploymentDebugLogLevel All
```

# Template Source

In addition to Microsoft Documentation, source template code was provided by Jason Boeshart&#39;s GitHub solution as part of the Microsoft Azure Quick Start Templates.

- **Solution Name** : Create/Upgrade a VM scale set running IIS configured for autoscale
- **URL** : https://github.com/Azure/azure-quickstart-templates/tree/master/demos/vmss-windows-webapp-dsc-autoscale

# Assessment Author

- **Name** : Jeff Colin Kozloff
- **LinkedIn** : https://www.linkedin.com/in/kozloffj/
