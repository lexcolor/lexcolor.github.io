---
title: Using BcContainers 
date: 2022-08-29 12:00:00 -500
categories: [BcContainers] 
tags: [containers, docker]
---

All of my time as a NAV/BC developer, I have been using virtual machines to deal with different versions in different projects. The downside of this approach is the disk space. Every VM uses around 20GB so, with a 512 SSD, I had to move the VMs to an external disk and keep in the SSD only those I would use more frequently.

I did try to use BcContainers back when they started being a thing with AL, but in those early days, you had to download an image for the specific version, so, there was not really a gain there. It was the same thing with new complications.
Then they started using artifacts, so I tried again, can’t remember the exact reasons why I did not succeed, but I discarded the idea and kept using VM’s.

I’m currently working in two projects for which I recycled an old PC, so I formatted the SSD, installed Windows from scratch and before installing any BC, I decided to try the docker approach.

For one of the projects I’m using BC 19.2, but with a twist, since it is a LS Retail project, so I have to replace the BC database.

For the second project, I’m using BC 20.4

I was able to install both versions of BC and deploy my extensions, but I wasn’t able at that time to make the 19.2 to work with the LS Retail database, since it implied changing the database, and copying some dependencies in the Add-ins folder.

Then August started and I kept moving with the 20.4 project, but This week, I needed to advance with the 19.2 version, so I gave it another try.

With this script I was able to configure BC to use the database in my local SQL Server instead of the container one.

```powershell
$containerName = 'ls192'
$credential = Get-Credential -Message 'Using UserPassword authentication. Please enter credentials for the container.'
$auth = 'UserPassword'
$artifactUrl = Get-BcArtifactUrl -type 'OnPrem' -version '19.2' -country 'w1' -select 'Latest'
$databaseServer = 'MYPCNAME'
$databaseInstance = ''
$databaseName = 'LS192'
$databaseUsername = 'sa'
$databasePassword = 'mysupersecurepassword'
$databaseSecurePassword = ConvertTo-SecureString -String $databasePassword -AsPlainText -Force
$databaseCredential = New-Object pscredential $databaseUsername, $databaseSecurePassword
$licenseFile = 'c:\tmp\bc19.flf'

New-BcContainer `
  -accept_eula `
  -containerName $containerName `
  -credential $credential `
  -auth $auth `
  -artifactUrl $artifactUrl `
  -databaseServer $databaseServer -databaseInstance $databaseInstance -databaseName $databaseName `
  -databaseCredential $databaseCredential `
  -licenseFile $licenseFile `
  -updateHosts 
```

This script uses an existing database in my host SQL Server, the database I restored previously and passed my SQL Credentials to the script. This is the point where I had an error back at the beggining of this month and stopped trying. The error was that when I tried this, I had the firewall in the host enabled. I tried a lot of things to get past the error, including changing MYPCNAME for localhost. Then disabled the firewall, configured named pipes for SQL Server and kept getting the same error. Just to be sure I tried everything, I reverted the localhost to MYPCNAME and it worked fine this time.

Then I had to create a user in the database, so I used this script:

```powershell
$password = "anothersupersecurepassword"
$securePassword = ConvertTo-SecureString -String $password -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -argumentList "myusername", $securePassword
New-BcContainerBcUser -Credential $credential -containerName ls192 -PermissionSetId SUPER -Verbose  -ChangePasswordAtNextLogOn 0 
```

After that, there were those dependencies I mentioned before. Being no expert in Docker, I can’t explain this behavior, but i copied the LSRetail folder from the host to the container, and then entered the container and tried to copy the folder to the add-ins folder, but that didn’t worked. It copied the folder, but not the contents…
So, i decided to push my luck and modify the docker cp command like this:

```powershell
docker cp C:\tmp\LSRetail "ls192:/program files/microsoft dynamics nav/190/service/add-ins" 
```

And, it worked just fine.

Now I have the development environment for both of my projects, and wrote this post so i can come back if I need this in the future.