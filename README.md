# Windows BOE (Berner Oberland Edition)

Windows BOE / is a stripped down, hardened windows 10/11 x86/x64/arm64 image think of it as a pure bare metal OS.

It’s not an image as per say, more a process of taking an image provided by Microsoft and stripping it in a supported documented manner. No tools, no tricks just pure RTFM stuff.

| Origianl | OBE |
| ------------- | ------------- |
| ![](https://raw.githubusercontent.com/nitr8/obe/main/images/win-11-orig.png) | ![](https://raw.githubusercontent.com/nitr8/obe/main/images/win-11-obe.png) |

To get started download any Official Microsoft windows iso preferably the latest with integrated service packs. [Microsoft Software Download (https://www.microsoft.com/software-download/windows11)]

For this purpose, We will use the official (Windows 11 IoT Enterprise, version 24H2) English International version. (Windows 11 IoT Enterprise LTSC 2024](https://learn.microsoft.com/en-us/windows/iot/iot-enterprise/whats-new/windows-11-iot-enterprise-ltsc-2024))

From the downloaded iso `Win11_23H2_EnglishInternational_x64v2.iso` extract ` install.wim` from the `\sources` folder into `c:\win-11\` for example.

Displays information about the images that are contained in a .wim
```
Dism /Get-ImageInfo /ImageFile:C:\win-11\install.wim | Select-String Index,Name
```

Now let’s use `Windows 11 Pro` which is index `6` and export just that one image to a new WIM. Don’t worry we will change it to `Windows 11 Enterprise` later on.

```
Dism /Export-Image /SourceImageFile:C:\win-11\install.wim /SourceIndex:6 /DestinationImageFile:C:\win-11\install_boe.wim
```

Mount the WIM to a local folder, to start to modify the window image.
```
Dism /Mount-Image /ImageFile:C:\win-11\install_boe.wim /index:1 /MountDir:C:\mnt
```

Before removing or disabling any components it is recommended to first update the images with the latest service. The best way to achive this would be to deploy a test image, go online see what updates are missing, take not of the KB and use that to download from the [Microsoft Update Catalog (https://www.catalog.update.microsoft.com). Once the file has been downloaded create a new folder `kb` under `c:\win-11\`. Move and rename the MSU files to something easier and slipstream them to the image. 

This will add two KB’s and then reset the image base.
```
Dism /Image:C:\mnt /Add-Package /PackagePath:C:\win-11\kb\windows11.0-kb5034467.msu
Dism /Image:C:\mnt /Add-Package /PackagePath:C:\win-11\kb\windows11.0-kb5034765.msu
Dism /Image:C:\mnt /cleanup-image /StartComponentCleanup /ResetBase
```

Get a list of install packages and remove what’s not needed. See table below for list of Packages/AppxPackages/Features not safe to remove/disable.
```
DISM /Image:C:\mnt /Get-Packages | Select-String Identity
DISM /Image:C:\mnt /Remove-Package /PackageName:Microsoft-OneCore-ApplicationModel-Sync-Desktop-FOD-Package~31bf3856ad364e35~amd64~~10.0.22621.3155
```

```
DISM /Image:C:\mnt /Get-ProvisionedAppxPackages | Select-String Packagename
DISM /Image:C:\mnt /Remove-ProvisionedAppxPackage /PackageName:Clipchamp.Clipchamp_2.2.8.0_neutral_~_yxz26nhyzhsrt
```

```
Dism /Image:C:\mnt /Get-Features /Format:Table | find "| Enabled"
DISM /Image:C:\mnt /Disable-Feature /FeatureName:Windows-Defender-Default-Definitions /Remove
```

Unmount WIM
```
Dism /Unmount-Image /MountDir:C:\mnt /Commit
```
