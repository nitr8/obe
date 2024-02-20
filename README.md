# Windows OBE (Berner Oberland Edition)

Windows OBE / is a stripped down, hardened windows 10/11 x86/x64/arm64 image think of it as a pure bare metal OS.

It’s not an image as per say, more a process of taking an image provided by Microsoft and stripping it in a supported documented manner. No tools, no tricks just pure RTFM stuff.

| Origianl | OBE |
| ------------- | ------------- |
| ![](https://raw.githubusercontent.com/nitr8/obe/main/images/win-11-orig.png) | ![](https://raw.githubusercontent.com/nitr8/obe/main/images/win-11-obe.png) |

To get started download any Official Microsoft windows iso preferably the latest with integrated service packs. For this purpose, lets download the official Windows 11 23H3 English International version from  [Microsoft Software Download (https://www.microsoft.com/software-download/windows11)] image.

From the downloaded iso `Win11_23H2_EnglishInternational_x64v2.iso` extract ` install.wim` from the `\sources` folder into `c:\win-11\` for example.

Displays information about the images that are contained in a .wim
```
Dism /Get-ImageInfo /ImageFile:C:\win-11\install.wim | Select-String Index,Name

Details for image : C:\win-11\install.wim

Index : 1
Name : Windows 11 Home

Index : 4
Name : Windows 11 Education

Index : 6
Name : Windows 11 Pro
```

Now let’s use `Windows 11 Pro` which is index `6` and export just that one image to a new WIM. Don’t worry we will change it to `Windows 11 Enterprise` later on.

```
Dism /Export-Image /SourceImageFile:C:\win-11\install.wim /SourceIndex:6 /DestinationImageFile:C:\win-11\install_boe.wim
```

Mount the WIM to a local folder, to start to modify the window image.
```
Dism /Mount-Image /ImageFile:C:\win-11\install_boe.wim /index:1 /MountDir:C:\mnt
```

Before removing or disabling any components it is recommended to first update the images with the latest service. The best way to achive this would be to deploy a test image, go online see what updates are missing, take not of the KB and use that to download from the [Microsoft Update Catalog (https://www.catalog.update.microsoft.com)]. Once the file is 
```
Dism /Image:C:\mnt /LogPath:logs\kb5034467.log /Add-Package /PackagePath:C:\win-11\kb\windows11.0-kb5034467.msu
Dism /Image:C:\mnt /LogPath:logs\kb5034765.log /Add-Package /PackagePath:C:\win-11\kb\windows11.0-kb5034765.msu
Dism /Image:C:\mnt /cleanup-image /StartComponentCleanup /ResetBase
```
