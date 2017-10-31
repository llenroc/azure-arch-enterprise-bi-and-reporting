# Manage the Deployed Infrastructure

## Summary
Once the deployment is successfully completed, you will find information about how to manage the deployment and change some key properties of your deployment.

## Table of Contents
[Change key properties of the deployment](#change-key-properties-of-the-deployment)

[Change the password to the datawarehouse user account](#change-the-password-to-the-datawarehouse-user-account)

[Add more virtual machines to your SSRS scale-out set](#add-more-virtual-machines-to-your-ssrs-scale-out-set)

[Add more virtual machines to your SSAS Read-Only scale-out set](#add-more-virtual-machines-to-your-ssas-read-only-scale-out-set)


### Change key properties of the deployment

The following data-driven values are tracked in the dbo.ControlServerProperties table in the Control Server database (search for ControlServerDB in resources under your resource group) and can be edited to change the desired setting:
1. `ComputeUnits_Active` and `ComputeUnits_Load`:
These two properties track the datawarehouse compute units for the datawarehouse that is active or load state respectively. Please exercise caution and pick an [allowable value](https://azure.microsoft.com/en-us/pricing/details/sql-data-warehouse/elasticity/) while also considering the pricing for the desired values. If the value is not allowed, it will be ignored in favor of a safe default which will not help you achieve the desired scale.

2. `FlipInterval`: This number stands for the number of hours after which we will attempt to flip the state of the logical datawarehouses (along with the physical datawarehouses under it) from `Active` to `Load`. The exact flip event times are visible in the Admin UI as well as the dbo.DWStateHistories table in the Control Server database.

### Change the password to the datawarehouse user account
This section is TBD.

### Add more virtual machines to your SSRS scale-out set
You can add new virtual machines of type SSRS or SSAS-Read Only if you want to want to scale-out your deployment to allow for more traffic.

1.  Add SSRS virtual machines:
    1. Find your deployment resource group in the [Azure portal](https://portal.azure.com).
    2. Add new VMs as needed along with network interfaces and join the VM to your Azure AD domain and the SSRS availability set. You can optionally search for the deployment (called 'deployVMSSRS') from the deployments in your resource group  and redeploy the additional VMs by simply increasing the number of instances.

2. Register DSC on the new virtual machines:
    1. Find your resource by searching for 'automationAccount' in your resource group.
    2. Under `Configuration Management`, click on `DSC Nodes` and add your new Azure VM and connect it to the 'DSCConfiguration.ssrs' node configuration. Choose 'ApplyAndMonitor' for the `Configuration Mode` and 'ContinueConfiguration' for the `Action after Reboot` while adding the node. Once the node is added, you will be able to find it listed under `DSC Nodes`.

3. Add the virtual machine to the SSRS scale-out set:
This step will require you to download a custom script extension (ConfigureSSRS-CSE.ps1) which can be found under the 'ssrs' container under the artifacts storage account (search for 'edwartifacts' under your resource group) and use the default parameter set ('NonPrimarySSRS') since you are adding the VM to an existing scale-out set.

### Add more virtual machines to your SSAS Read-Only scale-out set
The first two steps are very similar to the steps for adding SSRS virtual machines as noted above. The final step requires that you populate the dbo.TabularModelNodeAssignments table in the Control Server database (search for 'ControlServerDB' in the resources under your resource group). The data values depend on the Analysis Services tables that you intend to populate on the SSAS Read-Only virtual machine. You can examine the existing entries corresponding to existing nodes in this table for suitable examples. In order to catch with the other Read-Only nodes, we recommend restoring backups of the databases from those nodes onto your newly added node.
