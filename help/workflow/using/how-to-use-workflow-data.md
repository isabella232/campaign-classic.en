---
title: How to use workflow data
seo-title: How to use workflow data
description: How to use workflow data
seo-description: 
page-status-flag: never-activated
uuid: ed03f14b-1b53-426e-9213-22cb2f3deb19
contentOwner: sauviat
products: SG_CAMPAIGN/CLASSIC
audience: workflow
content-type: reference
topic-tags: -general-operation
discoiquuid: ec3844ca-8d80-4ddc-b08c-f18a6919bb28
index: y
internal: n
snippet: y
---

# How to use workflow data{#how-to-use-workflow-data}

## Updating the database {#updating-the-database}

All collected data can be used to update the database, or in deliveries. For example, you can enrich message content personalization possibilities (include the number of contracts in the message, specify the average shopping cart over the last year, etc.) or detail population targeting (send a message to contract co-holders, target the 1,000 best subscribers to online services, etc.). This data can also be exported or archived in a list.

### Lists and direct updates {#lists-and-direct-updates}

The data of the Adobe Campaign database and the existing lists can be updated using two dedicated activities:

* The **[!UICONTROL List update]** activity lets you store worktables in a datalist.

  You can select an existing list or create it. In this case, the name and possibly the record folder are computed.

  ![](assets/s_user_create_list.png)

  Refer to [List update](../../workflow/using/list-update.md).

* The **[!UICONTROL Update data]** activity performs a mass update of the fields in the database.

  For more on this, refer to [Update data](../../workflow/using/update-data.md).

### Subscription/unsubscription management {#subscription-unsubscription-management}

To find out about subscribing and unsubscribing recipients to an information service via a workflow, refer to [Subscription Services](../../workflow/using/subscription-services.md).

## Sending via a workflow {#sending-via-a-workflow}

### Delivery activity {#delivery-activity}

The delivery activity is detailed in [Delivery](../../workflow/using/delivery.md).

### Enriching and targeting deliveries {#enriching-and-targeting-deliveries}

Deliveries can process data from workflows in order to customize content or within the framework of target population selection.

For example, within the framework of a direct mail delivery, you can include the additional data, taken from data manipulation carried out in the workflow, in the extraction file:

![](assets/s_advuser_add_data_postal_mail.png)

In addition to the usual personalization fields, you can add personalization fields from workflow stages to the delivery content. The additional data defined in the workflow activities can be kept and made accessible in the delivery wizard, as shown in the example below, for defining the name of the output file within the framework of direct mail delivery:

![](assets/s_advuser_using_additional_data.png)

The data contained in the workflow table is identified by its name: it is always made up of the **targetData** link. For more on this, refer to [Target data](../../workflow/using/data-life-cycle.md#target-data).

Within the framework of email delivery, personalization fields can also use data from target extension performed in the targeting workflow stages, as shown in the example below: 

![](assets/s_advuser_add_data_email.png)

If a segment code is specified in a targeting activity, it is added to a specific column of the workflow table and will be offered along with the personalization fields. To display all personalization fields, click the **[!UICONTROL Target extension > Other...]** link accessible via the personalization button.

![](assets/s_advuser_segment_code_select.png)

## Exporting data {#exporting-data}

### Zipping or encrypting a file {#zipping-or-encrypting-a-file}

Adobe Campaign lets you export zipped or encrypted files. When defining an export through a **[!UICONTROL Data extraction (file)]** activity, you can define a post-processing to zip or to encrypt the file.

To be able to do so:

1. Install a GPG key pair for your instance using the [Control Panel](https://docs.adobe.com/content/help/en/control-panel/using/instances-settings/gpg-keys-management.html#encrypting-data).

    >[!NOTE]
    >
    >Control Panel is available to all customers hosted on AWS (excepted for customers who host their marketing instances on premise).

1. If your installation of Adobe Campaign is hosted by Adobe, contact Adobe Customer Care to have the necessary utilities installed on the server.
1. If your installation of Adobe Campaign is on premise, install the utility you want to use (for example: GPG, GZIP) as well as the necessary keys (encryption key) on the application server.

You can then use commands or code in the **[!UICONTROL Script]** tab of the activity or in a **[!UICONTROL JavaScript code]** activity. An example is presented in the use case below.

**Related topics:**

* [Unzipping or decrypting a file before processing](../../workflow/using/importing-data.md#unzipping-or-decrypting-a-file-before-processing)
* [Data extraction (file) activity](../../workflow/using/extraction--file-.md).

### Use case: Encrypting and exporting data using a key installed on Control Panel {#use-case-gpg-encrypt}

In this use case, we will build a workflow in order to encrypt and export data using a key installed on Control Panel.

A tutorial video showing how to use a GPG key to encrypt data is also available in [this section](https://docs.adobe.com/content/help/en/campaign-classic-learn/tutorials/administrating/control-panel-acc/gpg-key-management/using-a-gpg-key-to-encrypt-data.html).

The steps to perform this use case are as follows:

1. Generate a GPG key pair (public/private) using a GPG utility, then install the public key onto Control Panel. Detailed steps are available in [Control Panel documentation](https://docs.adobe.com/content/help/en/control-panel/using/instances-settings/gpg-keys-management.html#encrypting-data).

1. In Campaign Classic, build a workflow to export the data and export it using the private key that has been installed via the Control Panel. To do this, we will build a workflow as follows:

    ![](assets/gpg-workflow-encrypt.png)

    * **[!UICONTROL Query]** activity: In this example, we want to execute a query to target the data from the database that we want to export.
    * **[!UICONTROL Data extraction (file)]** activity: Extracts the data into a file.
    * **[!UICONTROL JavaScript code]** activity: Encrypts the data to extract.
    * **[!UICONTROL File transfer]** activity: Sends the data to an external source (in this example, an SFTP server).

1. Configure the **[!UICONTROL Query]** activity to target the desired data from the database. For more on this, refer to [this section](../../workflow/using/query.md).

1. Open the **[!UICONTROL Data extraction (file)]** activity then configure it according to your needs. Global concepts on how to configure the activity are available in [this section](../../workflow/using/extraction--file-.md).

    ![](assets/gpg-data-extraction.png)

1. Open the **[!UICONTROL JavaScript code]** activity, then copy-paste the command below to encrypt the data to extract.

    >[!IMPORTANT]
    >
    >Make sure you replace the **fingerprint** value from the command with the fingerprint of the public key installed on the Control Panel.

    ```
    var cmd='gpg ';
    cmd += ' --trust-model always';
    cmd += ' --batch -yes';
    cmd += ' --recipient fingerprint';
    cmd += ' --encrypt --output ' + vars.filename + '.gpg ' + vars.filename;
    execCommand(cmd,true);
    vars.filename=vars.filename + '.gpg'
    ```
  
      ![](assets/gpg-script.png)

1. Open the **[!UICONTROL File transfer]** activity, then specify the SFTP server to which you want to send the file. Global concepts on how to configure the activity are available in [this section](../../workflow/using/file-transfer.md).

    ![](assets/gpg-file-transfer.png)

1. You can now run the workflow. Once it is executed, data target by the query will be exported to the SFTP server into an encrypted .gpg file.

    ![](assets/gpg-sftp-encrypt.png)
