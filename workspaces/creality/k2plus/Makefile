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

CLOSEST_KLIPPER=01f90e8f28b5dccf803f92e9516fdc634083d1a1
outputs/stock-klipper.tar:
	curl --fail -L -'#' "https://github.com/Klipper3d/klipper/archive/$(CLOSEST_KLIPPER).tar.gz" -o $@

outputs/stock-klipper: outputs/stock-klipper.tar
	[[ ! -d $@ ]] || rm -rf $@
	mkdir $@
	tar -xz -f $< -C $@ \
		--strip-components 1 \
		--exclude 'src' \
		--exclude 'lib'
	touch $@

outputs/vendor-klipper: outputs/klipper.tar
	[[ ! -d $@ ]] || rm -rf $@
	mkdir $@
	tar -xf $< -C $@ \
		--exclude '*.pyc' \
		--exclude 'gcodes' \
		--exclude 'fw'
	touch $@

diffs: outputs/klipper.summary.txt outputs/klipper.diff

outputs/klipper.summary.txt: outputs/stock-klipper outputs/vendor-klipper
	(cd outputs && diff -q -r stock-klipper vendor-klipper || true) > $@

outputs/klipper.diff: outputs/stock-klipper outputs/vendor-klipper
	(cd outputs && diff -u -r -N stock-klipper vendor-klipper || true) > $@
