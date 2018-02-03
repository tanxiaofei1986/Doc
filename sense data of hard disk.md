
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

同一个任务不停的循环
drivers/scsi/scsi_lib.c
__scsi_queue_insert()  
scsi_dev_queue_ready()  unblocking  device at zero depth
scsi_log_send()  Send: scmd
	scsi_print_command() CDB: opcode=
scsi_print_result()   Done: UNKNOWN(0X2006) Result: hostbyte=0x00 driverbyte=0x00
scsi_print_command()

sd_print_result()  Result:hostbyte=%s driverbyte=%s  盘卸载过程打印
	
scsi_softirq_done()
	scsi_log_completion()
		scsi_print_result()
		scsi_print_command()
	scsi_queue_insert()
		__scsi_queue_insert()

		
scsi_request_fn
	scsi_dev_queue_ready
	scsi_dispatch_cmd  //判断host->shost_state == SHOST_DEL, 然后直接返回，这种情况，所有盘都不能跑io
		scsi_log_send
	host->hostt->queuecommand(host, cmd);
		ata_sas_queuecmd  //如果device disabled了，也会直接返回 cmd->result = DID_BAD_TARGET << 16
			__ata_scsi_queuecmd
				ata_scsi_translate
					ata_scsi_qc_new
						ata_qc_new_init //如果分配失败，cmd->result = (DID_OK << 16) | (QUEUE_FULL << 1)，后续会ADD_TO_MLQUEUE

IO释放过程
ata_qc_free //判断ap->flags & ATA_FLAG_SAS_HOST不成立，就不释放
	ata_sas_free_tag 
scsi_error_handler
	scsi_restart_operations
		scsi_host_set_state(shost, SHOST_DEL)
#
ata_qc_complete or __ata_eh_qc_complete
	__ata_qc_complete
		ata_scsi_qc_complete // == qc->complete_fn(qc)
			ata_qc_done
				ata_qc_free
#
ata_eh_finish
	ata_eh_qc_complete
		__ata_eh_qc_complete
#第一个是进入EH会
sas_ata_task_done //关键ap->pflags & ATA_PFLAG_FROZEN, or test_bit(SAS_HA_FROZEN, &sas_ha->state)
	ata_qc_complete
ata_scsi_qc_new
		
ata_do_eh
ata_eh_autopsy
ata_eh_link_autopsy
ata_eh_analyze_ncq_er		ror
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
