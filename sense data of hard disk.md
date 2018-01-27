
# Sense key and sense code of hard disk

## 1 Sense key definitions

Please refer to SPC-5 "4.4.8 Sense key and additional sense code definitions"
SENSE KEY
ADDITIONAL SENSE CODE

### 1.1 READ LOG EXT - 2Fh
### 1.2 READ LOG DMA EXT - 47h

ACS-4 "7.21 READ LOG EXT - 2Fh, PIO Data-In"
ACS-4 "7.22 READ LOG DMA EXT - 47h, DMA"
Log Address 10h: NCQ command error log, offset 14

ACS-4 "7.32 REQUEST SENSE DATA EXT - 0Bh, Non-Data"

## 2 driver call trace

ata_do_eh
ata_eh_autopsy
ata_eh_link_autopsy
ata_eh_analyze_ncq_error
ata_eh_read_log_10h(dev, &tag, &tf);
memcpy(&qc->result_tf, &tf, sizeof(tf));
ata_scsi_set_sense


 
hisi_sas_sata_done  ending_fis 保存 struct dev_to_host_fis
sas_ata_task_done  struct sata_device fis
ata_tf_from_fis
__ata_qc_complete 
qc->complete_fn = ata_scsi_qc_complete
ata_scsi_qc_complete --> ata_gen_ata_sense
if (ata_dev_disabled(dev)) ata_scsi_set_sense(dev, cmd, NOT_READY, 0x04, 0x21);

ata_do_eh
ata_dev_disable() class值加1，就变成disable了

ata_eh_revalidate_and_attach
