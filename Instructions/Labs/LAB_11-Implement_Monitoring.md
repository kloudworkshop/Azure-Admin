---
lab:
    title: 'Lab 11: Implement Monitoring'
    module: 'Administer Monitoring'
---

# Lab 11 - Implement Monitoring

## Lab introduction

In this lab, you learn about Azure Monitor. You learn to create an alert and send it to an action group. You trigger and test the alert and check the activity log.  

This lab requires an Azure subscription. Your subscription type may affect the availability of features in this lab. You may change the region, but the steps are written using **UK South**.

## Estimated timing: 40 minutes

## Lab scenario

Your organization has migrated their infrastructure to Azure. It is important that Administrators are notified of any significant infrastructure changes. You plan to examine the capabilities of Azure Monitor, including Log Analytics.

## Architecture diagram

![Diagram of the architecture tasks](../media/az104-lab11-architecture.png)

## Job skills

+ Task 1: Use a template to provision an infrastructure.
+ Task 2: Create an alert.
+ Task 3: Configure action group notifications.
+ Task 4: Configure an alert processing rule.
+ Task 5: Use Azure Monitor log queries.
+ Task 6: Trigger an alert and confirm it is working.

## Task 1: Use a template to provision an infrastructure

In this task, you will deploy a virtual machine that will be used to test monitoring scenarios.

1. If necessary, download the **\\Allfiles\\Lab11\\az104-11-vm-template.json** lab files to your computer.

1. Sign in to the **Azure portal** - `https://portal.azure.com`.

1. From the Azure portal, search for and select `Deploy a custom template`.

1. On the custom deployment page, select **Build you own template in the editor**.

1. On the edit template page, select **Load file**.

1. Locate and select the **\\Allfiles\\Labs11\\az104-11-vm-template.json** file and select **Open**.

1. Select **Save**.

1. Use the following information to complete the custom deployment fields, leaving all other fields with their default values:

    | Setting       | Value         | 
    | ---           | ---           |
    | Subscription  | Your Azure subscription |
    | Resource group| `az104-rg11` (If necessary, select **Create new**)
    | Region        | **UK South**   |
    | VM Size | Select an available size. Use **Standard_D2s_v5** if available.|
    | Username      | **localadmin**   |
    | Password      | **Pa55w.rd123abc** |
    
1. Select **Review + Create**, then select **Create**.

    >**NOTE:** The template provides three current VM sizes. Start with        **Standard_D2s_v5**. If the deployment fails because the size is unavailable or Azure lacks capacity, select **Standard_D2s_v6** and redeploy to the same resource group. If necessary, retry with **Standard_D2s_v7**. If a retry fails because an existing or partially deployed resource causes a conflict, delete **az104-rg11**. Restart Task 1 from **Search for and select Deploy a custom template**, reload the template, select Create new to recreate **az104-rg11**, and deploy again with the selected VM size.  

1. Wait for the deployment to finish, then click **Go to resource group**.

## Verify the deployment
1. On the az104-rg11 resource group page, confirm that the following resources exist:

    + Virtual machine **az104-vm0**
    + Log Analytics workspace with a name that begins with **az104-law11-**
    + Data collection rule **az104-dcr11**
    + The virtual network, network interface, public IP address, network security group, and storage account

1. Open **az104-vm0**.

1. If the virtual machine status is Stopped, select Start and wait for its status to change to Running.

1. Under **Settings**, select **Extensions + applications**.

1. Confirm that **AzureMonitorWindowsAgent** has a status of **Provisioning succeeded**.

1. Return to **az104-rg11**, and then open **az104-dcr11**.

1. Under **Configuration**, select **Resources**, and then confirm that **az104-vm0** is associated with the data collection rule.

    > **NOTE:** Data can take several minutes to appear after the Azure Monitor Agent and data collection rule are deployed. Do not delete the virtual machine until you complete Task 2.

## Task 2: Create an alert

In this task, you create an alert for when a virtual machine is deleted. 

1. Continue on the **Monitor** page , select **Alerts**. 

1. Select **+ Create** and select **Alert rule**. 

1. Select the box for the resource group, then select **Apply**. This alert will apply to any virtual machines in the resource group. Alternatively, you could just specify one particular machine. 

1. Select the **Condition** tab and then select the **See all signals** link.

1. Search for and select **Delete Virtual Machine (Virtual Machines)**. Notice the other built-in signals. Select **Apply**

1. In the **Alert logic** area (scroll down), review the **Event level** selections. Leave the default of **All selected**.

1. Review the **Status** selections. Leave the default of **All selected**.

1. Leave the **Create an alert rule** pane open for the next task.

## Task 3: Configure action group notifications

In this task, if the alert is triggered send an email notification to the operations team. 

1. Continue working on your alert. Select **Next: Actions**, and then select **+ Create action group**.

    >**Did you know?** You can add up to five action groups to an alert rule. Action groups are executed concurrently, in no specific order. Multiple alert rules can use the same action group. 

1. On the **Basics** tab, enter the following values for each setting.

    | Setting | Value |
    |---------|---------|
    | **Project details** |
    | Subscription | your subscription |
    | Resource group | **az104-rg11** |
    | Region | **Global** (default) |
    | **Instance details** |
    | Action group name | `Alert the operations team` (must be unique in the resource group) |
    | Display name | `AlertOpsTeam` |

1. Select **Next: Notifications** and enter the following values for each setting.

    | Setting | Value |
    |---------|---------|
    | Notification type | Select **Email/SMS message/Push/Voice** |
    | Name | `VM was deleted` |

