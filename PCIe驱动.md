
# SAS������ʼ������
��ʶPCIe����
https://www.cnblogs.com/xiaoya901109/archive/2012/12/14/2818057.html

## 1. PCIe�豸�ĳ�ʼ��
pci_enable_device
����PCI�豸

pci_set_master
���ó�������DMAģʽ

pci_request_regions
��ǰPCI��ʹ����Щ�ڴ��ַ�������豸������ʹ����

pci_set_dma_mask
�豸DMA��ʶ

pcim_iomap
map�Ĵ����ռ䣬����˵bar�ռ�	

## 2. SCSI host��ʼ��

scsi_host_alloc
Allocate a new Scsi_Host and perform basic initialization. The host is not published to the scsi midlayer until scsi_add_host is called.

scsi_add_host
��Ӷ���ע�ᵽscsi middle level��SCSI��3��ṹ��

sas_register_ha
libsas�Ľӿڣ�

scsi_scan_host
�����scan_start, ��ʼ��phy
