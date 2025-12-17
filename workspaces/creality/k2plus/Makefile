FW_VER=1.1.3.13

FW_OTA_FILENAME := CR0CN240110C10_ota_img_V$(FW_VER).img

OTA_CONTENTS=sw-description uboot kernel rootfs cpio_item_md5

all: outputs/klipper.tar outputs/moonraker.tar

%: %.url
	curl -L --fail -'#' -o "$@" "$(file < $<)"

$(foreach f,$(OTA_CONTENTS),artifacts/$(FW_OTA_FILENAME).d/$(f)) &: artifacts/$(FW_OTA_FILENAME)
	cpio -i -d --no-absolute-filenames -D artifacts/$(FW_OTA_FILENAME).d < "$<"

outputs/rootfs.tar: artifacts/$(FW_OTA_FILENAME).d/rootfs
	virt-tar-out -a "$<" / "$@"

outputs/moonraker.tar: artifacts/$(FW_OTA_FILENAME).d/rootfs
	virt-tar-out -a "$<" "/usr/share/moonraker"  "$@"

outputs/klipper.tar: artifacts/$(FW_OTA_FILENAME).d/rootfs
	virt-tar-out -a "$<" "/usr/share/klipper" "$@"

outputs/shadow: artifacts/$(FW_OTA_FILENAME).d/rootfs
	virt-cat -a "$<" /etc/shadow > "$@"
