---
published: true
layout: post
section-type: post
category: Blog
tags:
  - technical
---
There have been reports from users, who updated to the latest Windows 10 Anniversary Update, that some apps crash upon starting them and the newly released Edge extensions cannot be installed.

I faced the same problem. They weird thing was that only apps that were installed after the update crashed, apps that were already installed before worked fine.
I tried all the suggestions I found online, from running `wsreset.exe` in an admin cmd to running the following command in Powershell (again, with admin privileges):

        Get-AppXPackage | Foreach {Add-AppxPackage -DisableDevelopmentMode -Register "$($_.InstallLocation)\AppXManifest.xml"}

But none of that worked. After that I created a new user acoount on my machine and noticed that everything worked on that account so i thought it must be something with my main user's registry.

### My solution

Before attempting this fix uninstall all the problematic apps and Edge Extensions in `Settings -> System -> Apps and features`

1. Press `Win + R`
2. Type `regedit` and press `Enter`
3. (Optional) If you're afraid of messing up your registry create a backup by going to `File -> Export...`, it might take a while.
4. In the registry editor go to `HKEY_CURRENT_USER\SOFTWARE\Classes\Local Settings\Software\Microsoft\Windows\CurrentVersion\AppContainer` and delete the `Storage` folder.
![Regedit](http://i.imgur.com/kQKoA4V.png)

5. Open Powershell with admin privileges and run the following commands, one at a time:

          Get-appxpackage -packageType bundle |% {add-appxpackage -register -disabledevelopmentmode -ForceApplicationShutdown ($_.installlocation + “\appxmetadata\appxbundlemanifest.xml”)}
          
          $bundlefamilies = (get-appxpackage -packagetype Bundle).packagefamilyname
          
          get-appxpackage -packagetype main |? {-not ($bundlefamilies -contains $_.packagefamilyname)} |% {add-appxpackage -register -disabledevelopmentmode -ForceApplicationShutdown ($_.installlocation + “\appxmanifest.xml”)}

6. Reboot

I hope this works for you.
