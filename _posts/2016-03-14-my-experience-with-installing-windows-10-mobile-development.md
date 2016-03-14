---
layout: post
title: My experience with installing Windows 10 Mobile development
categories: windows-10 mobile development
---

I started with an Acer Aspire E1 laptop on Windows 8 and upgraded it to Windows Professional 8 then Windows Professional 8.1. I started using Visual Studio Express 2013 to maintain the Cordova SQLite plugin for Windows Phone 8 and and then develop the Windows 8.1/Windows Phone 8.1 version based of <https://github.com/doo/SQLite3-WinRT>. There has been demand for a Windows 10 (UWP) version so I upgraded the laptop to Windows 10 and installed Visual Studio Express 2015.

I had a number of challenges running a Windows 10 app on a mobile emulator or device. I had troubles and decided to put this part on hold. I went to a local electronics store, obtained a Microsoft Lumio 550 mobile device, and got a Visual Studio boilerplat app after a few tricks.

## What I tried to get the emulator to work

As directed by Visual Studio 2015 Express, I downloaded and ran the Windows 10 emulator from: <https://dev.windows.com/en-us/downloads/sdk-archive>

Some tips from <http://stackoverflow.com/questions/21059248/windows-phone-emulator-not-working>:

- Delete all virtual machines and virtual switches from Hyper V Manager ref: <http://stackoverflow.com/a/21084481/1283667>
- Add XDE.exe to the Windows Firewall ref: <http://stackoverflow.com/a/26291731/1283667>
- Enabled a couple more Hyper-V features in **Programs and Features** (opposite of: <http://stackoverflow.com/a/32137344/1283667>)

I am logging these in case they may help someone else or may have had an unknown effect on getting my app to run on the Windows 10 mobile device.

## Extra: Windows simulator

I was able to run the sample app in the Windows simulator. It looked like my Windows desktop.

## How I got it to work on my Windows 10 mobile device

I enabled developer mode in both my Windows development system (Acer laptop) and my Windows Mobile device, as documented in: <https://msdn.microsoft.com/en-us/windows/uwp/get-started/enable-your-device-for-development>

But I still got a Windows 10 developer license error 800704c7 and discovered the solution at: <https://disqus.com/home/discussion/makezine/tips_and_tricks_using_windows_10_iot_core_for_raspberry_pinbsp2/#comment-2031923826>

In summary:

- Run gpedit.msc from an Administrator command prompt
- In Local Group Policy Editor: follow Local Computer Policy => Computer Configuration => Administrative Templates => Windows Components => App Package Deployment
- Set "Allows development of Windows Store apps and installing them from an integrated development environment (IDE)" and "Allow all trusted apps to install" to Enabled
