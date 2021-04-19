# Overview
Multi-use Hybrid + Identity Cyber Range implementing a small Active Directory Domain in Azure alongside Azure AD and Azure Domain Services.  Automated templates for building your own Pentest / Red Team / Cyber Range in the Azure cloud!  Purple Cloud is a small Active Directory enterprise deployment automated with Terraform / Ansible Playbook templates to be deployed in Azure.  Purple Cloud also includes an adversary node accessible over RDP as well as a SIEM, DFIR, & Live Response system (Velociraptor + HELK).

![](images/pce.png)

# Use Cases
* Research and pentest lab for Azure AD and Azure Domain Services
* Security testing of Hybrid Join and Azure AD Joined devices 
* EDR Testing lab 
* Enterprise Active Directory lab with domain joined devices
* Malware / reverse engineering to study artifacts against domain joined devices
* SIEM / Threat Hunting / DFIR / Live Response lab with HELK + Velociraptor [1, 2]
* Log aggregator architecture to forward logs to a cloud native SIEM (Azure Sentinel)
* Data Science research with HELK server, Jupyter notebooks
* Detection Engineering research with Mordor [3, 4]


# Features and Information
* **New Feature:**  Azure Active Directory terraform module:  Deploys Azure Active Directory users, groups, applications, and service principals
* **New Feature:**  Azure Domain Services terraform module:  Deploys Azure AD Domain Services for your managed AD in the Azure cloud
* **New Feature:**  Three tools for Adversary Simulation: Script to automatically invoke Atomic Red Team unit tests using Ansible playbook.
* **New Feature:**  Velociraptor [1] + Hunting ELK [2] System: Windows 10 Endpoints instrumented with agents to auto register Velociraptor and send Sysmon logs
* Pentest adversary Linux VM accessible over RDP
* Deploys one Linux 18.04 HELK Server.  Deploys HELK install option #4, including KAFKA + KSQL + ELK + NGNIX + SPARK + JUPYTER + ELASTALERT
* Windows 10 endpoints with Sysmon (SwiftOnSecurity) and Winlogbeat
* Windows 10 endpoints are automatically configured to use HELK configuration + Kafka Winlogbeat output to send logs to HELK
* Automatically registers the endpoint to the Velociraptor server with TLS self-signed certificate configuration
* Windows endpoint includes Atomic Red Team (ART), Elastic Detection RTA, and APTSimulator
* One (1) Windows 2019 Domain Controller and one (1) Windows 10 Pro Endpoint (with three more that can be easily enabled)
* Automatically joins all Windows 10 computers to the AD Domain (with option to disable Domain Join per machine)
* Uses Terraform templates to automatically deploy in Azure with VMs
* Terraform templates write customizable Ansible Playbook configuration
* Post-deployment Powershell script provisions three domain users on the 2019 Domain Controller and can be customized for many more
* Azure Network Security Groups (NSGs) can whitelist your source prefix (for added security)
* Post-deploy Powershell script that adds registry entries on each Windows 10 Pro endpoint to automatically log in each username into the Domain as respective user.  This feature simulates a real AD environment with workstations with interactive domain logons.  
* Default Modules (Enabled):  One Windows Server, One Windows 10 Endpoint, One Velociraptor + HELK Server
* Extra Modules (Disabled):  One Linux Adversary, Three Windows 10 Endpoints, Azure AD, Azure AD Domain Services
* **Approximate build time:**  16 minutes

# Infrastructure and Credentials
* **All Domain User passwords:**  Password123
* **Domain:**  RTC.LOCAL
* Local Administrator Credentials on all Windows OS Systems
  * RTCAdmin:Password123
