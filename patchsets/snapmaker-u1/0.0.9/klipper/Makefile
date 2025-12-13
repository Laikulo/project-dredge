quilttree:
	[[ ! -d quilttree ]] || rm -rf quilttree
	mkdir quilttree
	curl --fail -L -'#' "https://github.com/Klipper3d/klipper/archive/$(file < stock.ref).tar.gz" | tar -xz --strip-components 1 -C quilttree
	ln -s ../patches quilttree/patches

worktree:
	git clone http://github.com/klipper3d/klipper.git --no-checkout --filter blob:none worktree
	(cd worktree && \
		git checkout $(file < stock.ref) && \
		git checkout -b vendor && \
		git tag fork-start && \
		git apply --index ../patches/0*.patch && \
		git commit -m "workdir_start" && \
		git apply ../patches/9999-*.patch && \
		git tag work-start )

rebuild-allother:
	(cd worktree && git stash show -p -u) | sed '/^diff --git/d;/^index/d;/^new file/d' > patches/9999-none-of-the-above.patch
