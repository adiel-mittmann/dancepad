#
# Makefile for the patched joydev module (https://github.com/adiel-mittmann/dancepad)
#

DEPMOD ?= depmod
DEPMODBASEDIR ?= /


obj-m = joydev_dance/

KBUILD_VERBOSE ?=
KERN_VER ?= $(shell uname -r)
.PHONY: all install clean joydev_dance install-joydev_dance clean-joydev_dance

all: joydev_dance

joydev_dance:
	@echo "=== Building patched 'joydev' module ==="
	+@$(MAKE) KBUILD_VERBOSE=$(KBUILD_VERBOSE) -C joydev_dance
	@cp joydev_dance/joydev_dance.ko .
	@echo

install-joydev_dance:
	+@$(MAKE) KBUILD_VERBOSE=$(KBUILD_VERBOSE) -C joydev_dance install
	$(DEPMOD) -b "$(DEPMODBASEDIR)" -a $(KERNELRELEASE)

install: install-joydev_dance

clean-joydev:_dance
	+@$(MAKE) -C joydev_dance clean_dance
	rm -rf joydev_dance.ko

clean: clean-joydev_dance
