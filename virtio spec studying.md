# VirtIO spec studying notes

## 1. Introduction

describe the "virtio" family of devices, in virtual environments.
straightforward
efficent
standard
extensible

## 2. Basic Facilities of a Virtio Device
Each device consists of the following parts:
* Device status field
* Feature bits
* Device Configuration space
* One or more virtqueues

### 2.1 Device status field

virtio_add_status() --> vp_set_status() --> virtio_pci_common_cfg.device_status

Driver: 
1). update device status, and must not clear a device status bit.
2). If FAILED bit set, the driver must reset the device before attempting to re-initialize.

### 2.2 Feature Bits
### 2.3 Device Configuration Space
### 2.4 Virtqueues
```
struct vring {
	unsigned int num;
	struct vring_desc *desc;
	struct vring_avail *avail; //written by driver, and read by device
	struct vring_used *used; //written by device, and read by driver
};
struct vring_virtqueue {
	...
	struct vring vring;
	...
};
```