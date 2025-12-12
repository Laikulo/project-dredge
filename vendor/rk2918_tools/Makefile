SOURCE_TARBALL=https://github.com/crosstyan/rk2918_tools/archive/refs/heads/master.tar.gz

bins := img_unpack afptool

all: $(foreach f,$(bins),bin/$f)

src/Makefile:
	curl -L --fail "$(SOURCE_TARBALL)" -'#' | tar --transform 's:^rk2918_tools-master:src:' -xvz


src/img_unpack: src/Makefile
	make -C src

bin/%: src/%
	install -m 755 "$^" bin

.PHONY: all
