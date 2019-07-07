# Libvirt-for-VFIO-ThreadRipper-
Libvirt for VFIO and ThreadRipper 


# Notes:

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


###### Use hugepages
* **Add:**
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

