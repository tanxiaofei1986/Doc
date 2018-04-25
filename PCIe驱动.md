
# SAS驱动初始化过程
初识PCIe驱动
https://www.cnblogs.com/xiaoya901109/archive/2012/12/14/2818057.html

## 1. PCIe设备的初始化
pci_enable_device
启动PCI设备

pci_set_master
设置成总线主DMA模式

pci_request_regions
当前PCI将使用这些内存地址，其他设备不能再使用了

pci_set_dma_mask
设备DMA标识

pcim_iomap
map寄存器空间，或者说bar空间	

## 2. SCSI host初始化

scsi_host_alloc
Allocate a new Scsi_Host and perform basic initialization. The host is not published to the scsi midlayer until scsi_add_host is called.

scsi_add_host
添加对象，注册到scsi middle level。SCSI的3层结构。

sas_register_ha
libsas的接口，

scsi_scan_host
会调用scan_start, 初始化phy
