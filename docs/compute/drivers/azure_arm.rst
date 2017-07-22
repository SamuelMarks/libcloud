Azure ARM Compute Driver Documentation
======================================

Azure driver allows you to integrate with Microsoft `Azure Virtual Machines`_
provider using the `Azure Resource Management`_ (ARM) API.

.. figure:: /_static/images/provider_logos/azure.jpg
    :align: center
    :width: 300
    :target: http://azure.microsoft.com/en-us/services/virtual-machines/

Azure Virtual Machine service allows you to launch Windows and Linux virtual
servers in many datacenters across the world.

Connecting to Azure
-------------------

To connect to Azure you need your tenant ID and subscription ID.  Using the
[Azure CLI 2.0](https://github.com/Azure/azure-cli), use ``az account list`` to get these
values.

Creating a Service Principal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following directions are based on
https://azure.microsoft.com/en-us/documentation/articles/resource-group-authenticate-service-principal/

.. sourcecode:: bash

  azure ad app create --display-name "<Your Application Display Name>" --homepage "<https://YourApplicationHomePage>" --identifier-uris "<https://YouApplicationUri>" --password <Your_Password>
  azure ad sp create --id "<Application_Id>"
  azure role assignment create --assignee "<Object_Id>" --role Owner --scope /subscriptions/{subscriptionId}/


.. sourcecode:: bash

  $ az ad app create --display-name t0 --identifier-uris "<https://YourApplicationHomePage>" --password sdfsdfs --homepage "<https://YourApplicationHomePage>"
  {
    "appId": "azAdAppCreateAppId",
    "appPermissions": null,
    "availableToOtherTenants": false,
    "displayName": "t0",
    "homepage": "<https://YourApplicationHomePage>",
    "identifierUris": [
      "<https://YourApplicationHomePage>"
    ],
    "objectId": "azAdAppCreateObjectId",
    "objectType": "Application",
    "replyUrls": []
  }
  $ az ad sp create --id azAdAppCreateAppId
  {
    "appId": "azAdAppCreateAppId",
    "displayName": "t0",
    "objectId": "azAdSpCreateObjectId",
    "objectType": "ServicePrincipal",
    "servicePrincipalNames": [
      "azAdAppCreateAppId",
      "<https://YourApplicationHomePage>"
    ]
  }
  $ az role definition list | python -c 'import json,sys;obj=json.load(sys.stdin); print next(o["id"] for o in obj if o["properties"]["description"] == "Lets you manage everything, including access to resources.");'
  /subscriptions/azRoleDefListSubscriptionChosen/providers/Microsoft.Authorization/roleDefinitions/foundSubscriptionId
  $ # choose location from: `az account list-locations --query [].name`
  $ az group create -n Spon0 -l australiasoutheast
  {
    "id": "/subscriptions/azRoleDefListSubscriptionChosen/resourceGroups/Spon0",
    "location": "australiasoutheast",
    "managedBy": null,
    "name": "Spon0",
    "properties": {
      "provisioningState": "Succeeded"
    },
    "tags": null
  }
  $ # Create role with relevant subscription from `az account list`:
  $ az role definition create --role-definition '{"AssignableScopes": ["/subscriptions/chosenSubscriptionId"], "Name": "All hands", "Actions": ["Microsoft.Compute/*/read", "Microsoft.Compute/virtualMachines/start/action", "Microsoft.Compute/virtualMachines/restart/action", "Microsoft.Network/*/read", "Microsoft.Storage/*/read", "Microsoft.Authorization/*/read", "Microsoft.Resources/subscriptions/resourceGroups/read", "Microsoft.Resources/subscriptions/resourceGroups/resources/read", "Microsoft.Insights/alertRules/*", "Microsoft.Support/*"], "Description": "Can monitor compute, network and storage, and restart virtual machines"}'
  {
    "id": "/subscriptions/chosenSubscriptionId/providers/Microsoft.Authorization/roleDefinitions/roleDefName",
    "name": "roleDefName",
    "properties": {
      "assignableScopes": [
        "/subscriptions/chosenSubscriptionId"
      ],
      "description": "Can monitor compute, network and storage, and restart virtual machines",
      "permissions": [
        {
          "actions": [
            "Microsoft.Compute/*/read",
            "Microsoft.Compute/virtualMachines/start/action",
            "Microsoft.Compute/virtualMachines/restart/action",
            "Microsoft.Network/*/read",
            "Microsoft.Storage/*/read",
            "Microsoft.Authorization/*/read",
            "Microsoft.Resources/subscriptions/resourceGroups/read",
            "Microsoft.Resources/subscriptions/resourceGroups/resources/read",
            "Microsoft.Insights/alertRules/*",
            "Microsoft.Support/*"
          ],
          "notActions": []
        }
      ],
      "roleName": "All hands",
      "type": "CustomRole"
    },
    "type": "Microsoft.Authorization/roleDefinitions"
  }

Instantiating a driver
~~~~~~~~~~~~~~~~~~~~~~

Use <Application_Id> for "key" and the <Your_Password> for "secret".

Once you have the tenant id, subscription id, application id ("key"), and
password ("secret"), you can create an AzureNodeDriver:

.. literalinclude:: /examples/compute/azure_arm/instantiate.py
   :language: python

Alternate Cloud Environments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can select an alternate cloud environment using the "cloud_environment"
parameter to AzureNodeDriver constructor.  Available alternate cloud
environments are 'AzureChinaCloud', 'AzureUSGovernment' and 'AzureGermanCloud'.
You can also supply explicit endpoints by providing a dict with the keys
'resourceManagerEndpointUrl', 'activeDirectoryEndpointUrl',
'activeDirectoryResourceId' and 'storageEndpointSuffix'.

API Docs
--------

.. autoclass:: libcloud.compute.drivers.azure_arm.AzureNodeDriver
    :members:
    :inherited-members:

.. _`Azure Virtual Machines`: http://azure.microsoft.com/en-us/services/virtual-machines/

.. _`Azure Resource Management`: https://msdn.microsoft.com/en-us/library/azure/Dn948464.aspx