1. Select **Email**, and in the **Email** box, enter your email address, and then select **OK**. (Use a personal email, not a company or the QA Admin email)

    >**Note:** You should receive an email notification saying you were added to an action group. There may be a few minutes delay, but that is a sure sign the rule has deployed.

1. Select **Review + create** to validate your input, then select **Create** (To create the Action group)

1. Once the action group is created move to the **Next: Details** tab and enter the following values for each setting.

    | Setting | Value |
    |---------|---------|
    | Alert rule name | `VM was deleted` |
    | Alert rule description | `A VM in your resource group was deleted` |

1. Select **Review + create** to validate your input, then select **Create**.

## Task 4: Configure an alert processing rule

In this task, you create an alert rule to suppress notifications during a maintenance period. 

1. Continue in the **Alerts** blade, select **Alert processing rules** and then **+ Create**. 
   
1. Select your **resource group**, then select **Apply**.
   
1. Select **Next: Rule settings**, then select **Suppress notifications**.
   
1. Select **Next: Scheduling**.
   
1. By default, the rule works all the time, unless you disable it or configure a schedule. You are going to define a rule to suppress notifications during overnight maintenance.
Enter these settings for the scheduling of the alert processing rule:

    | Setting | Value |
    |---------|---------|
    | Apply the rule | At a specific time |
    | Start | Enter tomorrows's date at 01:00 |
    | End | Enter tomorrow's date at 07:00 |
    | Time zone | Select the local timezone. |

    ![Screenshot of the scheduling section of an alert processing rule](../media/az104-lab11-alert-processing-rule-schedule.png)

1. Select **Next: Details** and enter these settings:

    | Setting | Value |
    |---------|---------|
    | Resource group | **az104-rg11** |
    | Rule name | `Planned Maintenance` |
    | Description | `Suppress notifications during planned maintenance.` |

1. Select **Review + create** to validate your input, then select **Create**.


## Task 5: Use Azure Monitor log queries

1. In this task, you verify that the Azure Monitor Agent is sending heartbeat and VM performance data to the Log Analytics workspace. Complete this task before deleting the virtual machine.

1. In **az104-rg11**, open the Log Analytics workspace whose name begins with **az104-law11-**.

1. 1. Under **General**, select **Logs**.

1. Close the welcome window or Queries hub if either appears.

1. If necessary, select KQL mode from the query editor mode menu.

1. Replace any text in the query editor with the following query, and then select Run.

    ```bash
    Heartbeat
    | where TimeGenerated > ago(30m)
    | where Computer =~ "az104-vm0"
    | summarize HeartbeatCount = count(), LastHeartbeat = max(TimeGenerated)
        by Computer, Category
    ```

1. Replace the query with this one, and then click **Run**. Review the resulting chart.

   ```bash
    InsightsMetrics 
    |where TimeGenerated > ago(1hr)
    |project TimeGenerated, Name, Val
    |render timechart 
   ```

1. If you have time, review and run other queries.

    >**Note**: Notice you can configure an alert rule based on a query. 

## Task 6: Trigger an alert and confirm it is working

In this task, you trigger the alert and confirm a notification is sent. 

>**Note:** If you delete the virtual machine before the alert rule deploys, the alert rule might not be triggered. 

1. In the portal, search for and select **Virtual machines**.

1. Check the box for the **az104-vm0** virtual machine.

1. Select **Delete** from the menu bar.

1. Check the box for **Apply force delete**. Enter `delete` to confirm and then select **Delete**. 

1. In the title bar, select the **Notifications** icon and wait until **vm0** is successfully deleted.

1. You should receive a notification email that reads, **Important notice: Azure Monitor alert VM was deleted was activated...** If not, open your email program and look for an email from azure-noreply@microsoft.com.

    ![Screenshot of alert email.](../media/az104-lab11-alert-email.png)
   
1. On the Azure portal resource menu, select **Monitor**, and then select **Alerts** in the menu on the left.

1. You should have three verbose alerts that were generated by deleting **vm0**.

   >**Note:** It can take a few minutes for the alert email to be sent and for the alerts to be updated in the portal. If you don't want to wait, continue to the next task and then return. 

1. Select the name of one of the alerts (For example, **VM was deleted**). An **Alert details** pane appears that shows more details about the event.


## Cleanup your resources

If you are working with **your own subscription** take a minute to delete the lab resources. This will ensure resources are freed up and cost is minimized. The easiest way to delete the lab resources is to delete the lab resource group. 

+ In the Azure portal, select the resource group, select **Delete the resource group**, **Enter resource group name**, and then click **Delete**.
+ Using Azure PowerShell, `Remove-AzResourceGroup -Name resourceGroupName`.
+ Using the CLI, `az group delete --name resourceGroupName`.

## Key takeaways

Congratulations on completing the lab. Here are the main takeaways for this lab. 

+ Alerts help you detect and address issues before users notice there might be a problem with your infrastructure or application.
+ You can alert on any metric or log data source in the Azure Monitor data platform.
+ An alert rule monitors your data and captures a signal that indicates something is happening on the specified resource.
+ An alert is triggered if the conditions of the alert rule are met. Several actions (email, SMS, push, voice) can be triggered.
+ Action groups include individuals that should be notified of an alert.

## Learn more with self-paced training

+ [Improve incident response with alerting on Azure](https://learn.microsoft.com/en-us/training/modules/incident-response-with-alerting-on-azure/). Respond to incidents and activities in your infrastructure through alerting capabilities in Azure Monitor.
+ [Monitor your Azure virtual machines with Azure Monitor](https://learn.microsoft.com/en-us/training/modules/monitor-azure-vm-using-diagnostic-data/). Monitor your Azure VMs by using Azure Monitor to collect and analyze VM host and client metrics and logs.
