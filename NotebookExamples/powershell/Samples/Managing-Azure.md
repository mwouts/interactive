---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.3.3+dev
  kernelspec:
    display_name: .NET (PowerShell)
    language: PowerShell
    name: .net-powershell
---

# Working with Azure PowerShell <img src="https://raw.githubusercontent.com/PowerShell/PowerShell/master/assets/Powershell_black_64.png" align="right"/>

## Prerequisites

You'll need to install a few Az modules to use this Notebook.

```powershell
Install-Module Az.Compute,Az.Resources,Az.KeyVault -Force
```

First connect to your Azure account.

```powershell
Connect-AzAccount
```

If your account contains more than one active subscription the first one will be selected for further use. To select another subscription, use Set-AzContext.

```powershell
$myAzSubscription = "My Subscription"
Set-AzContext -Subscription $myAzSubscription
```

### Setup common variables

These variables are used throughout the notebook so set them at the top so they can be used everywhere.

> NOTE: This also means that all you have to do is change the vaules here and the Notebook will just work still (so long as the values are correct)

```powershell
# Used all over
$RESOURCE_GROUP_NAME = 'JupyterTest'
$LOCATION = 'East US 2'

# Resource names
$VAULT_NAME = 'myAzVault'
$VM_NAME = 'myAzVM'

# Single instances
$VM_USERNAME = 'azureuser'
$VM_IMAGE = 'UbuntuLTS'
```

## Create a resource group with the `New-AzResourceGroup` command

An Azure resource group is a logical container into which Azure resources are deployed and managed. A resource group must be created before a virtual machine. In the following example, a resource group named myResourceGroupVM is created in the EastUS region:

```powershell
New-AzResourceGroup -ResourceGroupName $RESOURCE_GROUP_NAME -Location $LOCATION
```

## Create a KeyVault in Azure

We will use this to store the password to our VM for future use.

```powershell
New-AzKeyVault -Name $VAULT_NAME -Location $LOCATION -ResourceGroupName $RESOURCE_GROUP_NAME
```

## Creating a new Azure VM

### Generate a secret that will be used for the password

> Note: You should switch this to key-based authentication or something else in the future,
> but this is fine for the purposes of this demo.

```powershell
$secret = [System.IO.Path]::GetRandomFileName() | ConvertTo-SecureString -AsPlainText
```

### Store the secret in KeyVault for future usage

```powershell
Set-AzKeyVaultSecret -VaultName $VAULT_NAME -SecretValue $secret -Name VMpassword
```

### Generate a credential object for the `New-AzVM` command

```powershell
$cred = [pscredential]::new($VM_USERNAME, $secret)
```

### Create our VM

```powershell
$splat = @{
    Image = $VM_IMAGE
    Credential = $cred
    ResourceGroupName = $RESOURCE_GROUP_NAME
    Location = $LOCATION
    Name = $VM_NAME
}

New-AzVM @splat
```

At this point you should be able to run `Get-AzVM` and your VM should show up.

```powershell
Get-AzVM -Name $VM_NAME -Status
```
