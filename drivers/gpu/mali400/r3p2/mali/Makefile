#
# Copyright (C) 2010-2012 ARM Limited. All rights reserved.
# 
# This program is free software and is provided to you under the terms of the GNU General Public License version 2
# as published by the Free Software Foundation, and any use by you of this program is subject to the terms of such GNU licence.
# 
# A copy of the licence is included with the program, and can also be obtained from Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301, USA.
#

USE_UMPV2=0
USING_PROFILING ?= 1
USING_INTERNAL_PROFILING ?= 0
MALI_DMA_BUF_MAP_ON_ATTACH ?= 1

# The Makefile sets up "arch" based on the CONFIG, creates the version info
# string and the __malidrv_build_info.c file, and then call the Linux build
# system to actually build the driver. After that point the Kbuild file takes
# over.

# set up defaults if not defined by the user
ARCH ?= arm

OSKOS=linux
FILES_PREFIX=

check_cc2 = \
	$(shell if $(1) -S -o /dev/null -xc /dev/null > /dev/null 2>&1; \
	then \
		echo "$(2)"; \
	else \
		echo "$(3)"; \
	fi ;)

# This conditional makefile exports the global definition ARM_INTERNAL_BUILD. Customer releases will not include arm_internal.mak
-include ../../../arm_internal.mak

# Give warning of old config parameters are used
ifneq ($(CONFIG),)
$(warning "You have specified the CONFIG variable which is no longer in used. Use TARGET_PLATFORM instead.")
endif

ifneq ($(CPU),)
$(warning "You have specified the CPU variable which is no longer in used. Use TARGET_PLATFORM instead.")
endif

# Include the mapping between TARGET_PLATFORM and KDIR + MALI_PLATFORM
-include MALI_CONFIGURATION
export KDIR ?= $(KDIR-$(TARGET_PLATFORM))
export MALI_PLATFORM ?= $(MALI_PLATFORM-$(TARGET_PLATFORM))

ifneq ($(TARGET_PLATFORM),)
ifeq ($(MALI_PLATFORM),)
$(error "Invalid TARGET_PLATFORM: $(TARGET_PLATFORM)")
endif
endif

# validate lookup result
ifeq ($(KDIR),)
$(error No KDIR found for platform $(TARGET_PLATFORM))
endif


ifeq ($(USING_UMP),1)
export CONFIG_MALI400_UMP=y
export EXTRA_DEFINES += -DCONFIG_MALI400_UMP=1
ifeq ($(USE_UMPV2),1)
UMP_SYMVERS_FILE ?= ../umpv2/Module.symvers
else
UMP_SYMVERS_FILE ?= ../ump/Module.symvers
endif
KBUILD_EXTRA_SYMBOLS = $(realpath $(UMP_SYMVERS_FILE))
$(warning $(KBUILD_EXTRA_SYMBOLS))
endif

# Define host system directory
KDIR-$(shell uname -m):=/lib/modules/$(shell uname -r)/build

include $(KDIR)/.config

ifeq ($(ARCH), arm)
# when compiling for ARM we're cross compiling
export CROSS_COMPILE ?= $(call check_cc2, arm-linux-gnueabi-gcc, arm-linux-gnueabi-, arm-none-linux-gnueabi-)
endif

# report detected/selected settings
ifdef ARM_INTERNAL_BUILD
$(warning TARGET_PLATFORM $(TARGET_PLATFORM))
$(warning KDIR $(KDIR))
$(warning MALI_PLATFORM $(MALI_PLATFORM))
endif

# Set up build config
export CONFIG_MALI400=m

ifneq ($(MALI_PLATFORM),)
export EXTRA_DEFINES += -DMALI_FAKE_PLATFORM_DEVICE=1
export MALI_PLATFORM_FILES = $(wildcard platform/$(MALI_PLATFORM)/*.c)
endif

ifeq ($(USING_PROFILING),1)
ifeq ($(CONFIG_TRACEPOINTS),)
$(warning CONFIG_TRACEPOINTS reqired for profiling)
else
export CONFIG_MALI400_PROFILING=y
export EXTRA_DEFINES += -DCONFIG_MALI400_PROFILING=1
ifeq ($(USING_INTERNAL_PROFILING),1)
export CONFIG_MALI400_INTERNAL_PROFILING=y
export EXTRA_DEFINES += -DCONFIG_MALI400_INTERNAL_PROFILING=1
endif
endif
endif

ifeq ($(MALI_DMA_BUF_MAP_ON_ATTACH),1)
export CONFIG_MALI_DMA_BUF_MAP_ON_ATTACH=y
export EXTRA_DEFINES += -DCONFIG_MALI_DMA_BUF_MAP_ON_ATTACH
endif

ifeq ($(MALI_SHARED_INTERRUPTS),1)
export CONFIG_MALI_SHARED_INTERRUPTS=y
export EXTRA_DEFINES += -DCONFIG_MALI_SHARED_INTERRUPTS
endif

ifneq ($(BUILD),release)
export CONFIG_MALI400_DEBUG=y
endif

all: $(UMP_SYMVERS_FILE)
	$(MAKE) ARCH=$(ARCH) -C $(KDIR) M=$(CURDIR) modules
	@rm $(FILES_PREFIX)__malidrv_build_info.c $(FILES_PREFIX)__malidrv_build_info.o

clean:
	$(MAKE) ARCH=$(ARCH) -C $(KDIR) M=$(CURDIR) clean

kernelrelease:
	$(MAKE) ARCH=$(ARCH) -C $(KDIR) kernelrelease

export CONFIG KBUILD_EXTRA_SYMBOLS
