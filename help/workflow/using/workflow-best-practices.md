---
title: Workflow best practices
seo-title: Workflow best practices
description: Workflow best practices
seo-description: 
page-status-flag: never-activated
uuid: 1f905bcd-0c30-4d89-8147-9ea0ec0a4a8b
contentOwner: sauviat
products: SG_CAMPAIGN/CLASSIC
content-type: reference
discoiquuid: 90dc528e-aef5-4d1a-955a-bef1513ec2ed
index: y
internal: n
snippet: y
---

# Workflow best practices{#workflow-best-practices}

## Execution and performance {#execution-and-performance}

General guidelines about optimizing Campaign performance, including best practices to apply to your workflows, are listed in [this document](https://helpx.adobe.com/campaign/kb/best-practices-for-performance-improvement.html). Additional recommendations are listed below.

### Logs {#logs}

The JavaScript method **logInfo()** is a great solution for debugging a workflow. It is useful but it must be used carefully, especially for activities that are frequently run: it can overload the logs and significantly increase the size of the log table. But you might also need more than **logInfo()**.

Two additional solutions are available to help:

* **Keep the result of interim populations between two executions**

  This option keeps temporary tables between two executions of a workflow. It is available in the workflow properties' **General** tab, and can be used for development and test purpose to monitor data and check results. You can use this option in development environments, but never use it on production environments. Keeping temporary tables could result in the size of the database increasing significantly and eventually the size limit being reached. Moreover, it will slow down the backup.

  Only the working tables of the last execution of the workflow are kept. Working tables from previous executions are purged by the **cleanup** workflow, which runs on a daily basis.

  >[!CAUTION]
  >
  >This option must never be checked in a production workflow.

* **Log SQL queries in the journal**

  Available in the **Execution** tab of workflow properties, this option will log all SQL queries generated by the tool from the different activities. It is a good way to see what is actually executed by the platform. However, this option should only be used temporarily during development and not activated on production.

### Workflow planning {#workflow-planning}

* Try to maintain a stable level of activity along the day and avoid peaks to prevent the instance from overload. To do so, distribute workflow starting times evenly throughout the day.
* Schedule data load overnight to reduce resource contention.
* Long workflows can potentially have an impact on the server and database resources. Split the longest workflows to reduce processing time.
* To reduce overall run times, replace time-consuming activities with simplified and faster activities.

### Workflow execution {#workflow-execution}

It is a best practice not to schedule a workflow to run more than every 15 minutes because it may impede overall system performance and create blocks in the database.

Also, avoid leaving your workflows in a paused state. If you create a temporary workflow, make sure it will be able to finish correctly and not stay in a **paused** state. If it is paused, it would imply that you need to keep the temporary tables and thus increase the size of the database.

To avoid having workflows in a paused state:

* Check your workflows on a regular basis to ensure there are no unexpected errors.
* Keep your workflows as simple as possible, for example by splitting large workflows in several different workflows. You can use **External signal** activities trigger their execution based on other workflows' execution.

### Execute in the engine option {#execute-in-the-engine-option}

In the **Workflow properties** window, never check the **Execute in the engine** option. When this option is enabled, the workflow takes priority and all other workflows are stopped by the workflow engine until this one is finished.

![](assets/wf-execute-in-engine.png)

## Workflow properties {#workflow-properties}

### Workflow folders {#workflow-folders}

Adobe recommends you to create your workflows in a dedicated folder.

If the workflow affects the whole platform (cleansing processes for example), you can consider adding a sub-folder in the built-in **Technical Workflows** folder.

### Workflow naming {#workflow-naming}

Because it makes them easier to find and troubleshoot if they are not performing in the expected ways, Adobe recommends to give your workflows proper names and labels: fill in the workflow's description field to summarize the process to be performed so that the operator can easily understand it.

If the workflow is part of a process involving multiple workflows, you can be explicit when entering a label; using numbers is a great way to order the workflows (by Label).

For example:

* 001 - Import - Import recipients
* 002 - Import - Import sales
* 003 - Import - Import sales details
* 010 - Export - Export delivery logs
* 011 - Export - Export tracking logs

### Workflow severity {#workflow-severity}

You can configure the severity of a workflow in the workflow properties, in the **Execution** tab:

* Normal
* Production
* Critical

Providing this information when creating a workflow will help you understand the severity of the process configured.

This option has no functional impact on workflows other than campaign workflows.

Campaign workflows (workflows created as part of a campaign/operation) with a higher severity are executed in priority in case the campaign has many processes supposed to run simultaneously. By default, only 10 processes can run simultaneously in a campaign, according to the option NmsOperation_LimitConcurrency. For example, if a campaign contains 25 workflows, workflows with a higher severity will then be executed in the first pool of 10 processes.

### Workflow monitoring {#workflow-monitoring}

All your scheduled workflows running on production environments should be monitored in order to be alerted if there is an error.

In the workflow properties, select a Supervisor group, either the default **Workflow supervisors** or a custom group. Make sure that at least one operator belongs to this group, with an email set up.

Before you start building a workflow, remember to define workflow supervisors. They will be notified by email in case of errors. For more on this, refer to [Managing errors](../../workflow/using/workflow-best-practices.md#managing-errors).

Regularly check the **Monitoring** universe to view the overall status of the active workflows. For more on this, refer to [Instance supervision](../../workflow/using/workflow-best-practices.md#instance-supervision).

The Workflow HeatMap enables the Adobe Campaign platform administrators to monitor the load on the instance and plan workflows accordingly. For more on this, refer to [Workflow monitoring](../../production/using/monitoring-processes.md#workflow-monitoring).

## Using activities {#using-activities}

### Name of the activity {#name-of-the-activity}

While developing your workflow, all activities will have a name, as will all Adobe Campaign objects. While the name is generated by the tool, we recommend you rename it with an explicit name when configuring it. The risk with doing it later is that it may interrupt the workflow with activities using the name of another previous activity. So it would be a difficult job to update the names afterward.

The activity name can be found in the **Advanced** tab. Don’t leave them named **query**, **query1**, **query11**, but give them explicit names such as **querySubscribedRecipients**. This name will show up in the journal, and if applicable in the SQL logs, and this will help to debug the workflow when configuring it.

### First and last activities {#first-and-last-activities}

* Always start your workflow with a **Start** activity or a **Scheduler** activity. When relevant, you can also use an **External signal** activity.
* When building your workflow, only use one **Scheduler** activity per branch. If the same branch of a workflow has several schedulers (linked to each other), the number of tasks to be executed will be multiplied exponentially, which would considerably overload the database. This rule also applies to all activities with a **Scheduling & History** tab.

  ![](assets/wf-scheduler.png)

* Use **End** activities for every workflow. This lets Adobe Campaign free up temporary space used for calculations within workflows. For more on this, refer to: [Start and end](../../workflow/using/start-and-end.md).

### Javascript within an activity {#javascript-within-an-activity}

You may want to add JavaScript when initializing a workflow activity. This can be done in an activity's **Advanced** tab of the activity.

To make spotting the workflow easier, we recommend using double dashes at the start and end of the activity label as follows: -- My label --.

### Signal {#signal}

Most of the time, you will not know where the signal is called from. In order to avoid this problem, use the **Comment** field within the **Advanced** tab of the signal activity to document the expected origin of a signal for this activity.

![](assets/workflow-signal-bp.png)

## Workflow update {#workflow-update}

A production workflow should not be updated directly. Unless the process consists of creating a campaign with template workflows, processes should first be tested on a development environment. After this validation, the workflow can be deployed and started on production.

Archived workflows may be kept on development or test platforms, in an Archived folder, but production environment should stay as clean as possible. Old workflows should be removed from production environment if they are inactive.