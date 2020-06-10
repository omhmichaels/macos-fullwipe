# Wipe Macos

## Purpose:
* Fully wipe macos of mdm/rootkit shiz


### Usage 



### Resources 
*[ORIGINAL POST](https://onemoreadmin.wordpress.com/category/mac-administration/)

```
Skip to content
One More Admin

CATEGORY: MAC ADMINISTRATION
Goodbye WordPress – Hello blog.eriknicolasgomez.com
PUBLISHED ON December 28, 2016Leave a comment
As of today, I am officially closing this blog and moving entirely to Jekyll.

You can now find my new Jekyll site at http://blog.eriknicolasgomez.com

I hope you enjoy the new site (and the MUCH better code snippets).

CATEGORIES MAC ADMINISTRATION
System Integrity Protection (SIP) changes in macOS Sierra 10.12.2
PUBLISHED ON December 13, 20163 Comments
With the release of macOS Sierra 10.12.2, Apple has made one welcome change to System Integrity Protection (SIP): you can now re-enable the feature without being booted into the Recovery partition!

How To
To re-enable SIP, you run the following command:

/usr/bin/csrutil clear

Please note that you will need to run this as root. To see if the command was successful, run nvram -p and look for csr-active-config. If the key does not exist, then SIP has been re-enabled.

Example:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
csrutil status
System Integrity Protection status: disabled.
 
nvram -p
csr-active-config   w%00%00%00
 
sudo csrutil clear
Password:
Successfully cleared System Integrity Protection. Please restart the machine for the changes to take effect.
 
csrutil status
System Integrity Protection status: disabled.
 
nvram -p
Enhancement
I have asked for an enhancement to mimic the behavior of fdesetup status

Hopefully Apple can have csrutil status show something like this:

System Integrity Protection is Off, but will be enabled after the next restart.

Advertisements

REPORT THIS AD
Advertisements

REPORT THIS AD
CATEGORIES MAC ADMINISTRATION
The Untouchables Pt 2: Offline TouchBar activation with a purged disk
PUBLISHED ON November 30, 20164 Comments
A few days ago I wrote a post about a new activation mechanism for the TouchBar/WatchOS hybrid device.

After doing some more tests, I began to discover something interesting…

Preface
While I have tried to document and piece together as much as possible here, some of the statements could be inaccurate. Until Apple posts more information about this process, take everything you read below with a grain of salt. If you choose to use the methodologies in production, I offer no warranties to the integrity of your sytem.

If you just want the answer, go to Baking The Cake

ROOT_DARWIN_USER_TEMP_DIR
While the preflight container data located in /Library/Updates/PreflightContainers (Example: /Library/Updates/PreflightContainers/865FA1BB-3EF6-4F77-A4B7-01529BCE33F0.preflightContainer) changed each reboot, there was one common folder across all of my test machines:

/private/var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T

After discussing this with some colleagues (thanks yet again Michael and Pepijn) we realized that EmbeddedOSInstallService utilizes the temporary directory for the root user.

1
2
root# getconf DARWIN_USER_TEMP_DIR
/private/var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T/
More on this soon.

FDRData and ESP
In the EOSIS logging events, Apple often refers to two items: FDRData and ESP. From what I can tell, FDRData stands for FirmwareDirectoryRestoreData and ESP stands for EFI System Partition.

You can find FDRData in inside of the ESP and the root temp directory:
– /Volumes/EFI/EFI/APPLE/EMBEDDEDOS/FDRData
– /private/var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T/FDRData

PersonalizedBundle
When contacting the internet for activation, a folder is created at ROOT_DARWIN_USER_TEMP_DIR/%RANDOM_GUID%-EmbeddedOSInstall-PersonalizedBundle_.

This bundle contains Apple’s signed firmware files (img4 format), and a few plists.

A request is sent to URL (which I will not post) with a tss-request. Apple sends back a tss-response and then creates the BuildManifest.plist, EOS_RestoreOptions.plist and Restore.plist. These files have several certificates that have been signed by Apple.

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
/private/var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T/%RANDOM_GUID%-EmbeddedOSInstall-PersonalizedBundle:
BuildManifest.plist
EOS_RestoreOptions.plist
Firmware
KernelCache_kernelcache.release.img4
OSRamdisk_058-40573-247.img4
Restore.plist
RestoreKernelCache_kernelcache.release.img4
RestoreRamDisk_058-27707-457.img4
amai
 
/private/var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T/%RANDOM_GUID%-EmbeddedOSInstall-PersonalizedBundle/Firmware:
all_flash
dfu
 
/private/var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T/%RANDOM_GUID%-EmbeddedOSInstall-PersonalizedBundle/Firmware/all_flash:
all_flash.x619ap.production
 
/private/var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T/%RANDOM_GUID%-EmbeddedOSInstall-PersonalizedBundle/Firmware/all_flash/all_flash.x619ap.production:
DeviceTree_DeviceTree.x619ap.img4
LLB_LLB.x619.RELEASE.img4
RestoreDeviceTree_DeviceTree.x619ap.img4
RestoreSEP_sep-firmware.x619.RELEASE.img4
SEP_sep-firmware.x619.RELEASE.img4
iBoot_iBoot.x619.RELEASE.img4
manifest
 
/private/var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T/%RANDOM_GUID%-EmbeddedOSInstall-PersonalizedBundle/Firmware/dfu:
iBEC_iBEC.x619.RELEASE.img4
iBSS_iBSS.x619.RELEASE.img4
 
/private/var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T/%RANDOM_GUID%-EmbeddedOSInstall-PersonalizedBundle/amai:
apimg4ticket.der
debug
receipt.plist
 
/private/var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T/%RANDOM_GUID%-EmbeddedOSInstall-PersonalizedBundle/amai/debug:
tss-request.plist
tss-response.plist
combined.memboot
You can find the combined.memboot in inside of the ESP:
– /Volumes/EFI/EFI/APPLE/EMBEDDEDOS/combined.memboot.

While I have not been able to look into the subcomponents of this file, this is the boot file that is loaded into the TouchBar’s memory.

My theory is this file is combined both with the PersonalizedBundle and the local firmware currently located in /usr/standalone/firmware/iBridge1_1Customer.bundle.

The differences between online activation and offline activation/upgrades.
ONLINE ACTIVATION
Online Activation is typically required after Internet Recovery or a full disk wipe and subsequent re-image.

During online activation the following occurs through EmbeddedOSInstallService:
1. EOSIS detects that the TouchBar cannot boot and attempts to heal
2. After failing to detect the EMBEDDEDOS folder structure in the EFI partition and the corresponding FDRData, EOSIS triggers an online repair
3. EOSIS downloads a package from Apple with an identifier of com.apple.mac.EmbeddedOSInstall
– For an example of this package see this url
4. The PersonalizedBundle, FDRData and combined.memboot are created in the ROOT_DARWIN_USER_TEMP folder.
5. The ESP is mounted (equivalent to diskutil mount disk0s1)
6. The EMBEDDEDOS folder is created if it does not exist and the FDRData and combined.memboot are copied over.
7. Any EmbeddedOSInstall-FDRMemoryStore temporary folders are purged.
8. The TouchBar attempts to boot and if everything goes well the machine presents the loginwindow.
9. TouchID is now available for configuration via SetupAssistant

OFFLINE ACTIVATION
Offline Activation seems to occur each time the machine is booted.

During offline activation the following occurs through EmbeddedOSInstallService:
1. EOSIS finds the combined.memboot and FDRData from the ESP and matches the FDRData from the ROOT_DARWIN_USER_TEMP folder.
2. EOSIS attempts to boot the TouchBar with the combined.memboot
3. After a successful load, the PersonalizedPreflight container is created in /Library/Updates/PreflightContainers
4. Loginwindow is presented
5. User authenticates, unlocks the login.keychain, and TouchID is then available for use.

OFFLINE UPGRADES
Offline Upgrades seems to occur each time the machine there is a new firmware detected in the iBridge1_1Customer.bundle.

During offline activation the following occurs through EmbeddedOSInstallService:
1. EOSIS finds the combined.memboot and FDRData from the ESP and matches the FDRData from the ROOT_DARWIN_USER_TEMP folder and iBridge1_1Customer.bundle.
2. EOSIS detects a difference in version between the combined.memboot and the iBridge1_1Customer.bundle
3. The PersonalizedBundle, FDRData and combined.memboot are re-created in the ROOT_DARWIN_USER_TEMP folder.
4. The ESP is mounted (equivalent to diskutil mount disk0s1)
5. The old FDRData and combined.memboot are purged and replaced.
6. Any EmbeddedOSInstall-FDRMemoryStore temporary folders are purged.
7. The TouchBar attempts to boot and if everything goes well the machine presents the loginwindow.
8. User authenticates, unlocks the login.keychain, and TouchID is then available for use.

Baking The Cake
My spidey sense tingled when I first noticed offline activations and offline upgrades. It was clear that Apple didn’t want to force a “Critical Update required” screen every time there was a new point release and we could use this to our advantage.

With this I tried multiple re-images and finally found the correct procedure to wipe a full disk and still have offline activation during the first boot of the OS.

Here are the steps you must do:

Mount the EFI partition
Capture EFI folder from EFI Partition (ex: cp -r /Volumes/EFI/EFI /path/to/EFIbackup)
Capture contents of FDRData from preflight folder (ex: cp -r /Volumes/Macintosh\ HD/private/var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T /path/to/PersonalizedBundleBackup)
Destroy entire disk
Apply image
Copy EFI folder back to new ESP
Copy PersonalizedBundle(s) and FDRData back to the ROOT_DARWIN_USER_TEMP_DIR
Boot machine normally
If done correctly, you should be presented the normal SetupAssistant without the Critical Update required message.

CAVEATS
In order to capture the contents of the FDRData and preflight folders from a FileVault encrypted volume, you will need to unlock this disk. This may be a tall order if you still require 100% automation.
It is currently unknown what would happen if you capture a combined.memboot and FDRData on an older OS (Ex. 10.12.1) and then apply it to a newer OS image (10.12.2 once it is released).
Apple may not like us doing this and could break it at any time.
Final Thoughts
Quite a bit of time has been taken to piecemeal this together. While it has been a great academic study, I will emphasize once again that Apple needs to document this process for enterprise customers.

This will impact both imaging workflows and DEP workflows and once again, people who “remain in the past” can continue to fully automate this process if needed.

Modern workflows? Yeah about that…

CATEGORIES MAC ADMINISTRATION
Managing (or setting) the Mini TouchBar Control Strip
PUBLISHED ON November 28, 20162 Comments
While Apple documented how to customize the TouchBar, a macadmin or intrepid user may want to configure it via CLI tools.

The following is a brief overview on how to quickly set these defaults.

The Control Strip
The Control Strip is the persistant, right area of the Touch Bar. You can customize four quick use actions.

Control Strip

You can customize it through System Preferences -> Keyboard -> Customize Touch Bar

Once there, a GUI overlay will be displayed, allowing you to drag the desired customization directly to the Touch Bar.

Control Strip Customization

How to configure the Control Strip
The values
As of 10.12.1, these are the following values you can configure.
* com.apple.system.brightness
* com.apple.system.dashboard
* com.apple.system.dictation
* com.apple.system.do-not-disturb
* com.apple.system.input-menu
* com.apple.system.launchpad
* com.apple.system.media-play-pause
* com.apple.system.mission-control
* com.apple.system.mute
* com.apple.system.notification-center
* com.apple.system.screen-lock
* com.apple.system.screen-saver
* com.apple.system.screencapture
* com.apple.system.search
* com.apple.system.show-desktop
* com.apple.system.siri
* com.apple.system.sleep
* com.apple.system.volume
I think they are fairly easy to comprehend, so I will not be detailing each value.

You can customize up to four buttons here and even deploy zero button.

The preference file is located at ~/Library/Preferences/com.apple.controlstrip.plist

1
2
3
4
5
6
7
8
9
10
11
12
13
14
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>MiniCustomized</key>
    <array>
        <string>com.apple.system.do-not-disturb</string>
        <string>com.apple.system.media-play-pause</string>
        <string>com.apple.system.sleep</string>
        <string>com.apple.system.volume</string>
    </array>
    <key>last-messagetrace-stamp</key>
</dict>
</plist>
The order of the strings are important as they correspond to each of the four items. If you want to change the right icon, change the bottom string.

If you want to deploy a single item, just deploy a single string inside the array.

Option 1 – Yucky UNIX command
Use this method to quickly set the default values for your users. This could be a login-once script via outset or something similar.

Just know that by using this truly terrible method, a California macadmin loses their wings.

In the userspace:

1
defaults write ~/Library/Preferences/com.apple.controlstrip MiniCustomized '(com.apple.system.do-not-disturb, com.apple.system.media-play-pause, com.apple.system.sleep, com.apple.system.screen-lock )'
followed by:

1
killall ControlStrip
Option 2 – Apple Approved Profile
By installing a profile on your devices, you can force a configuration for all users of the TouchBar.

While this might be an “Apple approved” management process, it is clunky for the following reasons:
* The GUI does not inform the user that their TouchBar is being managed.
* The user can customize their TouchBar on top of the management, but after a reboot or logout/login, it will be re-configured per the profile.
* If you attempt to only manage one item in the Control Strip, it will still manage the entire key and the user will be limited to one single item.

If you would like a profile example, you can find one here. This example configures the ControlStrip in the exact way as the yucky UNIX command.

Note:

While your mileage may vary, you could configure the profile to use a Set-Once value versus a Forced value. This will be similar to a defaults write however it is not gauranteed to work.

For an example profile, see here.

Option 3 – Chef dynamic profiles.
This is my preferred option as you can manage the configuration of the profile, while also extending the attributes to your userbase.

A user could configure all of their machines with the following chef code:

1
2
3
4
5
6
7
# Configure MiniBar for DND, Play/Pause, Sleep, & Screen Lock
node.default['cpe_controlstrip']['MiniCustomized'] => [
  'com.apple.system.do-not-disturb',
  'com.apple.system.media-play-pause',
  'com.apple.system.sleep',
  'com.apple.system.screen-lock'
]
Please note that this requires the cpe_controlstrip cookbook, which you can find here

Example
In this example we would want to remove all current Control Strip items and replace them with just a single item – the screen lock.

1
2
3
defaults write ~/Library/Preferences/com.apple.controlstrip MiniCustomized '(com.apple.system.screen-lock)'
 
killall ControlStrip
Your users would then be left with the following:

Customized Control Strip

Final Notes
If you have not noticed, in all examples, there are four values set. This is because there are exactly four “buttons” you can customize. Make sure that whichever mechanism you deploy has all four values configured.

While some macadmins hate over-managing configurations, there is some benefits to this approach:
1. You can now configure a shortcut key to instantly lock your devices. Your security team may love this.
2. Some users may never know or care to customize their Control Strip. This at least allows you to be consistent.
3. More than likely you know what’s best for your company, not Apple.

CATEGORIES MAC ADMINISTRATION
The Untouchables – Apple’s new OS “activation” for Touch Bar MacBook Pros
PUBLISHED ON November 27, 201617 Comments
Last week Joe Chilcote discovered an interesting message when imaging a Late 2016 MacBook Pro TouchBar (from here on out referred to as TBP):

A critical software update is required for your Mac. To install this update you need to connect to a network. Select a Wi-Fi network below, or click Other Network Options to connect to the internet using other network devices.

Given that I did not see this issue when testing DEP, I set forth to to attempt to duplicate the issue and find out what triggered this event.

(If you don’t want to read this whole post or want to dupe the radars I submitted, scroll down to As a macadmin, does this impact me? for a TL;DR version.)

Critical Software Update?
I think it’s safe to say the macadmin community has been hearing rumblings about the future of macOS administration. Whether it was Michael Lynn’s excellent blog post, m(DM)acOS, APFS or even Sal Saghoian’s position being axed, many macadmins (myself included) are worried about the future of macOS administration being a MDM only world.

What if the new TBP Macs were the first piece to this future?

I was initially worried about this discovery due to it breaking a very typical thin imaging workflow:
1. asr an AutoDMG thin image
2. Install/Touch /private/var/db/.AppleSetupDone
3. Run bootstrapping software

Upon reboot, SetupAssistant was not skipped and you were instead greeted with this lovely screen:

Critical Update 1

Attempting to skip this page would lead to an additional failure:

A critical software update is required for your Mac, but an error was encountered while installing this update.

Even more worrying was the final note:
Your Mac can't be used until this update is installed. Shutdown / Try Again
Critical Update 2

If you did connect your TBP to an online source, the critical update was downloaded, installed and your system was rebooted.
Critical Update 3

This entire process takes about two minutes to finish and then you can login into the Mac. Obviously a lot of engineering effort went into this – this is not a fluke.

What triggers this? Is Mac imaging finally dead?

Trying to recreate the issue
Good news everyone: Mac imaging isn’t dead … yet.

While trying to recreate this issue, I started off with most simple workflow and methodically tried adding features.

UNEVENTFUL WORKFLOW TESTS
Workflow 1
Deploy image to current volume – no wipe
Workflow 2
Same as workflow 1
Add fingerprint to Touch ID
Workflow 3
Same as workflow 2
Convert booted volume to Core Storage
Workflow 4
Same as workflow 2
Enable FileVault 2
Workflow 5
In Imagr NetInstall environment
Open up Disk Utility
Delete FileVault 2 volume
Re-run Workflow 4
In none of these tests did I receive the critical software update. For informational purposes, workflow 5 was tested due to Imagr’s inability to wipe FileVault encrypted volumes and was the closest thing to Joe’s testing.

After some discussions with Joe, we had a theory as to the true culpit of the issue.

WORKFLOW GOLDMINE
Workflow 6
In Image NetInstall environment
Open up Disk Utility
Delete entire disk
Deploy image to newly created volume
It was with this workflow that I was finally able to recreate the issue. SetupAssistant immediately prompted the critical software update.

So what is being deleted when wiping the entire disk?

Apple’s EFI container for TouchBar
For some time, Apple has been installing EFI/firmware updates through standalone packages. Allister Banks first wrote about this during the Thunderstrike vulnerability and I have been complaining about this for some time.

Unfortunately, it looks like this how now been taken to a new level:

As a guess, we decided to look at the EFI volume

1
2
3
4
5
6
7
diskutil list
/dev/disk0 (internal):
#: TYPE NAME SIZE IDENTIFIER
0: GUID_partition_scheme 500.3 GB disk0
1: EFI EFI 314.6 MB disk0s1
2: Apple_HFS Macintosh HD 499.3 GB disk0s2
3: Apple_Boot Recovery HD 650.0 MB disk0s3
1
2
3
diskutil mount disk0s1
ls /Volumes/EFI/EFI/APPLE/EMBEDDEDOS/
FDRData combined.memboot version.plist
Embedded OS you say. And something called FDRData

After searching for Embedded OS logs (thanks to Joe Chilcote), we found some interesting tidbits:

log show --debug --predicate 'process =="EmbeddedOSInstallService"'

1
2
EmbeddedOSInstallService: Couldn't find memboot image in ESP:
file:///Volumes/EFI/EFI/APPLE/EMBEDDEDOS/combined.memboot
So clearly, Apple is looking for this “Embedded OS” and if it can’t find it, it attempts to rebuild the boot process.

EmbeddedOSInstallService
As a test, I decided to delete the contents of /Volumes/EFI/EFI, reboot and look at the logs.
It appears that the following happens:
* During macOS system start, the TouchBar attempts to boot running it’s own derivative of iOS.
* If it cannot find the embedded OS in the EFI volume, it triggers a repair.
* Taking iOS files from /usr/standalone/firmware/iBridge1_1Customer.bundle/Contents/Resources and /Library/Updates/PreflightContainers:
* If a valid preflight exists, no internet connection is required.
* macOS will show an extended/long boot process to the user, typically taking 2-3 minutes before the desktop is available.
* If a valid preflight does not exist, an internet connection is required
* SetupAssistant is triggered, informing the user of a Critical Update needed
* Once the TouchBar has either repaired itself or booted properly, biometrics/Touch ID is now available to the user.

Valid Preflight:

1
2
3
4
EmbeddedOSInstallService: network reachability check
EmbeddedOSInstallService: ---- Starting network reachability check ----
EmbeddedOSInstallService: We have a valid preflighted container, no network is required
EmbeddedOSInstallService: (EmbeddedOSInstall) prepare device
Invalid Preflight:

1
2
3
4
5
6
EmbeddedOSInstallService: No matching preflight container found
EmbeddedOSInstallService: network reachability check
EmbeddedOSInstallService: ---- Starting network reachability check ----
EmbeddedOSInstallService: Checking for reachability to gs.apple.com
EmbeddedOSInstallService: personalization
EmbeddedOSInstallService: ---- Starting personalization ----
No Internet

1
EmbeddedOSInstallService: Can't continue the restore because you are not connected to the Internet.
Here is an abridged version of the logging events:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
101
102
103
104
105
106
107
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
124
125
EmbeddedOSInstallService: (EmbeddedOSInstall) FDR preflight
EmbeddedOSInstallService: ---- Starting FDR preflight ----
EmbeddedOSInstallService: Waiting for device to be connected
EmbeddedOSInstallService: Device connected: EOSDevice , boardID: 0x12, chipID: 0x8002, secure: YES, prod fused: YES>
EmbeddedOSInstallService: Starting FDR preflight
EmbeddedOSInstallService: Wrote preflighted FDR to memory store URL: /var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T/6140FB57-C5C9-4E4D-8534-1BA6F879A77A-EmbeddedOSInstall-FDRMemoryStore
EmbeddedOSInstallService: (EmbeddedOSInstall) preflight container stash
EmbeddedOSInstallService: ---- Starting preflight container stash ----
EmbeddedOSInstallService: Saved preflight container to disk: EOSPreflightContainer <file:///Library/Updates/PreflightContainers/FE0EC8F3-E032-49C1-B6E2-EA42171165AB.preflightContainer> (preflighted on 2016-11-24 07:00:02 +0000)
EmbeddedOSInstallService: Preflight time: 3.0 seconds
EmbeddedOSInstallService: Preflight was successful!
EmbeddedOSInstallService: Diagnostic summary: Preflight (14Y363 -> 14Y363 (Customer Boot), preflighted = 0, prod fused = 1, user auth = 0, retries = 0, after boot failure = 0, failing phase = 0, uuid = A21D9039-E0B8-42DA-A3D6-A037EE04484B): success
EmbeddedOSInstallService: ---- End Embedded OS Preflight ----
EmbeddedOSInstallService: [com.apple.mac.install.EmbeddedOSInstall] Adding client: loginwindow (pid = 91, uid = 0, path = /System/Library/CoreServices/loginwindow.app/Contents/MacOS/loginwindow)
EmbeddedOSInstallService: Checking if we should heal the device
EmbeddedOSInstallService: No data found in ios-boot-in-progress NVRAM key
EmbeddedOSInstallService: Device isn't booted yet and boot isn't in progress (EFI failed to bootstrap?)
EmbeddedOSInstallService: ---- Begin Embedded OS Boot ----
EmbeddedOSInstallService: (EmbeddedOSInstall) force reset
EmbeddedOSInstallService: ---- Starting force reset ----
EmbeddedOSInstallService: Resetting device
EmbeddedOSInstallService: (EmbeddedOSSupportHost) connection with driver establish (connect: 4907, service: 4807)
EmbeddedOSInstallService: Waiting for device to be connected
EmbeddedOSInstallService: Entering recovery mode, starting command prompt
EmbeddedOSInstallService: recovery mode device matches (using device type)
EmbeddedOSInstallService: ---- Starting memboot from EFI system partition ----
EmbeddedOSInstallService: Waiting for device to be connected
EmbeddedOSInstallService: recovery mode device matches (using locationID)
EmbeddedOSInstallService: (EmbeddedOSSupportHost) registered for 'com.apple.EmbeddedOS.DeviceConnected' darwin notification
EmbeddedOSInstallService: (EmbeddedOSSupportHost) registered for 'com.apple.EmbeddedOS.DeviceUnresponsive' darwin notification
EmbeddedOSInstallService: Waiting for Storage Kit to populate disks
EmbeddedOSInstallService: Done waiting for Storage Kit to populate disks
EmbeddedOSInstallService: Mounting ESP
EmbeddedOSInstallService: Couldn't find memboot image in ESP: file:///Volumes/EFI/EFI/APPLE/EMBEDDEDOS/combined.memboot
EmbeddedOSInstallService: Unmounting ESP
EmbeddedOSInstallService: Boot failed with error: Code=1201 "No memboot image was found in the EFI system partition."
EmbeddedOSInstallService: ---- End Embedded OS Boot ----
EmbeddedOSInstallService: Resetting the device into recovery mode since an error occurred during boot
EmbeddedOSInstallService: Waiting for recovery mode device
EmbeddedOSInstallService: Done waiting for recovery mode device
EmbeddedOSInstallService: Waiting for device boot
EmbeddedOSInstallService: Checking for healing overrides
EmbeddedOSInstallService: SMC is in app mode
EmbeddedOSInstallService: Should heal: YES, Found recovery mode device (after reboot attempt) (took 4.915 seconds)
EmbeddedOSInstallService: Enqueuing restore
EmbeddedOSInstallService: Starting restore
EmbeddedOSInstallService: Disabling retrying with AC for loginwindow
EmbeddedOSInstallService: Setting bootFailedAfterShouldHeal
EmbeddedOSInstallService: (EmbeddedOSInstall) Embedded OS Restore
EmbeddedOSInstallService: Getting information about current device for diagnostics
EmbeddedOSInstallService: Choosing between bundle specifiers: (
"EOSRestoreBundle (14Y363)"
)
EmbeddedOSInstallService: Chose EOSRestoreBundle (14Y363)
EmbeddedOSInstallService: Using restore bundle: EOSRestoreBundle (14Y363)
EmbeddedOSInstallService: Attempting to locate preflight container for restore bundle
EmbeddedOSInstallService: Preflight container matches bundle specifier: EOSPreflightContainer <file:///Library/Updates/PreflightContainers/FE0EC8F3-E032-49C1-B6E2-EA42171165AB.preflightContainer/> (preflighted on 2016-11-24 07:00:02 +0000)
EmbeddedOSInstallService: Found preflight container: EOSPreflightContainer <file:///Library/Updates/PreflightContainers/FE0EC8F3-E032-49C1-B6E2-EA42171165AB.preflightContainer/> (preflighted on 2016-11-24 07:00:02 +0000)
EmbeddedOSInstallService: Set restore bundle: EOSRestoreBundle (14Y363) (PKBundleComponentVersion )
EmbeddedOSInstallService: Set FDR memory store: /Library/Updates/PreflightContainers/FE0EC8F3-E032-49C1-B6E2-EA42171165AB.preflightContainer/FDRData.plist
EmbeddedOSInstallService: Loading restoreOptions from plist: /Library/Updates/PreflightContainers/FE0EC8F3-E032-49C1-B6E2-EA42171165AB.preflightContainer/personalized/EOS_RestoreOptions.plist
EmbeddedOSInstallService: Updating restore options with preflight container paths
EmbeddedOSInstallService: (EmbeddedOSInstall) network reachability check
EmbeddedOSInstallService: ---- Starting network reachability check ----
EmbeddedOSInstallService: We have a valid preflighted container, no network is required
EmbeddedOSInstallService: (EmbeddedOSInstall) prepare device
EmbeddedOSInstallService: ---- Starting prepare device ----
EmbeddedOSInstallService: Waiting for Storage Kit to populate disks
EmbeddedOSInstallService: Done waiting for Storage Kit to populate disks
EmbeddedOSInstallService: Restore bundle was preflighted, forcing reset into recovery for restore
EmbeddedOSInstallService: Entering recovery mode, starting command prompt
EmbeddedOSInstallService: Device is now in recovery mode
EmbeddedOSInstallService: (EmbeddedOSInstall) personalization
EmbeddedOSInstallService: ---- Starting personalization ----
EmbeddedOSInstallService: We already have a valid preflighted container, skipping
EmbeddedOSInstallService: (EmbeddedOSInstall) bootstrap recovery mode
EmbeddedOSInstallService: ---- Starting bootstrap recovery mode ----
EmbeddedOSInstallService: Starting recovery mode restore
EmbeddedOSInstallService: Starting recovery restore
EmbeddedOSInstallService: Recovery mode restore succeeded
EmbeddedOSInstallService: Waiting for device to be disconnected
EmbeddedOSInstallService: ---- Starting restore mode restore (using restored) ----
EmbeddedOSInstallService: Waiting for device to be connected
EmbeddedOSInstallService: Starting restore mode restore
EmbeddedOSInstallService: Restore mode restore success!
EmbeddedOSInstallService: Sleeping 10 seconds to wait for nvram flush
EmbeddedOSInstallService: (EmbeddedOSInstall) force reset
EmbeddedOSInstallService: ---- Starting force reset ----
EmbeddedOSInstallService: Resetting device
EmbeddedOSInstallService: Waiting for device to be connected
EmbeddedOSInstallService: (EmbeddedOSInstall) recovery mode restore (OS ramdisk)
EmbeddedOSInstallService: ---- Starting recovery mode restore (OS ramdisk) ----
EmbeddedOSInstallService: Waiting for device to be connected
EmbeddedOSInstallService: Using OSRamdisk tag for second boot
EmbeddedOSInstallService: Starting recovery restore
EmbeddedOSInstallService: Recovery mode restore succeeded
EmbeddedOSInstallService: Waiting for device to be disconnected
EmbeddedOSInstallService: (EmbeddedOSInstall) wait for boot
EmbeddedOSInstallService: ---- Starting wait for boot ----
EmbeddedOSInstallService: Waiting for device to be connected
EmbeddedOSInstallService: Device boot complete!
EmbeddedOSInstallService: (EmbeddedOSInstall) EFI system partition installation
EmbeddedOSInstallService: ---- Starting EFI system partition installation ----
EmbeddedOSInstallService: Memboot image checksum (/var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T/3E465740-A3BD-4EE6-98D6-FE4DB003C10B-EmbeddedOSInstallESPSandbox/combined.memboot): 1904138235
EmbeddedOSInstallService: Mounting EFI system partition
EmbeddedOSInstallService: Error getting size of directory: /Volumes/EFI/EFI/APPLE/EMBEDDEDOS, returning 0: Error Domain=NSPOSIXErrorDomain Code=2 "No such file or directory" UserInfo={NSFilePath=/Volumes/EFI/EFI/APPLE/EMBEDDEDOS}
EmbeddedOSInstallService: Creating intermediate directory: /Volumes/EFI/EFI/APPLE
EmbeddedOSInstallService: Shoving /var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T/3E465740-A3BD-4EE6-98D6-FE4DB003C10B-EmbeddedOSInstallESPSandbox to /Volumes/EFI/EFI/APPLE/EMBEDDEDOS
EmbeddedOSInstallService: Memboot image checksum (/Volumes/EFI/EFI/APPLE/EMBEDDEDOS/combined.memboot): 1904138235
EmbeddedOSInstallService: Cleaning up sandbox: /var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T/3E465740-A3BD-4EE6-98D6-FE4DB003C10B-EmbeddedOSInstallESPSandbox
EmbeddedOSInstallService: Unmounting EFI system partition
EmbeddedOSInstallService: (EmbeddedOSInstall) purge preflight containers
EmbeddedOSInstallService: ---- Starting purge preflight containers ----
EmbeddedOSInstallService: Purging all preflight containers
EmbeddedOSInstallService: (EmbeddedOSInstall) [com.apple.mac.install.EmbeddedOSSerial]
EmbeddedOSInstallService: Restore time: 104.8 seconds
EmbeddedOSInstallService: Restore was successful!
EmbeddedOSInstallService: Diagnostic summary: Restore ((null) -> 14Y363 (Customer Boot), preflighted = 1, prod fused = 1, user auth = 0, retries = 0, after boot failure = 1, failing phase = 0, uuid = 2359AA1C-98E9-4B21-96A5-D710317AD57E): success
EmbeddedOSInstallService: ---- End Embedded OS Restore ----
EmbeddedOSInstallService: Adding client: biometrickitd (pid = 232, uid = 0, path = /usr/libexec/biometrickitd)
EmbeddedOSInstallService: Enqueing get local FDR data
EmbeddedOSInstallService: Starting get local FDR data
EmbeddedOSInstallService: Waiting for Storage Kit to populate disks
EmbeddedOSInstallService: Done waiting for Storage Kit to populate disks
EmbeddedOSInstallService: (CoreFoundation) Loading Preferences From System CFPrefsD For Search List
Local Preflight Container
The local preflight container has some interesting information, although I won’t dive into this too much on this post.

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
ls /Library/Updates/PreflightContainers/guid.preflightContainer
FDRData.plist metadata.plist personalized
 
ls /Library/Updates/PreflightContainers/guid.preflightContainer/personalized/
BuildManifest.plist
EOS_RestoreOptions.plist
Firmware
KernelCache_kernelcache.release.img4
OSRamdisk_058-40573-247.img4
Restore.plist
RestoreKernelCache_kernelcache.release.img4
RestoreRamDisk_058-27707-457.img4
amai
version.plist
 
ls /Library/Updates/PreflightContainers/guid.preflightContainer/personalized/amai
apimg4ticket.der debug receipt.plist
 
ls /Library/Updates/PreflightContainers/guid.preflightContainer/personalized/amai/debug
tss-request.plist tss-response.plist
 
ls /Library/Updates/PreflightContainers/guid.preflightContainer/personalized/Firmware
all_flash dfu
 
ls /Library/Updates/PreflightContainers/guid.preflightContainer/personalized/Firmware/all_flash
all_flash.x619ap.production
 
ls /Library/Updates/PreflightContainers/guid.preflightContainer/personalized/Firmware/all_flash/all_flash.x619ap.production/
DeviceTree_DeviceTree.x619ap.img4
LLB_LLB.x619.RELEASE.img4
RestoreDeviceTree_DeviceTree.x619ap.img4
RestoreSEP_sep-firmware.x619.RELEASE.img4
SEP_sep-firmware.x619.RELEASE.img4
iBoot_iBoot.x619.RELEASE.img4
manifest
 
ls /Library/Updates/PreflightContainers/guid.preflightContainer/personalized/Firmware/dfu
iBEC_iBEC.x619.RELEASE.img4 iBSS_iBSS.x619.RELEASE.img4
A few interesting pieces:
* There appears to be a tss-request, tss-response and receipt, which seem to be signing data from Apple. One could assume this is for production activation of the OS.
* There are img4 files, all related to booting, restoring and DFU modes. These are more than likely signed files extracted from /usr/standalone/firmware/iBridge1_1Customer.bundle
* From the Receipt.plist, the TouchBar appears to be denoted as Watch2,5 running watchOS 3.0-14Y363.
* Many of these plist’s contain keys, more than likely pointing to certificate based authentication/identities.
* There appears to be logic to detect whether the TBP has a fuse set. We might also be able to send specific variants of the OS. No build variant specified and device is prod-fused, choosing customer variant.

Fun bugs
Given that iOS has had difficulty in the past with time, I hopped in my DeLorean and went back to 01-01-1970. Sure enough, the TouchBar failed to load, Touch ID was broken and the machine took a significant amount of time to boot.

While there were no logging events with EmbeddedOSInstall, the user experience was terrible. This is what the user sees for 2+ minutes.

1970 TouchBar

One can imagine that the OS is no longer validated, but why doesn’t Apple attempt to detect a network, and then run ntpdate if the time if incorrect? While I have not submitted a radar for this, I plan very soon. Why Apple continues to not test time sync situations is beyond me.

The wonderful future of hybrid hardware
It’s quite clear – Welcome to the future of Apple’s hybrid ARM/x86 platform.

It’s also quite clear that destroying entire disks is going to lead to some pain points for people still imaging.

I have some concerns though. For several years, Apple has moved firmware/EFI updates into the delta/combo updaters and imaging has not been able to solve this issue.

What happens when we build a 10.12.2 image and the EFI is out of date?
Does EmbeddedOSInstallService also check signing windows?
If it does, what happens when we deploy an image that doesn’t contain the new firmware?
Will there come a time that much like iOS, a MacBook Pro cannot be restored to an older OS?
Will Apple consider wiping the primary OS volume only a security vulnerability, and cause this message to occur during any re-image of the volume?
I think we will have our answers soon, but it will be up to the community to figure this out.

As a macadmin, does this impact me?
Are deploying a thin, modular or thick image?
Are you doing some kind of first boot scripting / boot process, ie LoginLog or DeployStudio finalize scripts?
Are you bootstrapping munki?
If so, the key to not encountering this issue is by targeting only the current Macintosh HD volume of the machine.

IMAGR USERS
Imagr targets the first available volume by default. For FileVault enabled disks, you typically deleted the entire drive or just the encrypted volume via Disk Utility.

If you are deleting the entire drive, stop! Delete the current volume and then run your imagr workflow. This will allow your automation workflows/bootstraps to continue to work.

DEPLOYSTUDIO USERS
DeployStudio should be okay, but I recommend changing your restore workflow to:

Target Volume: Enter Value – Macintosh HD

While this option may not work for everyone (and may fail if people rename the default Macintosh HD naming convention), it will ensure that the volume itself is targeted.

First Disk available is more than likely an option you want to stay away from, but I have not tested this theory. Please report your findings and if you run into any issues, post on the DeployStudio forums. Or better yet, move to Imagr! :)

INTERNET RECOVERY USERS
Unfortunately, it seems if you are using Internet Recovery and wipe the entire disk, the critical update component will still be needed to complete, even though Internet Recovery has an internet component.

This is problematic if you wipe your devices prior to re-allocation and using DEP. This may also need a radar.

RADARS SUBMITTED TO APPLE
SetupAssistant does not detect non-wireless network to repair EmbeddedOS on Touch Bar MacBook Pros
Internet Recovery does not activate Embedded OS on Touch Bar MacBook Pros
Embedded OS/TouchBar MacBook Pro causes significant boot delays and malfunctions when the time isn’t functioning.
Apple are you listening?
Apple if you are reading this, can you please outline this process for us? A simple knowledge base will not be enough. We need a up-to-date portal with information regarding the future of mac management. Documentation should not be put on the backs of the companies that use your products.

We need answers and we need answers soon. Leaving us in the dark about future processes is bad for everyone. It’s bad for the community, bad for the companies using your products and eventually it will become bad for those millions of iOS/macOS developers you cherish.

Bring back the portion of WWDC for macadmins – invite us! We would love to talk! :)

Thanks to Michael Lynn and Pepijn Bruienne for working with me late Wednesday night. As we continue to dissect this, I hope to see more in-depth discoveries.

CATEGORIES MAC ADMINISTRATION
Apple’s EFI logonui – managing macOS Sierra’s wallpaper
PUBLISHED ON September 24, 20166 Comments
Update 1: Now with fix (Thanks Owen for inspiration!)

Update 2: Now with safer methodology (Thanks Michael Lynn/Frogor)

Update 3: Now with further management tools/discoveries.

Update 4: Note about non-blurred wallpapers and extra data

A few days ago there was a post on MacEnterprise about issues setting Sierra’s login wallpaper. I immediately posted as I had not seen the issues that were described, but after doing a deep dive last night, it is true – there are some changes to this functionality.

El Capitan Notes
In El Capitan, one could simply do the following:
* Copy your selected wallpaper in .png format to /Library/Caches (Example: cp -R ~/Desktop/mysuperawesomewallpaper.png /Library/Caches/com.apple.desktop.admin.png)
* ensure root was the owner (Example: chown root:wheel /Library/Caches/com.apple.desktop.admin.png )
* ensure it was world readable (Example: chmod 755 /Library/Caches/com.apple.desktop.admin.png
* set idempotent flag, so it cannot be overwritten (Example: chflags uchg /Library/Caches/com.apple.desktop.admin.png)

These four steps could be automated with a package/script/etc and were pretty simple to configure.

Sierra Notes
With Sierra, while this methodology still works, it is incomplete. You will notice the following:
* Loginwindow not get updated
* FileVault pre-boot window not updated
* User lock screen not updated

Unfortunately for the 3rd item, this cannot be modified, as Apple automatically overlays a blur on top of the user’s wallpaper. With that said, the other two items can be corrected, albeit with some unfortunate requirements.

Down the rabbit hole we go…
If you go to System Preferences -> Desktop & Screen Saver and change your wallpaper you will notice that immediately com.apple.desktop.admin.png is changed, the loginwindow is updated and if you reboot, the FileVault pre-boot is also updated.

loginwindow

Let’s do a quick inspection into what could be happening:

1
2
3
4
5
6
7
8
<snippedforclarity>
fs_usage | grep desktop
/Users/User/Desktop/test.png
/Users/User/Library/Application Support/Dock/desktoppicture.db
/private/var/folders/zr/57b51nwn08ddmvzlc5fplk1r0000gn/C/com.apple.desktoppicture/83D89DB4E2232FC43CE0DB6E06AFD223-2880.png
/private/var/folders/zr/57b51nwn08ddmvzlc5fplk1r0000gn/T//.6EB3C2D5-43C8-4464-980D-76782C758FEB-com.apple.desktop.admin.png-kmyp
/private/var/folders/zr/57b51nwn08ddmvzlc5fplk1r0000gn/T//6EB3C2D5-43C8-4464-980D-76782C758FEB-com.apple.desktop.admin.png
/Library/Caches/com.apple.desktop.admin.png
This is pretty clear as to what is happening. Apple is taking our png, writing it’s value to the desktoppicture.db, creating a temporary folder and generating a resolution specific png, then transferring it to /Library/Caches.

So if that’s all it does, why is it when we recreate this scenario through a package/script, only some of the elements are updated? Perhaps our grep left out some important pieces…

UNLEASH THE KRAKEN
To give you an idea on why one would grep, fs_usage is extremely verbose. The prior example’s output was around 400 event lines, whereas without grep, we are at 281,000 event lines.

With that said, we need to get to the bottom of what is happening.

1
2
3
4
5
6
7
8
9
10
11
12
13
14
<snippedforclarity>
fs_usage
/Users/User/Desktop/test.png
/Users/User/Library/Application Support/Dock/desktoppicture.db
/private/var/folders/zr/57b51nwn08ddmvzlc5fplk1r0000gn/C/com.apple.desktoppicture/83D89DB4E2232FC43CE0DB6E06AFD223-2880.png
/private/var/folders/zr/57b51nwn08ddmvzlc5fplk1r0000gn/T//.6EB3C2D5-43C8-4464-980D-76782C758FEB-com.apple.desktop.admin.png-kmyp
/private/var/folders/zr/57b51nwn08ddmvzlc5fplk1r0000gn/T//6EB3C2D5-43C8-4464-980D-76782C758FEB-com.apple.desktop.admin.png
/Library/Caches/.com.apple.updateEFIResources
/usr/standalone/bootcaches.plist
/System/Library/PrivateFrameworks/EFILogin.framework/Versions/A/Resources/efilogin-helper
/System/Library/PrivateFrameworks/EFILogin.framework/Versions/A/Resources/EFIResourceBuilder.bundle
/System/Library/Caches/com.apple.corestorage/EFILoginLocalizations
/Library/Preferences/SystemConfiguration/com.Apple.Boot.plist
/Library/Caches/com.apple.desktop.admin.png
What a difference!

A couple of things immediately stick out:
* Two preference files
* Many references to EFI Resources
* A dotfile that seems to trigger an update to the EFI
* A SIP Private Framework with a helper and rebuild tool
* Another cache (!) that is SIP protected

All signs point to EFI resources, but we need to prove this theory.

PLISTS
First, let’s take a look at these plist files.

1
2
3
4
5
6
7
8
9
defaults read /usr/standalone/bootcaches.plist
PostBootPaths =     {
    BootConfig = "/Library/Preferences/SystemConfiguration/com.apple.Boot.plist";
    EncryptedRoot =         {
        BackgroundImage = "/Library/Caches/com.apple.desktop.admin.png";
        DefaultResourcesDir = "/usr/standalone/i386/EfiLoginUI/";
        LocalizationSource = "/System/Library/PrivateFrameworks/EFILogin.framework/Resources/EFIResourceBuilder.bundle/Contents/Resources";
        LocalizedResourcesCache = "/System/Library/Caches/com.apple.corestorage/EFILoginLocalizations";
    };
This is fairly interesting. We now know the bootcache BackgroundImage is set (which is why if you simply deploy the admin.png and reboot it works), there is a default EFI boot cache and a subsequent localization EFI boot cache.

1
2
3
4
defaults read /Library/Preferences/SystemConfiguration/com.Apple.Boot.plist
{
    "Kernel Flags" = "";
}
This preference file is a little less interesting, however this is an older preference file that is becoming less relevant. Older macadmins may remember this was used prior to Mavericks to set a custom boot image.

EFI BOOT CACHES
1
2
3
4
5
6
7
8
9
10
11
12
13
ls -lO /usr/standalone/i386/EfiLoginUI/
-rw-r--r--  1 root  wheel  restricted 1069412 Jul 30 15:41 Lucida13.efires
-rw-r--r--  1 root  wheel  restricted 1067888 Jul 30 15:41 Lucida13White.efires
-rw-r--r--  1 root  wheel  restricted   17468 Jul 30 15:41 appleLogo.efires
-rw-r--r--  1 root  wheel  restricted 1060947 Jul 30 15:41 battery.efires
-rw-r--r--  1 root  wheel  restricted  437314 Jul 30 15:41 disk_passwordUI.efires
-rw-r--r--  1 root  wheel  restricted 2656877 Jul 30 15:41 flag_picker.efires
-rw-r--r--  1 root  wheel  restricted  212250 Jul 30 15:41 guest_userUI.efires
-rw-r--r--  1 root  wheel  restricted 2013232 Jul 30 15:41 loginui.efires
-rw-r--r--  1 root  wheel  restricted   56468 Jul 30 15:41 recoveryUI.efires
-rw-r--r--  1 root  wheel  restricted    9882 Jul 30 15:41 recovery_user.efires
-rw-r--r--  1 root  wheel  restricted  170570 Jul 30 15:41 sound.efires
-rw-r--r--  1 root  wheel  restricted  250828 Jul 30 15:41 unknown_userUI.efires
So this cache seems to be SIP protected and dated in between Sierra Developer Beta 1 & 2. As we saw in /usr/standalone/bootcaches.plist, this is the default cache and probably only used for first-time boots of Sierra.

1
2
3
4
5
6
7
8
9
10
11
12
ls -lO /System/Library/Caches/com.apple.corestorage/EFILoginLocalizations
-rw-r--r--  1 root  wheel  -  1069412 Sep 24 22:03 Lucida13.efires
-rw-r--r--  1 root  wheel  -  1067888 Sep 24 22:03 Lucida13White.efires
-rw-r--r--  1 root  wheel  -    13258 Sep 24 22:03 appleLogo.efires
-rw-r--r--  1 root  wheel  -  1043933 Sep 24 22:03 battery.efires
-rw-r--r--  1 root  wheel  -   443692 Sep 24 22:03 disk_passwordUI.efires
-rw-r--r--  1 root  wheel  -  2832469 Sep 24 22:03 flag_picker.efires
-rw-r--r--  1 root  wheel  -   214346 Sep 24 22:03 guest_userUI.efires
-rw-r--r--  1 root  wheel  - 10392791 Sep 24 22:03 loginui.efires
-rw-r--r--  1 root  wheel  -    25939 Sep 24 22:03 preferences.efires
-rw-r--r--  1 root  wheel  -   171012 Sep 24 22:03 sound.efires
-rw-r--r--  1 root  wheel  -   252398 Sep 24 22:03 unknown_userUI.efires
Now we’re getting somewhere. These files were modified at the same time we changed the Desktop wallpaper. There’s also conveniently a logonui.efires, which might be the file we are looking for.

All signs are pointing to this EFI resource location, but we still don’t definitely know if this is where the issue is.

What the hell is an efires file?
If you’ve never kept up with Hackintosh community, you probably have never heard about these files. Piker-Alpha and his sister (RIP) are largely the ones who first started doing deep dives into these files.

.efires files are EFI Resource files. They are LZVN compressed files, that contain various images that Apple uses during the boot process.

BACKSTORY AND SHAMELESS PLUG
Last year, with the help Michael Lynn/Frogor, I released BootPicker, a PSD file for easily creating Apple boot documentation. We dumped the .efire files and extracted the boot files, giving us the exact images for Apple’s boot process. It’s better than Apple’s own documentation, with fake assets!. Piker/Sam’s original work with LZVN and efires-extract were instrumental in our success.

I never did a post on BootPicker, but check it out – I think you’ll love it!

BACK TO SIERRA
Unfortunately, Piker-Alpha’s efires-extract does not take into account Sierra’s new folder structure, however I have modified it to now work. You can find a copy here.

Let’s spin it up.

1
2
3
4
5
6
7
8
9
./efires-extract
<snipped>
Filename: loginui.efires
EFI revision: 2
Number of packed images: 119
Header length: 8644
<snipped>
Image(1): loginui_background.png (offset: 15320/0x3bd8, size: 9805757/0x959fbd) Read: 9805757
<snipped>
Well look at that. loginui_background.png :tada:

So what exactly is that file?

loginui_background

Oh look, it’s the same file as com.apple.desktop.admin.png!

So we have definitely confirmed our initial suspicions:

One simply can’t deploy com.apple.desktop.admin.png and be done.
/System/Library/PrivateFrameworks/EFILogin.framework/Versions/A/Resources/EFIResourceBuilder.bundle is the key to rebuilding the EFI cache.
There may be a trigger at /Library/Caches/.com.apple.updateEFIResources
GUI Triggers and CLI SIP Brick Wall
You may already know where this is going.

The following GUI actions, cause EFIResourceBuilder to trigger:
* Right Click, settings Wallpaper
* System Preferences -> Desktop & Screensaver -> Selecting new wallpaper
* System Preferences -> Security & Privacy -> Set Lock Message

In my further investigation using Hopper, I attempted to find out how to use the dot file to trigger updating the EFI. Unfortunately I could not figure it out. I also attempted to configure /Library/Preferences/com.apple.loginwindow.plist LoginwindowText which is what the Security & Privacy Lock Message does, but unfortunately that alone did not trigger an EFI rebuild.

Interestingly enough, when googling about the EFILogin framework, I found this link on JAMF Nation that outlined a method to trigger the EFI rebuild process.

Let’s try it:

1
2
touch /System/Library/PrivateFrameworks/EFILogin.framework/Resources/EFIResourceBuilder.bundle/Contents/Resources
touch: /System/Library/PrivateFrameworks/EFILogin.framework/Resources/EFIResourceBuilder.bundle/Contents/Resources: Permission denied
:unamused:

1
2
3
4
5
6
ls -lO /System/Library/PrivateFrameworks/EFILogin.framework/Resources/EFIResourceBuilder.bundle/Contents/
-rw-r--r--    1 root  wheel  restricted,compressed 1264 Jul 30 18:47 Info.plist
drwxr-xr-x    3 root  wheel  restricted             102 Sep 13 17:57 MacOS
drwxr-xr-x  131 root  wheel  restricted            4454 Sep 24 11:53 Resources
drwxr-xr-x    3 root  wheel  restricted             102 Jul 30 18:48 _CodeSignature
-rw-r--r--    1 root  wheel  restricted,compressed  523 Jul 30 18:48 version.plist
:cry:

DISABLING SIP
I just want to get this out of the way…

DO NOT DISABLE SIP TO DEPLOY A CUSTOM WALLPAPER!

We are going to disable SIP, temporarily, to test if touching this file still works. You know the drill: reboot, disable, reboot.

1
touch /System/Library/PrivateFrameworks/EFILogin.framework/Resources/EFIResourceBuilder.bundle/Contents/Resources
Okay so no more operation not permitted errors, but did it work?

1
2
ls -lO /System/Library/Caches/com.apple.corestorage/EFILoginLocalizations
-rw-r--r--  1 root  wheel  - 10392791 Sep 24 22:38 loginui.efires
Yep! The .efires timestamp are reflecting the current time. So while we had to disable SIP, this method still works and was probably an oversight on Apple’s part. Shame on all of us for not noticing this until now.

Closing Thoughts (OpenRadar)
It’s quite apparent that Apple will continue to use the .efires files for their boot process and while they allow an alternative cache to exist, if you want your loginwindow to be updated, you must update the EFI cache.

While I do wish Apple would have more documentation related to Enterprise Management, Apple has historically been cagey on this process. SIP bug withstanding, we now definitely know what the process is:
* Add custom com.apple.desktop.admin.png to /Library/Caches
* Take ownership of the png and mark it as idempotent to ensure no one else can modify it accidentally
* Rebuild EFI cache for loginwinow and FileVault screens

For admins that want to duplicate my issue or reach out to their System Engineer, here is the link.

rdar://28462923

Updated with fix
Thanks to some detective work by my buddies, we have found an alternative method to triggering the event.

1
2
3
4
5
6
7
8
defaults read /System/Library/PreferencePanes/DesktopScreenEffectsPref.prefPane/Contents/Resources/DesktopPictures.prefPane/Contents/Resources/com.apple.updateEFIDesktopPicture.plist
{
Label = "com.apple.updateEFIDesktopPicture";
ProgramArguments = (
"/usr/sbin/kextcache",
"-u",
"/"
);
From the manpage of kextcache:

1
2
3
4
5
6
7
8
-u os_volume, -update-volume os_volume
Rebuild out-of-date caches and update any helper partitions associated with os_volume.
os_volume/System/Library/Caches/com.apple.bootstamps/ is used to track whether any helper partitions
are up to date. See -caches-only and -force.
 
Which caches are rebuilt depends on the Mac OS X release installed on os_volume. If kextcache
cannot find or make sense of os_volume/usr/standalone/bootcaches.plist the volume is treated
as if no caches need updating: success is returned.
By invoking kextcache, we can update the caches! We just need to force it, since once has previously been created.

Here are the exact steps to have a working/admin deployed LoginUI.
* Copy your selected wallpaper in .png format to /Library/Caches (Example: cp -R ~/Desktop/mysuperawesomewallpaper.png /Library/Caches/com.apple.desktop.admin.png)
* ensure root was the owner (Example: chown root:wheel /Library/Caches/com.apple.desktop.admin.png )
* ensure it was world readable (Example: chmod 755 /Library/Caches/com.apple.desktop.admin.png
* set idempotent flag, so it cannot be overwritten (Example: chflags uchg /Library/Caches/com.apple.desktop.admin.png)
* delete the unprotected EFI caches (Exmaple: rm -rf /System/Library/Caches/com.apple.corestorage/EFILoginLocalizations/*.*)
* force a rebuild of the cache (Example: /usr/sbin/kextcache -fu /)

Update 2
Instead of deleting the unprotected EFI caches and using kextcache, you can do the following:

touch /System/Library/Caches/com.apple.corestorage/EFILoginLocalizations
force a rebuild of the cache (Example: /usr/sbin/kextcache -fu /)
Much love to the -fu flag Apple. :)

Update 3
Earlier someone reached out to me and informed me that the login screen was not being truly forced. After looking at things again, I discovered /System/Library/PrivateFrameworks/LoginUIKit.framework/Versions/A/LoginUIKit. In this binary are some interesting leads.

This tool automatically creates a gaussian blur each time the loginwindow is displayed to the end user. By default it will look at /System/Library/CoreServices/DefaultDesktop.jpg which is actually a SIP protected symbolic link that in Sierra points to /Library/Desktop Pictures/Sierra.jpg. Pay close attentions here. While the symbolic link is protected, the default Sierra wallpaper is not.

If an admin wants have a default wallpaper upon login for their users and a default loginwindow for their users, replacing this jpg with a custom one would be sufficient.

But what if an admin wants to control this at all times? Interestingly enough, while not documented anywhere online, Apple created a special preference for just this thing! There is a key ForceDefaultDesktop, a boolean type that can be enabled on com.apple.loginwindow.

To finalize – If an admin wants to fully control the FileVault EFI loginwindow and the user loginwindow, one must do everything from Update 2 and the following:

Replace /Library/Desktop Pictures/Sierra.jpg with a custom picture (Example: cp -R ~/Desktop/mysuperawesomewallpaper.jpg /Library/Desktop Pictures/Sierra.jpg
Optionally make the new Sierra.jpg idempotent to prevent changes. (Example: chflags uchg /Library/Desktop Pictures/Sierra.jpg)
Configure com.apple.loginwindow (Example: defaults write /Library/Preferences/com.apple.loginwindow ForceDefaultDesktop -bool true)
To be clear, this method will use a gaussian blur at the loginwindow, but allow you to force a specific wallpaper.

Update 4
In further testing, if your non-blurred wallpaper is not working, there are chances that you have extra EXIF data added to your png, which will cause the loginwindow to bail and use the default Sierra wallpaper.

I recommend opening up your wallpaper in Preview.app and then saving it. This should remove the extra data and your wallpaper should then load at the loginwindow.

*This post proudly brought to you by, 3.5mm jack headphones.

CATEGORIES MAC ADMINISTRATION
Advanced Logging for Apple Configurator 2
PUBLISHED ON February 1, 2016Leave a comment
While I don’t have much time to write a thorough post on how I found this information, while playing with Hopper I found out that you can have some incredibly verbose logs for Apple Configurator 2.

defaults write ~/Library/Containers/com.apple.configurator.ui/Data/Library/Preferences/com.apple.configurator.ui.plist ACULogLevel -string ALL

While I still enjoy iOS Logger, these logs contain advanced logging information outside of the device itself.

If you find the logs too verbose run the following command.

defaults delete ~/Library/Containers/com.apple.configurator.ui/Data/Library/Preferences/com.apple.configurator.ui.plist ACULogLevel

```

