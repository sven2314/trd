# (C) Copyright 2020 - 2021 Xilinx, Inc.
# SPDX-License-Identifier: Apache-2.0

CP = cp -f

PWD = $(shell readlink -f .)

# the platform directory has to be an absolute path when passed to v++
PFM_DIR = $(PWD)/platforms
PFM_VER = 202210_1
BOOT_DIR= $(PWD)/platforms/boot
SD_DIR=$(PWD)/platforms/imagedir
# valid platforms / overlays
PFM_LIST = vck190_mipiRxSingle_hdmiTx vck190_mipiRxQuad_hdmiTx vck190_hdmiRx_hdmiTx
OVERLAY_LIST = filter2d xvdpu

PFM_XPFM = $(PFM_DIR)/xilinx_$(PFM)_$(PFM_VER)/$(PFM).xpfm
VITIS_FLOW_XSA=$(PFM_DIR)/vivado/$(PFM)/project/$(PFM)_emu.xsa
PLNX_DIR = petalinux/xilinx-vck190-base-trd
PLNX_WIC = $(PLNX_DIR)/images/linux/Image
TARGETS=
OVERLAY_DIR = overlays
OVERLAY_KERNEL_DIR = $(OVERLAY_DIR)/$(OVERLAY)/kernels
OVERLAY_APP_DIR = $(OVERLAY_DIR)/$(OVERLAY)/apps
OVERLAY_APP = $(shell ls $(OVERLAY_APP_DIR))
OVERLAY_XCLBIN = $(OVERLAY_KERNEL_DIR)/binary_container_1.xclbin
OVERLAY_XSA = $(OVERLAY_KERNEL_DIR)/binary_container_1.xsa

.PHONY: help
help:
	@echo 'Usage:'
	@echo ''
	@echo '  make sdcard PFM=<val> OVERLAY=<val> TARGET=<val> YES=<val>'
	@echo '    Generate an SD card wic image using PetaLinux.'
	@echo ''
	@echo '    Valid options for PFM: ${PFM_LIST}'
	@echo '    Valid options for OVERLAY: ${OVERLAY_LIST}'
	@echo '    YES: optional param to accept SDK changes if set to 1 (default: 0)'
	@echo ''
	@echo '  make overlay PFM=<val> OVERLAY=<val>'
	@echo '    Build the Vitis application overlay.'
	@echo ''
	@echo '    Valid options for PFM: ${PFM_LIST}'
	@echo '    Valid options for OVERLAY: ${OVERLAY_LIST}'
	@echo ''
	@echo '  make platform PFM=<val> JOBS=<n>'
	@echo '    Build the Vitis platform.'
	@echo ''
	@echo '  make xsa PFM=<val> JOBS=<n>'
	@echo '    Build the xsa for Vitis platform.'
	@echo ''
	@echo '    Valid options for PFM: ${PFM_LIST}'
	@echo '    JOBS: optional param to set number of synthesis jobs (default 8)'
	@echo ''
	@echo '  make docs'
	@echo '    Generate html documentation using sphinx'
	@echo ''
	@echo '  make clean'
	@echo '    Clean runs'
	@echo ''

.PHONY: all
all: sdcard

.PHONY: sdcard
sdcard: $(OVERLAY_XCLBIN)

$(PLNX_WIC): $(PLNX_DIR) $(VITIS_FLOW_XSA) 
	@echo 'Build PetaLinux wic image for $(PFM) - $(OVERLAY)'
	@cp $(VITIS_FLOW_XSA) $(PLNX_DIR)/project-spec/hw-description/system.xsa
	@mkdir -p $(PLNX_DIR)/images/linux
