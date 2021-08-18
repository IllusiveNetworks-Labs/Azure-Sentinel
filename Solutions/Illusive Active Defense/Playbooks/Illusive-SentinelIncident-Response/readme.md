<p align="left">  
<img width="300" height="100" src="./Images/logo.jpg"> </a>
</p>

# Illusive Incident Response Playbook

The Incident Response playbook leverages Sentinel analytic rules and CrowdStrike or Microsoft Defender for Endpoint integration to automate incident response when specified Illusive incidents are discovered. 

Use this playbook to quickly stop or slow down ransomware attacks and critical incidents detected by Illusive in your organization. Upon detection, Sentinel is instructed to use the triggering process information reported by Illusive remove or kill the process. If the triggering process cannot be killed, Sentinel is instructed to isolate the host. These capabilities are available for organizations with CrowdStrike Falcon or Microsoft Defender for Endpoint.

 1. [Playbook workflow](#playbook-workflow)
 2. [Playbook execution](#playbook-execution)
 3. [Playbook output](#playbook-output)
 4. [Access Playbook](#Access_playbook)
 5. [Playbook retry mechanism](#playbook-retry-mechanism) 


## Playbook Workflow
 
 1. Perform the general solution setup. [(see instructions here)](https://github.com/IllusiveNetworks-Labs/Azure-Sentinel/tree/Illusive/Solutions/Illusive%20Active%20Defense)
 2. [Add API permissions to the Azure app](#add-api-permissions)
 3. [Enable Microsoft Defender for Endpoint](#enable-mde) (Only when using MDE for incident response) 
 4. [Create the Illusive playbook](#create-illusive-playbook)

<a name="add-api-permissions">
 
## Add API permissions to the Azure app 

 1. From the Azure console, find the Azure app you created to run the Illusive Sentinel Solution. 
 2. Go to <b>API Permissions</b>.
 3. Click <b>Add a permission</b>.
 4. Under <b>Request API permissions>API’s my organization uses</b>, search for and select <b>WindowsDefenderATP</b>, select select <b>Delegated permissions</b> and check the following permissions:
    - Machine.Isolate – to isolate device
    - Machine.Read – to find agent ID - to collect data from a single machine. 
    - File.Read.All – for process handling, find and erase/stop suspicious executables
    - Machine.StopAndQuarantine – for process handling, find and erase/stop suspicious executables
 5.	Select Application permissions and check the following permissions:
    - Machine.Isolate – to isolate device
    - Machine.Read.All – to find agent ID – to query all machines and collect device information even if we don’t have a device ID. 
    - File.Read.All – for process handling, find and erase/stop suspicious executables
    - Machine.StopAndQuarantine – for process handling, find and erase/stop suspicious executables
 6. Click <b>Add permissions</b>.
 7. Once all the API permissions are added, click <b>Grant admin consent for Default Directory</b> and click <b>Yes</b>.
 8. Verify admin consent has been granted. This step is important, even if the admin consent status is green. Only a Global Admin can approve admin consent requests.
       1. Go to <b>Enterprise>Admin Consent requests</b>.
       1. Go to <b>My pending</b> and verify that this permission is not pending.

The result should look like this: 
   <p align="center">  
      <img src="./Images/azure-app-api-incident-response-permissions-admin-consent-granted.png"> </a>
   </p>

<a name="enable-mde">
 
## Enable Microsoft Defender for Endpoint

Allow the Illusive Incident Response playbook to stop an attack by triggering an incident response from MDE. 

<b>Attention:</b> If you use CrowdStrike as your incident response tool, you can skip this procedure.

 1. From the Azure Search bar, search for the <b>Subscription</b> in which MDE is installed.
     <p align="center">  
        <img src="./Images/Configure_MDE_1(Subscriptions_MDE_1).png"> </a>
     </p>
 2. Click on the existing <b>Subscription.</b>
 3. Click <b>Security</b> in the Subscription menu.
 4. Ensure Microsoft Defender for Endpoint is <b>On.</b>
     <p align="center">  
        <img src="./Images/Configure_MDE_2(Subscriptions_MDE_2).png"> </a>
     </p>
 5. If MDE is off, click <b>Security Center.</b>
     <p align="center">  
        <img src="./Images/Configure_MDE_OFF_(Subscriptions_MDE_2).png"> </a>
     </p>
 6. Find the Azure Defender card and click <b>Enable Azure Defender.</b>
     <p align="center">  
        <img src="./Images/Configure_MDE_3(Security_Center)_Enable.png"> </a>
     </p>
 7. Select the desired subscription and click <b>Upgrade.</b>
     <p align="center">  
        <img src="./Images/Configure_MDE_3(Security_Center)_Upgrade.png"> </a>
     </p>

<a name="create-illusive-playbook">
 
# Create the Illusive Incident Response playbook

Deploying the Illusive Incident Enrichment playbook requires a custom deployment template. 
 - The playbook should be deployed under the same resource group, subscription, and workspace as the Azure app.
 - The Illusive API key should contain only the API key and no keywords such as “Bearer” or “Basic”.
 - You will not be prompted for missing information when saving the custom deployment configuration. If the playbook is incorrect or incomplete, the incident response playbook will not be able to isolate hosts, and you will get a playbook execution level error message. 
 - Though it is possible to enter integration information for both CrowdStrike and Microsoft Defender for Endpoint, the Illusive solution requires you to select just one tool for incident response.
 - Use the generic CrowdStrike API URL: https://api.crowdstrike.com. 
The playbook will fail to execute if the URL contains a hyphen  which is not supported by Sentinel (i.e., certain region-specific URLs). 

 1. On Azure home page, filter for <b>Deploy a custom template.</b>
     <p align="center">  
        <img src="./Images/deploy-custom-template-search.png"> </a>
     </p>
 2. Under <b>Custom Deployment>Select a template,</b> click <b>Build your own template in the editor.</b>
     <p align="center">  
        <img src="./Images/deploy-custom-template-page.png"> </a>
     </p>
 3. From <b>Edit template,</b> click <b>Load file,</b> the file named IllusiveSentinelIncidentResponse.json provided by Illusive and click <b>Save.</b>
     <p align="center">  
        <img src="./Images/deploy-custom-template-load-file.png"> </a>
     </p>
     <p align="center">  
        <img src="./Images/deploy-custom-template-edit-template-incident-response.png"> </a>
     </p>
 4. Under <b>Custom Deployment>Basics:</b>
    - Specify the <b>Subscription</b> that contains the dedicated Azure app that will run the Illusive Sentinel solution 
    - Specify the <b>Resource group</b> that contains the Workspace where you want to install the playbook.
    - Under <b>Instance details:</b>
      <table>
       <tr>
        <td><b>Field</b></td>
        <td><b>Instructions</b></td>
       </tr>
       <tr>
        <td>Region</td>
        <td>Filled automatically based on the subscription and cannot be changed.</td>
       </tr>
       <tr>
        <td>Workspace Name</td>
        <td>Specify the Azure Sentinel <b>Workspace Name</b> where you want to create the playbook.</td>
       </tr>
       <tr>
        <td>Illusive API URL <br/> Illusive API Key</td>
        <td>Supply the authentication parameters required to access the Illusive API
         <b>Important:</b> Enter the API key without the keyword</td>
       </tr>
       <tr>
        <td>Azure-Sentinel Client ID:  <br/> Azure-Sentinel Client Secret:  <br/> Azure-Sentinel Tenant ID:</td>
        <td>Supply the authentication parameters required to access the Azure-Sentinel API</td>
       </tr>
       <tr>
        <td>EDR deployed</td>
        <td>The EDR which is deployed in the organization and can be used for incident mitigation</td>
       </tr>
       <tr>
        <td>CrowdStrike API URL <br/> CrowdStrike Client ID <br/> CrowdStrike Client Secret</td>
        <td>If <b>EDR deployed = CrowdStrike,</b> specify CrowdStrike authentication parameters</td>
       </tr>
       <tr>
        <td>Azure MDE Client ID <br/>Azure MDE Client Secret <br/>Azure MDE Tenant ID</td>
        <td>If <b>EDR deployed = MDE,</b> specify MDE authentication parameters</td>
       </tr>
      </table>
      <p align="center">  
         <img src="./Images/custom-deployment-basics-incident-response.PNG"> </a>
      </p>      
 5. When finished entering details, click <b>Review + Create.</b>
      <p align="center">  
         <img src="./Images/custom-deployment-review-create.png"> </a>
      </p>      
 6. On successful validation, click <b>Create.</b>
This completes the playbook deployment. 
      <p align="center">  
         <img src="./Images/custom-deployment-is-complete.png"> </a>
      </p>      
 7. To view the playbook, click <b>Go to resource group.</b>
   - If there is only one installed playbook in the workspace, clicking on Go to resource group will take you to the playbook page. 
   - If there are multiple installed playbooks in the workspace, clicking on <b>Go to resource group</b> will take you to the <b>All resources page.</b> The deployed playbook will be available in the list.

<a name="playbook-execution">

## Playbook Execution 
 1.	This playbook is triggered by a new Sentinel Alert originating from a new Illusive event syslog.
 2.	Sentinel uses Illusive API to fetch the incident details and determine whether this is a Ransomware or a Critical severity incident.
 3.	In either of the above cases, the playbook determines the response as follows:
    a.	If no response has been executed, the playbook tries to stop the triggering process by either killing or deleting it (depending on the integrated EDR). The information about the triggering process is extracted from the Illusive API and the response executed on the triggering process of the most current event.
    b.	If an attempt to stop the triggering process has been made in response to a previous event, or if there is presently no option to stop the process, the playbook tries to isolate the host.
    c.	<b Important>: If an attempt to isolate the host has been made in response to a previous event, the playbook will take no further action on this host for future detected events.
 4.	The playbook leverages the response capabilities of Crowdstrike or Microsoft Defender for Endpoint, whichever is configured in the custom playbook deployment.

<a name="Access_playbook">
  
## Access and view the playbook 

You can view and manage the playbook as well as review the playbook run history. This can be helpful for troubleshooting or for understanding playbook behavior and results. 

1. Find the playbook on the Azure Sentinel or All resources page. 
2. Click on the playbook to view the playbook run History.
3. Select any executed playbook to view the results.

Sample playbook history:
<p align="center">  
   <img src="./Images/playbook-history-incident-response.PNG"> </a>
</p>

<a name="playbook-retry-mechanism">

## Playbook retry mechanism

<p align="center">  
   <img src="./Images/playbook-retry-mechanism.png"> </a>
</p>

Azure Sentinel handles the retry mechanism. If any condition is not met, Sentinel retries the playbook four times.
