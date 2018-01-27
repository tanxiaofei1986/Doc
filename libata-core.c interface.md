/**
 *	ata_do_set_mode - Program timings and issue SET FEATURES - XFER
 *	@link: link on which timings will be programmed
 *	@r_failed_dev: out parameter for failed device
 *
 *	Standard implementation of the function used to tune and set
 *	ATA device disk transfer mode (PIO3, UDMA6, etc.).  If
 *	ata_dev_set_mode() fails, pointer to the failing device is
 *	returned in @r_failed_dev.
 *
 *	LOCKING:
 *	PCI/etc. bus probe sem.
 *
 *	RETURNS:
 *	0 on success, negative errno otherwise
 */
int ata_do_set_mode(struct ata_link *link, struct ata_device **r_failed_dev)
ATA_CMD_SET_FEATURES
/* step 1: calculate xfer_mask */
Prepare for step2 and step3	
/* step 2: always set host PIO timings */ Only for PATA
ap->ops->set_piomode(ap, dev);
/* step 3: set host DMA timings */
ap->ops->set_dmamode(ap, dev);
sas_ata_set_dmamode 
	/* HISI_SAS not implimented. */
	--> i->dft->lldd_ata_set_dmamode
/* step 4: update devices' xfer mode */
ATA_CMD_SET_FEATURES
err_mask=0x40 (AC_ERR_SYSTEM)
ata_dev_set_mode
	--> ata_dev_set_xfermode
		--> ata_exec_internal
			--> ata_exec_internal_sg
				...
				/* no internal command while frozen */
				if (ap->pflags & ATA_PFLAG_FROZEN) {
					spin_unlock_irqrestore(ap->lock, flags);
					return AC_ERR_SYSTEM;
				}
				...



				
				
ata_eh_freeze_port
ata_port_freeze
ata_scsi_cmd_error_handler
__ata_port_freeze



sas_ata_eh
ata_scsi_cmd_error_handler

/**
 *	ata_exec_internal_sg - execute libata internal command
 *	@dev: Device to which the command is sent
 *	@tf: Taskfile registers for the command and the result
 *	@cdb: CDB for packet command
 *	@dma_dir: Data transfer direction of the command
 *	@sgl: sg list for the data buffer of the command
 *	@n_elem: Number of sg entries
 *	@timeout: Timeout in msecs (0 for default)
 *
 *	Executes libata internal command with timeout.  @tf contains
 *	command on entry and result on return.  Timeout and error
 *	conditions are reported via return value.  No recovery action
 *	is taken after a command times out.  It's caller's duty to
 *	clean up after timeout.
 *
 *	LOCKING:
 *	None.  Should be called with kernel context, might sleep.
 *
 *	RETURNS:
 *	Zero on success, AC_ERR_* mask on failure
 */
unsigned ata_exec_internal_sg(struct ata_device *dev,
			      struct ata_taskfile *tf, const u8 *cdb,
			      int dma_dir, struct scatterlist *sgl,
			      unsigned int n_elem, unsigned long timeout)
执行内部io，有设置超时，一般5s, 如果io出现异常，执行post_internal_cmd()会abort task
	/* do post_internal_cmd */
	if (ap->ops->post_internal_cmd)
		ap->ops->post_internal_cmd(qc);

ap->ops->post_internal_cmd(qc);
	--> sas_ata_post_internal
		--> sas_ata_internal_abort
			res = si->dft->lldd_abort_task(task);

/**
 *	ata_read_log_page - read a specific log page
 *	@dev: target device
 *	@log: log to read
 *	@page: page to read
 *	@buf: buffer to store read page
 *	@sectors: number of sectors to read
 *
 *	Read log page using READ_LOG_EXT command.
 *
 *	LOCKING:
 *	Kernel thread context (may sleep).
 *
 *	RETURNS:
 *	0 on success, AC_ERR_* mask otherwise.
 */
unsigned int ata_read_log_page(struct ata_device *dev, u8 log,
			       u8 page, void *buf, unsigned int sectors)
{
	unsigned long ap_flags = dev->link->ap->flags;
	struct ata_taskfile tf;
	unsigned int err_mask;
	bool dma = false;

	DPRINTK("read log page - log 0x%x, page 0x%x\n", log, page);

	/*
	 * Return error without actually issuing the command on controllers
	 * which e.g. lockup on a read log page.
	 */
	if (ap_flags & ATA_FLAG_NO_LOG_PAGE)
		return AC_ERR_DEV;

retry:
	ata_tf_init(dev, &tf);
	if (dev->dma_mode && ata_id_has_read_log_dma_ext(dev->id) &&
	    !(dev->horkage & ATA_HORKAGE_NO_DMA_LOG)) {
		tf.command = ATA_CMD_READ_LOG_DMA_EXT;
		tf.protocol = ATA_PROT_DMA;
		dma = true;
	} else {
		tf.command = ATA_CMD_READ_LOG_EXT;
		tf.protocol = ATA_PROT_PIO;
		dma = false;
	}
	tf.lbal = log;
	tf.lbam = page;
	tf.nsect = sectors;
	tf.hob_nsect = sectors >> 8;
	tf.flags |= ATA_TFLAG_ISADDR | ATA_TFLAG_LBA48 | ATA_TFLAG_DEVICE;

	err_mask = ata_exec_internal(dev, &tf, NULL, DMA_FROM_DEVICE,
				     buf, sectors * ATA_SECT_SIZE, 0);

	if (err_mask && dma) {
		dev->horkage |= ATA_HORKAGE_NO_DMA_LOG;
		ata_dev_warn(dev, "READ LOG DMA EXT failed, trying PIO\n");
		goto retry;
	}

	DPRINTK("EXIT, err_mask=%x\n", err_mask);
	return err_mask;
}