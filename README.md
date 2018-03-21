# VMWareFusion-DEP

**Contributors**
* _Martin Gunnarssson_
* _Various persons from macadmins slack-channel_

This is a manual on how you can test your DEP environment without the need of constantly re-install a physical machine.

_Pre-reqs.:_
* A DEP enrolled Mac
* VMWare Fusion
* vfuse https://github.com/chilcote/vfuse
* AutoDMG https://github.com/MagerValp/AutoDMG
* Some terminal knowledge

1. Create a new unbooted virtual machine
  * Download "Install macOS Sierra" from Mac App store
  * Create a "never-booted" .dmg using AutoDMG
  * Create a new VMWare Fusion virtual machine (.vmdk) (sudo vfuse -i .dmg-file -o outputlocation)
  * Import (drag and drop) into VMWare Fusion virtual machine library

2. Change setting so that Apple thinks the virtual machine is DEP enrolled
  * "Show package content" on file "/Users/userid/Documents/Virtual Machines/*newly created VM-file.vmdk"
  * Find the following information from a DEP computer:
    * board-id `var_ID=$(ioreg -p IODeviceTree -r -n / -d 1 | grep board-id);var_ID=${var_ID##*<\"};var_ID=${var_ID%%\">};echo $var_ID`
    * hw.model `sysctl hw.model` (only the model, not "hw.model")
    * serialNumber `system_profiler SPHardwareDataType | awk '/Serial/ {print $4}'`
  * Open .vmx file with a text editor of your choice
  * Add the following lines, and save the file:
    * board-id.reflectHost = "FALSE"
    * board-id = "board-id"
    * hw.model.reflectHost = "FALSE"
    * hw.model = "model"
    * serialNumber.reflectHost = "FALSE"
    * serialNumber = "serialnumber"
    * smbios.reflectHost = "FALSE"

3. Start the virtual machine and run the setup assistant as normal (DEP normally doesn't "hit" the first time you boot)

4. Once booted, run the following commands to "reset" the Apple Setup Assistant, this will trigger the DEP enrollment once rebooted:
```
sudo mv /var/log/system.log{,.bak}
sudo rm /var/db/.AppleSetupDone
sudo rm -r /var/db/ConfigurationProfiles
sudo rm /Library/Keychains/apsd.keychain
sudo touch /var/db/MDM_EnableDebug
sudo touch /var/db/MDM_CKSupportRequestsFromDaemon
sudo shutdown -r now
```

After the machine has rebooted, DEP should now be active
