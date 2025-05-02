====== Enable nested KVM ======

Open /etc/modprobe.d/kvm.conf as root using a text editor.
If the file does not exist create /etc/modprobe.d/kvm.conf
  * #Uncomment kvm_intel line if your CPU make is Intel
  * #options kvm_intel nested=1
  * #Uncomment kvm_intel line if your CPU make is AMD
  * #options kvm_amd nested=1

Save the file and reboot the system
Once the system reboots verify nested by checking

   cat /sys/module/kvm_intel/parameters/nested
   
   Y
 
For AMD the file is /sys/module/kvm_amd/parameters/nested

