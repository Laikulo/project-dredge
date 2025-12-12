FIRMWARE_VERSION=0.0.9

UPDATE_IMG=artifacts/$(FIRMWARE_VERSION)-upgrade.d/update.img
all: $(UPDATE_IMG)

UPFILE_NAMES=at32f403a.bin at32f415.bin MCU_DESC update.img UPFILE_BUILD_DATE UPFILE_VERSION



get-fw: artifacts/$(FIRMWARE_VERSION)-upgrade.bin

%: %.url
	curl --fail -l -'#' -o "$@" "$(file < $<)"

$(foreach f,$(UPFILE_NAMES),artifacts/%-upgrade.d/$f) &: artifacts/%-upgrade.bin
	[[ -d $(dir $@) ]] || mkdir $(dir $@)
	$(UPFILE) unpack $< $(dir $@)

artifacts/$(FIRMWARE_VERSION)-rk.d/update.img: artifacts/$(FIRMWARE_VERSION)-upgrade.d/update.img 
	[[ -d $(dir $@) ]] || mkdir $(dir $@)
	$(IMG_UNPACK) $< $(dir $@)

artifacts/$(FIRMWARE_VERSION)-afp.d/foobar.img: artifacts/$(FIRMWARE_VERSION)-rk.d/update.img
	[[ -d $(dir $@) ]] || mkdir $(dir $@)
	$(AFPTOOL) -unpack $< $(dir $@)



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