#@cp $(OVERLAY_XCLBIN) $(PLNX_DIR)/images/linux
	@cp -r $(OVERLAY_APP_DIR)/* $(PLNX_DIR)/project-spec/meta-base-trd/recipes-apps
	@echo 'CONFIG_$(OVERLAY_APP)' >> $(PLNX_DIR)/project-spec/meta-user/conf/user-rootfsconfig
	@echo 'CONFIG_$(OVERLAY_APP)=y' >> $(PLNX_DIR)/project-spec/configs/rootfs_config
#cp sanity.conf  $(PLNX_DIR)/components/yocto/layers/core/meta/conf/sanity.conf
#sed -i "s/INHERIT += "sanity"/#INHERIT += "sanity"/g" $(PLNX_DIR)/components/yocto/layers/core/meta/conf/sanity.conf
	-$(MAKE) -C $(PLNX_DIR) wic PFM=$(PFM)

$(PLNX_DIR):
	$(MAKE) -C petalinux project

.PHONY: overlay
overlay: $(OVERLAY_XCLBIN)
$(OVERLAY_XCLBIN): $(PFM_XPFM)
	@valid=0; \
	for o in $(OVERLAY_LIST); do \
	  if [ "$$o" = "$(OVERLAY)" ]; then \
	    valid=1; \
	    break; \
	  fi \
	done; \
	if [ "$$valid" -ne 1 ]; then \
	  echo 'Invalid parameter OVERLAY=$(OVERLAY). Choose one of: $(OVERLAY_LIST)'; \
	  exit 1; \
	fi; \
	echo 'Build $(OVERLAY) Vitis overlay using platform $(PFM)'; \
	$(MAKE) -C $(OVERLAY_KERNEL_DIR) all PLATFORM=$(PFM_XPFM) TARGET=$(TARGETS)

.PHONY: platform
platform: $(PFM_XPFM)
$(PFM_XPFM): $(VITIS_FLOW_XSA) $(PLNX_WIC)
	mkdir -p $(BOOT_DIR)
	mkdir -p $(SD_DIR)
	@cp $(PLNX_DIR)/images/linux/u-boot.elf $(BOOT_DIR)
	@cp $(PLNX_DIR)/images/linux/bl31.elf $(BOOT_DIR)
	@cp $(PLNX_DIR)/images/linux/system.dtb $(BOOT_DIR)
	@cp $(PLNX_DIR)/images/linux/boot.scr $(SD_DIR)
	@valid=0; \
	for p in $(PFM_LIST); do \
	  if [ "$$p" = "$(PFM)" ]; then \
	    valid=1; \
	    break; \
	  fi \
	done; \
	if [ "$$valid" -ne 1 ]; then \
	  echo 'Invalid parameter PFM=$(PFM). Choose one of: $(PFM_LIST)'; \
	  exit 1; \
	fi; \
	echo 'Create Vitis platform $(PFM)'; \
	$(MAKE) -C $(PFM_DIR) platform PLATFORM=$(PFM) VERSION=$(PFM_VER) TARGET=$(TARGETS)

.PHONY: xsa
xsa: $(VITIS_FLOW_XSA)
$(VITIS_FLOW_XSA):
	@valid=0; \
	for p in $(PFM_LIST); do \
	  if [ "$$p" = "$(PFM)" ]; then \
	    valid=1; \
	    break; \
	  fi \
	done; \
	if [ "$$valid" -ne 1 ]; then \
	  echo 'Invalid parameter PFM=$(PFM). Choose one of: $(PFM_LIST)'; \
	  exit 1; \
	fi; \
	echo 'generate xsa for coresponding $(PFM)'; \
	$(MAKE) -C $(PFM_DIR) xsa PLATFORM=$(PFM) VERSION=$(PFM_VER) TARGET=$(TARGETS)

.PHONY: docs
docs:
	$(MAKE) -C docs html

.PHONY: clean
clean:
	$(MAKE) -C docs clean
	$(MAKE) -C $(PLNX_DIR) clean
	$(foreach o, $(OVERLAY_LIST), $(MAKE) -C $(OVERLAY_DIR)/$(o)/kernels clean;)
	$(foreach p, $(PFM_LIST), $(MAKE) -C $(PFM_DIR) clean PLATFORM=$(p) VERSION=$(PFM_VER);)