* **Subnets**
  * 10.100.1.0/24 (Server Subnet with Domain Controller, HELK + Velociraptor)
  * 10.100.10.0/24 (waf subnet - currently reserved)
  * 10.100.20.0/24 (AD Domain Services (If Enabled))
  * 10.100.30.0/24 (User Subnet with Windows 10 Pro)
  * 10.100.40.0/24 (Linux Adversary (If Enabled)
  * 10.100.50.0/24 (db subnet - currently reserved)
* **Velociraptor + HELK Internal IP**
  * 10.100.1.5
* **Windows Server 2019**  
  * 10.100.1.4
* **HELK + Velociraptor Linux OS username**  
  * helk (Uses SSH public key auth)
* **HELK Kibana Administrator Password for https port 443**  
  * helk:hunting
* **Velociraptor GUI Administrator Password for Port 8889**  
  * vadmin:vadmin
* **Azure Active Directory**
  * Users created:  25
  * Groups created:  8
  * Applications created:  2
  * Service Principals created:  2
  * Module Directory:  /modules/azure_ad/

# Remote Access
* **Velociraptor + HELK Server:**  View contents of modules/velocihelk-vm/hosts.cfg.  The second line should show the IP address of the Velociraptor + HELK server that is provisioned a public IP from Azure.  You can SSH to the host from within that directory:
```
$ ssh -i ssh_key.pem helk@<IP ADDRESS>
```

* **Kibana GUI:**  Use the step above to get the public Azure IP address of the HELK Server.  Use Firefox browser to navigate to:
```
https://<IP ADDRESS>
```
* **Velociraptor GUI:**  Use the step above to get the public Azure IP address of the Velociraptor Server.  Use Firefox browser to navigate to:
```
https://<IP ADDRESS>:8889
```
* **Windows Systems:**  For remote RDP access to the Windows 2019 Server or Windows 10 Pro endpoints:  Change into correct modules directory and view contents of hosts.cfg.  The second line should show the public IP address provisioned by Azure.  Just RDP to it with local Admin credentials above.  The Windows Server will be located in the ```/modules/dc-vm/hosts.cfg```.  The Windows endpoints will be in the directory:  ```/modules/win10-vm```.  Depending on which Windows 10 module is enabled, the following files will specify the public IP address for which host you are attempting to RDP into:
```
hosts-Win10-John.cfg
hosts-Win10-Lars.cfg
hosts-Win10-Liem.cfg
hosts-Win10-Olivia.cfg
```

# Installation
**Note:**  Tested on Ubuntu Linux 20.04 

## Requirements
* Azure tenant and subscription
* Global Administrator role
* Terraform:  Tested on v0.14.7
* Ansible:  Tested on 2.9.6

## Note for Azure AD Module:  Azure Active Directory Permission Requirements
If you want to enable and use the Azure Active Directory module, it requires your service principal to have special permissions.  There are two options listed below.  Either one of them will work just fine.  Both are not required:

**Method #1:**  Assign the "Global Administrator" role to the service principal

**Method #2:**  Assign the following API permissions to the service principal:
* Azure Active Directory Graph --> Application.ReadWrite.All
* Azure Active Directory Graph --> Directory.ReadWrite.All
* Microsoft Graph --> User.Read

This is what it looks like:

![](images/aad_permissions.png)



## Note for Azure AD DS Module:  Azure AD Domain Services Permission Requirements
If you want to enable and use the Azure AD Domain Services, it requires your service principal to have special permissions.  The only requirement here is that your Service Principal must have the "Global Administrator" role assigned to it.

## Installation Steps

**Note:**  Tested on Ubuntu 20.04

**Step 1:** Install Terraform and Ansible on your Linux system

Download and install Terraform for your platform --> https://www.terraform.io/downloads.html

Install Ansible
```
$ sudo apt-get install ansible
```

**Step 2:** Set up an Azure Service Principal on your Azure subscription that allows Terraform to automate tasks under your Azure subscription.


Follow the exact instructions in this Microsoft link:
https://docs.microsoft.com/en-us/azure/developer/terraform/getting-started-cloud-shell

These were the two basic commands that were run based on this link above:
```
az ad sp create-for-rbac --role="Owner" --scopes="/subscriptions/<subscription_id>"
```
and this command below.  From my testing I needed to use a role of "Owner" instead of "Contributor".  Default Microsoft documentation shows role of "Contributor" which resulted in errors.  
```
az login --service-principal -u <service_principal_name> -p "<service_principal_password>" --tenant "<service_principal_tenant>"
```
Take note of the following which we will use next to configure our Terraform Azure provider:
```
subscription_id = ""
client_id = ""
client_secret = ""
tenant_id = ""
```

**Step 3:** Clone this repo
```
$ git clone https://github.com/iknowjason/PurpleCloud.git
```

**Step 4:** First, copy the terraform.tfexample to terraform.tfvars.  Next, using your favorite text editor, edit the terraform.tfvars file for the Azure resource provider matching your Azure Service Principal credentials.  

```
cd PurpleCloud/deploy
cp terraform.tfexample terraform.tfvars
vi terraform.tfvars
```

Edit these parameters in the terraform.tfvars file, replacing "REPLACE_WITH_YOUR_VALUES" to correctly match your Azure environment.  
```
arm_client_id = "REPLACE_WITH_YOUR_VALUES"
arm_client_secret = "REPLACE_WITH_YOUR_VALUES"
subscription_id = "REPLACE_WITH_YOUR_VALUES"
tenant_id = "REPLACE_WITH_YOUR_VALUES"
```

Your terraform.tfvars file should look similar to this but with your own Azure Service Principal credentials:
```
arm_client_id = "7e9c2cce-8bd4-887d-b2b0-90cd1e6e4781"
arm_client_secret = ":+O$+adfafdaF-?%:.?d/EYQLK6po9`|E<["
subscription_id = "aa9d8c9f-34c2-6262-89ff-3c67527c1b22"
tenant_id = "8b6817d9-f209-2071-8f4f-cc03332847cb"
```

**Step 5a: (Optional Azure AD)** If you wish to enable Azure AD, configure the Azure AD provider service principal credentials in the terraform.tvfars file.  Edit these parameters, replacing "REPLACE_WITH_YOUR_VALUES" to correctly match your Azure environment.  These credentials are necessary for the Azure AD provider to use the service principal and create users, groups, and applications with the correct API permissions.

```
aad_client_id = "REPLACE_WITH_YOUR_VALUES"
aad_client_secret = "REPLACE_WITH_YOUR_VALUES"
```

Your terraform.tfvars file should look similar to this but with your own credentials:
```
aad_client_id = "7e9c2cce-8bd4-887d-b2b0-90cd1e6e4781"
aad_client_secret = ":+O$+adfafdaF-?%:.?d/EYQLK6po9`|E<["
```
**Step 5b: (Optional Azure AD)** Uncomment the Azure AD module in main.tf.  It is disabled by default.  Look for this section block and remove the multi-line comments.  It should look like this:

```
##########################################################
## Azure AD Module - Create Azure AD Users, Groups, and Application
##########################################################
module "azure_ad" {
  source              = "../modules/azure_ad"
  upn_suffix          = local.upn_suffix
  tenant_id           = local.tenant_id
}
```

**Step 5c: (Optional Azure AD)** In main.tf, set the following two variables correctly for Azure AD:
* tenant_id
* upn_suffix

The tenant_id variable should be the username associated with your tenant account when it was setup and it forms the universal principal name (UPN) suffix for your AD users.  Let's say your username is acmecorp.  Then the upn suffix will be acmecorp.onmicrosoft.com.  If you have added a custom domain to Azure, then you can use that custom domain as your upn suffix instead of 'tenant_id.onmicrosoft.com'.


**Step 6a: (Optional Azure AD Domain Services)** Azure Domain Servies is disabled by default.  If you wish to enable Azure AD Domain Services, uncomment the Azure AD Domain Services module in main.tf.  Look for this section block and remove the multi-line comments.  It should look like this at the beginning and end of the module:
```
module "azure_adds" {
<Skipping Lines>
}
```

**Step 6b: (Optional Azure AD Domain Services)** In main.tf, set the following two variables correctly for Azure AD Domain Services:
* tenant_id_ds
* upn_suffix_ds

**Step 7:**  Edit the terraform.tfvars file to include your source network prefix for properly white listing Azure Network Security Groups (NSGs).
Edit the following file:  deploy/terraform.tfvars
At the bottom of the file, uncomment the "src_ip" variable and populate it with your correct source IP address.  If you don't do this, the Azure NSGs will open up your two VMs to the public Internet.  Below is exactly where the variable should be uncommented and an example of what it looks like:
```
# Set variable below for IP address prefix for white listing Azure NSG
# uncomment variable below; otherwise, all of the public Internet will be permitted
# https://ifconfig.me/
# curl https://ifconfig.me
src_ip = "192.168.87.4"
```

**Step 8:** Run the commands to initialize terraform and apply the resource plan

```
$ cd PurpleCloud/deploy
$ terraform init
$ terraform apply -var-file=terraform.tfvars -auto-approve
```

This should start the Terraform automated deployment plan

**Step 9:** Optional:  Unzip and run Badblood from C:\terraform directory (https://github.com/davidprowe/BadBlood)


# Running APT Simulation Tools
This project includes three security tools to run APT simulations for generating forensic artifacts in an automated way.  Here is a quick walkthrough on the three tools that are automatically deployed.  To test efficacy of the detection solution, it is recommended to disable Windows Defender real-time protection setting.  This will allow the simulation tools to run in an environment that will allow them to fully execute, allowing you to look deeper at the forensic artifacts.

**1.  Atomic Red Team (ART)**

The Atomic Red Team scripts are downloaded from the official Github repo [5] and the Invoke-AtomicRedTeam execution framework is automatically downloaded and imported from the following repo [6].  This allows you to more easily run atomic tests and the modules are imported into the powershell session everytime you launch a powershell session.  This is controlled from the following powershell environment script:

```C:\Users\RTCAdmin\Documents\WindowsPowerShell\Microsoft.Powershell_profile.ps1```

Now that this is out of the way, let's show how to run an atomic test for ART!

**Remotely Running Atomics from Linux**

First, there is a python script that you can run to remotely invoke ART from your linux system.  It's a simple wrapper to Ansible Playbook and the location of the script is here:
```
PurpleCloud/modules/win10-vm/invoke-art.py
```
Run it like this:
```
python3 invoke-art.py
```
The script looks for all hosts.cfg files in the working directory and runs the atomic tests against all Windows 10 hosts.  If you only want to run against one of the hosts, run it with the -h flag.  For example:
```
python3 invoke-art.py -h hosts-Win10-Liem.cfg
```
Running it with the -a flag will specify an atomic.
```
python3 invoke-art.py -a T1558.003
```
The script looks for the atomic tests in windows index file:
```
/modules/win10-vm/art/atomic-red-team/atomics/Indexes/Indexes-CSV/windows-index.csv

```

**Manually Running Atomics from the Windows Endpoint**

RDP into the Windows 10 endpoint.  From a powershell session, simply run:
```PS C:\ > Invoke-AtomicTest <ATOMIC_TEST> -PathToAtomicsFolder C:\terraform\ART\atomic-red-team-master\atomics```


The atomics are in the main project directory path of ```C:\terraform\ART\atomic-red-team-master\atomics```.  Browse through them to find which atomic test you want to run.


Example of running T1007: 

```PS C:\Users\RTCAdmin> Invoke-AtomicTest T1007 -PathToAtomicsFolder C:\terraform\ART\atomic-red-team-master\atomics```

**2.  Elastic Detection Rules RTA (Red Team Attacks) scripts**

In June of 2020, Elastic open sourced their detection rules, including Python attack scripts through the Red Team Automation (RTA) project.  The following repo [7] is automatically downloaded and extracted using Terraform and Ansible scripts.  To run them, launch a cmd or powershell session and use python to run each test from the following directory:

Change into the directory:  

```C:\terraform\Elastic_Detections\detection-rules-main```

Run each python script test that you wish.  Each test is in the RTA directory and you invoke the test by removing the *.py (TTPs are referenced as a name by just removing the last *.py from the script):

```PS C:\terraform\Elastic_Detections\detection-rules-main> python -m rta <TTP_NAME>```

Example of 'smb_connection' ttp:

```PS C:\terraform\Elastic_Detections\detection-rules-main> python -m rta smb_connection```

You can browse all TTPs in the 'rta' sub-directory

**3.  APTSimulator**

The APTSimulator tool [8] is automatically downloaded.  Simply extract the Zip archive and supply the zip password of 'apt'.

```C:\terraform\APTSimulator.zip```

Invoke a cmd prompt and run the batch file script:

```C:\terraform\ATPSimulator\APTSimulator\APTSimulator.bat```


# Known Issues

"Error:  Adding group members"
While adding members to Group with ID "xxxxx" waiting for group membership

This issue happens when running terraform apply and it is related to timing with Azure AD Groups.

Fix:  run "terraform apply" again

# References

[1] Velociraptor Website:  https://www.velocidex.com/

[2] HELK Website:  https://github.com/Cyb3rWard0g/HELK

[3] Mordor Website:  https://mordordatasets.com/introduction.html

[4] Mordor Use Case:  https://posts.specterops.io/enter-mordor-pre-recorded-security-events-from-simulated-adversarial-techniques-fdf5555c9eb1

[5] Atomic Red Team:  https://github.com/redcanaryco/atomic-red-team

[6] Invoke ART:  https://github.com/redcanaryco/invoke-atomicredteam

[7] Elastic Detection Rules:  https://github.com/elastic/detection-rules

[8] APTSimulator:  https://github.com/NextronSystems/APTSimulator

# Future Ideas for Consideration
1. Automatically configure AD Connect client to synchronize Azure AD and Domain Controller
2. Automatically Azure Hybrid Join Windows 10 Pro endpoints
3. Pick a few C2s to auto deploy with terraform

# Credits

@ghostinthewires for his Terraform templates (https://github.com/ghostinthewires)

@mosesrenegade for his Ansible Playbook integration with Terraform + Powershell script (https://github.com/mosesrenegade)

