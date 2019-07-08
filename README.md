# Libvirt-for-VFIO-ThreadRipper
Libvirt for VFIO and ThreadRipper 

# Notes:
```
  The following are my notes to get my threadripper AMD system to run VFIO.
  Any corrections are more than welcomed.
```

## Use Arch Kernel 5.1.16 with patches 
* **Linux Kernel with acs-overrides patches**
  * **Ref:** https://github.com/net-error/VFIO-ThreadRipper-arch1

## Regression: QEMU 4.0 hangs the host
* **Fix:** Add `ernel_irqchip=on` or `<ioapic driver='kvm'/>` to guest xml.

* **Example:**
```
</features>
   ...
   ...
   <vmport state='off'/>
   <ioapic driver='kvm'/>
</features>
```

* **Ref:** https://bugs.launchpad.net/qemu/+bug/1826422


## Use hugepages
* **Add: Guest xml**
```
<domain type='kvm'>
  ...
  ...
  <memoryBacking>    
    <hugepages/>    
  </memoryBacking>
  ...
</domain>
```

* **Ref:** https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF#Huge_memory_pages

## Use 2 iothreads
* **Add: Guest xml**
```
<domain type='kvm'>
  ...
  ...
  <iothreads>2</iothreads>
  ...
```

## Use cputune for your CPU
* **Example for a AMD ThreadRipper 2920x:**
   * 12 cores for guest
```
  <cputune>
    <vcpupin vcpu='0' cpuset='2'/>
    <vcpupin vcpu='1' cpuset='3'/>
    <vcpupin vcpu='2' cpuset='4'/>
    <vcpupin vcpu='3' cpuset='5'/>    
    <vcpupin vcpu='4' cpuset='6'/>    
    <vcpupin vcpu='5' cpuset='7'/>    
    <vcpupin vcpu='6' cpuset='8'/>    
    <vcpupin vcpu='7' cpuset='9'/>    
    <vcpupin vcpu='8' cpuset='10'/>    
    <vcpupin vcpu='9' cpuset='11'/>    
    <vcpupin vcpu='10' cpuset='12'/>    
    <vcpupin vcpu='11' cpuset='13'/>    
    <emulatorpin cpuset='0-1'/>    
    <iothreadpin iothread='1' cpuset='0-1'/>    
  </cputune>
```
**Ref:** https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF#CPU_pinning

## Use Q35 chipset and Pure EFI
  * Q35 will allow PCIE speeds
  * Pure EFI will allow Nvidia VGA passthrough

* **Add: Guest xml**
```
  <os>
    <type arch='x86_64' machine='pc-q35-4.0'>hvm</type>
    <loader readonly='yes' type='pflash'>/usr/share/ovmf-x64/OVMF_CODE-pure-efi.fd</loader>
    <nvram>/var/lib/libvirt/qemu/nvram/WIN10_VARS.fd</nvram>
  </os>
```
* **Ref:** https://forum.level1techs.com/t/increasing-vfio-vga-performance/133443

## Nvidia VGA passthrough fixes
  * Add vendor_id to hyperv to fix Nvidia code 43.
```
    <hyperv>
      <relaxed state='on'/>
      <vapic state='on'/>
      <spinlocks state='on' retries='8191'/>
      <vendor_id state='on' value='123456789012'/>
    </hyperv>
```
  * Hide KVM from windows to fix Nvidia code 43.
```
    <kvm>
      <hidden state='on'/>
    </kvm>
```

* **Ref:** https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF#%22Error_43:_Driver_failed_to_load%22_on_Nvidia_GPUs_passed_to_Windows_VMs

## Use host-passthrough and topoext for AMD cpus. 
* **Add: Guest xml**
```
  <cpu mode='host-passthrough' check='none'>
    <topology sockets='1' cores='6' threads='2'/>
    <cache level='3' mode='emulate'/>
    <feature policy='require' name='topoext'/>
  </cpu>
```

* **Ref:** https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF#Improving_performance_on_AMD_CPUs

## Use looking-glass and scream.
  ###### looking-glass is low latency virtual monitor using ivshmem
  ###### sream is low latency virtual audio using ivshmem

* **Add: Guest xml**
```
    <shmem name='looking-glass'>
      <model type='ivshmem-plain'/>
      <size unit='M'>64</size>
      <address type='pci' domain='0x0000' bus='0x0b' slot='0x01' function='0x0'/>
    </shmem>
    <shmem name='scream-ivshmem'>
      <model type='ivshmem-plain'/>
      <size unit='M'>2</size>
      <address type='pci' domain='0x0000' bus='0x0c' slot='0x07' function='0x0'/>
    </shmem>

```
  **Ref:** https://looking-glass.hostfission.com/
  **Ref:** https://github.com/duncanthrax/scream
  
