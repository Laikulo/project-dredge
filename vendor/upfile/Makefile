SOURCE_TARBALL=https://github.com/paxx12/SnapmakerU1-Extended-Firmware/archive/refs/heads/main.tar.gz

all: bin/upfile

bin/upfile: src/upfile
	install -m 755 $^ $@

src/upfile: src/Makefile
	make -C src

src/Makefile:
	curl --fail -L -'#' "$(SOURCE_TARBALL)" | tar -xvz --transform 's:^SnapmakerU1-Extended-Firmware-main/tools/upfile:src:' SnapmakerU1-Extended-Firmware-main/tools/upfile 


.PHONY: upfile
