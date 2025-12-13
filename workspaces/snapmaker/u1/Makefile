FIRMWARE_VERSION=0.0.9

FW_BASE := artifacts/$(FIRMWARE_VERSION)
UPDATE_IMG=$(FW_BASE)-upgrade.d/update.img

all: extract diffs

extract: outputs/moonraker.tar outputs/klipper.tar
diffs: outputs/klipper/summarydiff.txt outputs/klipper/full.diff

UPFILE_NAMES=at32f403a.bin at32f415.bin MCU_DESC update.img UPFILE_BUILD_DATE UPFILE_VERSION
CLOSEST_KLIPPER=248d3dbf8bcda69a51e3a303862bbcc64a32119d


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

outputs/klipper.tar: $(FW_BASE)-afp.d/rootfs.img
	virt-tar-out -a $< /home/lava/klipper $@

outputs/moonraker.tar: $(FW_BASE)-afp.d/rootfs.img
	virt-tar-out -a $< /home/lava/moonraker $@

outputs/rootfs.tar: $(FW_BASE)-afp.d/rootfs.img
	virt-tar-out -a $< / $@

outputs/klipper:
	mkdir $@

outputs/moonraker:
	mkdir $@

artifacts/stock-klipper.tar:
	curl --fail -L -'#' "https://github.com/Klipper3d/klipper/archive/$(CLOSEST_KLIPPER).tar.gz" -o $@

outputs/klipper/stock: artifacts/stock-klipper.tar | outputs/klipper
	[[ ! -d $@ ]] || rm -rf $@
	mkdir $@
	tar -xz -f $< -C $@ \
		--strip-components 1 \
		--exclude COPYING \
		--exclude Makefile \
		--exclude README.md \
		--exclude lib \
		--exclude src \
		--exclude test \
		--exclude .github \
		--exclude .gitignore \
		--exclude docs/\* \
		--exclude config/\* \
		--exclude klippy/chelper/\*.c \
		--exclude klippy/chelper/\*.h
	touch $@

outputs/klipper/vendor: outputs/klipper.tar | outputs/klipper
	[[ ! -d $@ ]] || rm -rf $@
	mkdir $@
	tar -xf $< -C $@ \
		--exclude __pycache__ \
		--exclude klippy/chelper/\*.so
	touch $@

outputs/klipper/summarydiff.txt: outputs/klipper/stock outputs/klipper/vendor
	(cd outputs/klipper && diff -q -r stock vendor || true) > $@

outputs/klipper/full.diff: outputs/klipper/stock outputs/klipper/vendor
	(cd outputs/klipper && diff -u -r -N stock vendor || true) > $@

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

.PHONY: extract container all diffs
