FIRMWARE_VERSION=0.0.9

FW_BASE := artifacts/$(FIRMWARE_VERSION)
UPDATE_IMG=$(FW_BASE)-upgrade.d/update.img

extract: outputs/moonraker.tar outputs/klipper.tar

UPFILE_NAMES=at32f403a.bin at32f415.bin MCU_DESC update.img UPFILE_BUILD_DATE UPFILE_VERSION


%: %.url
	curl --fail -l -'#' -o "$@" "$(file < $<)"

$(foreach f,$(UPFILE_NAMES),$(FW_BASE)-upgrade.d/$f) &: artifacts/$(FIRMWARE_VERSION)-upgrade.bin
	[[ -d $(dir $@) ]] || mkdir $(dir $@)
	$(UPFILE) unpack $< $(dir $@)

$(FW_BASE)-rk.d/update.img: $(FW_BASE)-upgrade.d/update.img 
	[[ -d $(dir $@) ]] || mkdir $(dir $@)
	$(IMG_UNPACK) $< $(dir $@)

$(FW_BASE)-afp.d/rootfs.img: $(FW_BASE)-rk.d/update.img
	[[ -d $(dir $@) ]] || mkdir $(dir $@)
	$(AFPTOOL) -unpack $< $(dir $@)

# TODO: rewrite these to use virt-tar-out
outputs/klipper.tar: $(FW_BASE)-afp.d/rootfs.img
	$(eval tempdir = $(shell mktemp -d))
	unsquashfs -dest "$(tempdir)" "$<" home/lava/klipper
	tar -cf "$@" -C"$(tempdir)"/home/lava klipper
	rm -rf "$(tempdir)"


outputs/moonraker.tar: $(FW_BASE)-afp.d/rootfs.img
	$(eval tempdir = $(shell mktemp -d))
	unsquashfs -dest "$(tempdir)" "$<" home/lava/moonraker
	tar -cf "$@" -C"$(tempdir)"/home/lava moonraker
	rm -rf "$(tempdir)"

outputs/rootfs.tar: $(FW_BASE)-afp.d/rootfs.img
	virt-tar-out -a $< / $@

CONTAINER=snapmaker-u1:$(FIRMWARE_VERSION)
container: outputs/rootfs.tar
	podman import --arch aarch64 \
		-c 'ENTRYPOINT ["/bin/sh", "-c"]' \
		-c 'CMD ["bash"]' \
		-c 'WORKDIR /home/lava' \
		-m "Project Dredge" \
		$< $(CONTAINER)
	

# TODO: Consider making a lib function for this
RK2918_DIR=../../../vendor/rk2918_tools
IMG_UNPACK=$(RK2918_DIR)/bin/img_unpack
AFPTOOL=$(RK2918_DIR)/bin/afptool
$(RK2918_DIR)/bin/img_unpack $(RK2918_DIR)/bin/afptool &:
	make -C $(RK2918_DIR)

UPFILE_DIR=../../../vendor/upfile
UPFILE=$(UPFILE_DIR)/bin/upfile
$(UPFILE)/bin/upfile:
	make -C $(UPFILE_DIR)

.PHONY: extract container
