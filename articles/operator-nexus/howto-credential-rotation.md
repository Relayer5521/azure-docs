---
title: Azure Operator Nexus credential rotation
description: Describes the credential rotation lifecycle including automated rotation & requests for a manual rotation.
ms.service: azure-operator-nexus
ms.custom: template-how-to
ms.topic: how-to
ms.date: 12/02/2024
author: matternst7258
ms.author: matthewernst
---

# Credential rotation management for Operator Nexus on-premises devices

This article describes the Operator Nexus credential rotation lifecycle including automated rotation & requests for manual rotation.

## Prerequisites

- Target cluster and fabric must be in running and healthy state.
- Platform credential updates are written to a user provided key vault, if provided. Users provide key vault information on the Cluster resource during Cluster create or update.
  - For more information on adding key vault information to the Cluster, see [Create and provision a Cluster](howto-configure-cluster.md).
  - The Cluster update command allows users to add or change key vault information.
  - For information on configuring the key vault to receive credential rotation updates, see [Setting up Key Vault for Managed Credential Rotation](how-to-credential-manager-key-vault.md).

> [!IMPORTANT]
> A key vault must be provided on the Cluster, otherwise credentials aren't retrievable. Microsoft Support doesn't have access to the credentials.

## Rotating credentials

The Operator Nexus Platform offers a managed credential rotation process that automatically rotates the following credentials:

- Baseboard Management Controller (BMC)
- Pure Storage Array Administrator
- Console User for emergency access
- Console User SSH keys for emergency access
- Local path storage

When a new Cluster is created, the credentials are automatically rotated during deployment. The managed credential process then automatically rotates these credentials periodically based on the credential type. The updated credentials are written to the key vault associated with the Cluster resource.

> [!NOTE]
> The introduction of this capability enables auto rotation for existing instances. Rotation occurs during the management upgrade if any of the supported credentials are due for rotation within the expected rotation time period.

The 2024-07-01-GA version of the Network Cloud API added the credential rotation status on the Bare Metal Machine and Storage Appliance resources. This information can be found in the secretRotationStatus data construct for each of the rotated credentials. The 2025-07-01-preview & subsequent versions of the API adds the keyVaultUri to this data construct to indicate which Key Vault contains the rotated secret.

One example of this `secretRotationStatus` looks like:
```
       "secretRotationStatus": [
            {
                "lastRotationTime": "2024-10-30T17:48:23Z",
                "rotationPeriodDays": 14,
                "secretArchiveReference": {
                    "keyVaultUri": "<KV URI>:,
                    "keyVaultId": "<KV Resource ID>",
                    "secretName": "YYYYYYYYYYYYYYYYYYYYYY-storage-appliance-credential-manager-ZZZZZZZ",
                    "secretVersion": "XXXXXXXXXXXXXX"
                },
                "secretType": "Bare Metal Machine Identity - console"
            },
```

In the `secretRotationStatus` object, the following fields provide context to the state of the last rotation:

- `lastRotationTime`: The timestamp in UTC of the previous successful rotation.
- `rotationPeriodDays`: The number in days the Credential Manager service is scheduled to rotate this credential. This value isn't remaining days from the `lastRotatedTime` since rotation can be delayed, but how many days on a schedule the service rotates a particular credential.
- `secretArchiveReference`: A reference to the Key Vault that the credential is stored. It contains:
    - the URI of the key vault
    - the ID of the key vault
    - the secret name of the stored credential
    - the version of the secret that was previously rotated

>[!CAUTION]
> If a credential is changed on a device outside of the automatic credential rotation service, the next rotation will likely fail due to the secret not being known by the software. This issue prevents further automated rotation.

Operator Nexus also provides a service for preemptive rotation of the above Platform credentials. This service is available to customers upon request through a support ticket. Credential rotation for Operator Nexus Fabric devices also requires a support ticket. Instructions for generating a support request are described in the next section.

## Manual changes to credentials

The Credential Manager generates a secure password from the current value updates all BMC nodes and the KeyVault associated with the cluster. The Credential Manager checks KeyVault accessibility and uses the last known rotated secret to access the BMC and then performs the rotation.

The Platform doesn't recognize manually rotated secrets, preventing the Credential Manager from accessing the BMC to update the new password. For iDRAC rotation, the Credential Manager passes a new credential to the BareMetalMachine controller and the attempts to access the iDRAC password for rotation. 

The unknown state of credentials to the platform impacts monitoring and the ability to perform future runtime version upgrades.

In order to restore the state of the credential, it must be reset to a value that the platform recognizes. There are two options for this situation:

1. Run a [BareMetalMachine replace](./howto-baremetal-functions.md) action providing the current active credentials. The replace action allows the machine to use provided credentials to reset credential rotation. This action is the recommended option if significant changes are made to the machine.
1. Reset the BMC credential back to the value before the manual change. If a key vault is configured for receiving rotated credential, then the proper value can be obtained using information from the `secretRotationStatus` data for the Bare Metal Machine resource. The rotation status for the BMC Credential indicates the secret key and version within the key vault for the appropriate value. Once the credential is set to match the value expected by the credential rotation system, rotation occurs normally.

Example `secretRotationStatus` for BMC credential. Use the `secretName` and `secretVersion` to find the proper value in the cluster key vault.
```
{
  {
    ...
    "secretArchiveReference": {
      "secretName": "YYYYYYYYYYYYYYYYYYYYYY-idrac-credential-manager-ZZZZZZZ",
      "secretVersion": "XXXXXXXXXXXXXX"
    },
    "secretType": "BMC Credential"
  }
},
```

## Create a support request

Users raise credential rotation requests by [contacting support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade). These details are required in order to perform the credential rotation on the requested target instance:

- Type of credential that needs to be rotated.
- Provide Tenant ID.
- Provide Subscription ID.
- Provide Resource Group Name in which the target cluster or fabric resides based on type of credential that needs to be rotated.
- Provide Target Cluster or Fabric Name based on type of credential that needs to be rotated.
- Provide Target Cluster or Fabric Azure Resource Manager (ARM) ID based on type of credential that needs to be rotated.
- Provide the Customer Key Vault ID where rotated credentials are written.

For more information about Support plans, see [Azure Support plans](https://azure.microsoft.com/support/plans/response/).
