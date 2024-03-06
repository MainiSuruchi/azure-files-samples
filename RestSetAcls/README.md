# RestSetAcls.psm1

## Description

RestSetAcls.psm1 is a PowerShell module that provides functions to set Access Control Lists (ACLs) for Azure file shares, using the Azure Files REST API.

> [!NOTE]  
> RestSetAcls.psm1 currently only supports setting the same owner, group and permissions on all files within a share or subdirectory.
> It does not yet support:
>
> - Updates to one field without updating others (e.g., updating the owner without updating the group and permissions)
> - Adding or removing a permission, without otherwise changing the permissions

## Prerequisites

- PowerShell 5.1 or later ([installation instructions](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell))
- `Az.Storage` module v4.1.1 or later ([installation instructions](https://learn.microsoft.com/en-us/powershell/azure/install-azure-powershell))
- Azure Storage account with a file share
- Azure Storage account key ([instructions on how to find the key](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-keys-manage?tabs=azure-portal#view-account-access-keys))

## Installation

1. Download the `RestSetAcls.psm1` file from this repo.
1. Save the file to a local directory.

## Usage

1. Make sure you meet the prerequisites above.
1. Open a PowerShell session.
1. Define how to connect your storage account with storage account key:

   ```powershell
   $AccountName = "<storage-account-name>" # replace with the storage account name
   $AccountKey = "<storage-account-key>" # replace with the storage account key

   $context = New-AzStorageContext -StorageAccountName $AccountName -StorageAccountKey $AccountKey
   ```

   If your storage account is on another Azure environment than the public Azure cloud, you can use the `-Environment` or `-FileEndpoint` flags of `New-AzStorageContext` to configure the address at which the storage account is reachable. See the documentation of [New-AzStorageContext](https://learn.microsoft.com/en-us/powershell/module/az.storage/new-azstoragecontext).
   
1. Determine the SDDL string for the desired permissions.

   If you already know what SDDL string you want, you can define it directly:

   ```powershell
   $sddl = "<sddl-string>" # replace with the SDDL string
   ```

   If you do not know the SDDL string for the permission you want to set, the easiest approach is to get it from another file. In other words, the idea is to use the permissions of another file as a template of what permissions should be set recursively on your file share. To do so, we either need a file that has the right permissions already, or we need to create it. To create it, you can create a file (anywhere you want, for instance on your Desktop), right click on it, click on Properties, go to the Security tab, click Edit, and then add, remove or edit permissions until you get the permissions you want. Press OK to save and OK to close the Properties window. You can then get the SDDL of your sample file as follows:

   ```powershell
   $filepath = "<path-to-file-or-folder>" # replace with the path to your file or folder
   $sddl = (Get-Acl -Path $filepath).Sddl
   ```
   
1. Call `Set-AzureFilesAclRecursive` as follows. This will recursively find all files and folders on your file share, and set the SDDL permission on each one of them.

   ```powershell
   $FileShareName = "<file-share-name>" # replace with the name of your file share
   
   Import-Module -Name "Path\To\RestSetAcls.psm1" # replace with the path to the downloaded RestSetAcls.psm1 file
   Set-AzureFilesAclRecursive -Context $context -FileShareName $FileShareName -FilePath "/" -SddlPermission $sddl
   ```

## Contributing

> [!WARNING]
> These instructions are only meant for contributors to this project.
> If you want to use the script, refer to the Usage instructions above.

1. Mount the file share to a local drive

   ```cmd
   net use X: \\<storage-account-name>.file.core.windows.net\<file-share-name> /u:<storage-account-name> <storage-account-key>
   ```

2. Create test files

    ```powershell
    .\RestSetAcls\TestSetup.ps1
    ```

3. Try to set permissions via icacls

    ```cmd
    icacls X:\ /t /grant "Everyone:(OI)(CI)F"
    ```

4. Compare the speed of this operation to the `Set-AzureFilesAclRecursive` function

    ```powershell
    Set-AzureFilesAclRecursive -Context $context -FileShareName $FileShareName -FilePath "/" -SddlPermission $sddl
    ```