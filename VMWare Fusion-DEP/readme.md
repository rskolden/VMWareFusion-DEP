This is a manual on how you can test your DEP environment without the need of constantly re-install a physical machine.

Pre-reqs.:
1. A DEP enrolled Mac
2. VMWare Fusion
3. Some terminal knowledge

1. Create a new unbooted virtual machine
  a. Download "Install macOS Sierra" from Mac App store
  b. Create a "never-booted" .dmg using AutoDMG (https://github.com/MagerValp/AutoDMG)
  c. Create a new VMWare Fusion virtual machine (.vmdk) (sudo vfuse -i *.dmg-file -o *outputlocation)
  d. Import (drag and drop) into VMWare Fusion virtual machine library

2. Change setting so that Apple thinks the virtual machine is DEP enrolled
  a. "Show package content" on file "/Users/*userid/Documents/Virtual Machines/*newly created VM-file.vmdk"
  b. Open .vmx file with *text editor of your choice
  c. Find the following information from a DEP computer:
    I. board-id (var_ID=$(ioreg -p IODeviceTree -r -n / -d 1 | grep board-id);var_ID=${var_ID##*<\"};var_ID=${var_ID%%\">};echo $var_ID)
    II. hw.model (sysctl hw.model) (only the model, not "hw.model")
    III. serialNumber (system_profiler SPHardwareDataType | awk '/Serial/ {print $4}')
  d. Add the following lines, and save the file:
        board-id.reflectHost = "FALSE"
          board-id = "*board-id"
            hw.mode.reflectHost = "FALSE"
              hw.model = "*model"
                serialNumber.reflectHost = "FALSE"
                  serialNumber = "*serialnumber"
                    smbios.reflectHost = "FALSE"

3. Start the virtual machine and run the setup assistant as normal (DEP normally doesn't "hit" the first time you boot)

4. Once booted, run the following commands to "reset" the Apple Setup Assistant, this will trigger the DEP enrollment once rebooted:
    sudo mv /var/log/system.log{,.bak}
      sudo rm /var/db/.AppleSetupDone
        sudo rm -r /var/db/ConfigurationProfiles
          sudo rm /Library/Keychains/apsd.keychain

            sudo touch /var/db/MDM_EnableDebug
              sudo touch /var/db/MDM_CKSupportRequestsFromDaemon
                sudo shutdown -r now

5. After the machine has rebooted, DEP should now be active
