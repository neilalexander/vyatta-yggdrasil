#!/usr/bin/make -f
#
# vyatta-yggdrasil - A yggdrasil package for Ubiquiti EdgeOS, VyOS and other
# Vyatta-based systems.
# Copyright (C) 2015 Neil Alexander T.
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.
#

PKGNAME?=vyatta-$(shell cd $(BUILDDIR_YGG) && sh contrib/semver/name.sh --bare)
PKGARCH?=mipsel
PKGVERSION?=$(shell cd $(BUILDDIR_YGG) && sh contrib/semver/version.sh --bare)

BUILDDIR_YGG?=/tmp/yggdrasil

# cd $(BUILDDIR_YGG) && sh contrib/semver/version.sh | cut -c 2-
# cd $(BUILDDIR_YGG) && sh contrib/semver/name.sh

all: cross

cross: clean build-cross package cleanpackage
native: clean build-native package cleanpackage

build-cross: yggdrasilclone yggdrasilbuild-cross yggdrasilstage
build-native: yggdrasilclone yggdrasilbuild-native yggdrasilstage

package:
	@echo "Packaging $(PKGNAME) version $(PKGVERSION) for $(PKGARCH)..."
	@sed -i "s/^Architecture: .*/Architecture: $(PKGARCH)/" debian/control || \
		sed -i "" "s/^Architecture: .*/Architecture: $(PKGARCH)/" debian/control
	@sed -i "s/^Version: .*/Version: $(PKGVERSION)/" debian/control || \
		sed -i "" "s/^Version: .*/Version: $(PKGVERSION)/" debian/control
	@sed -i "s/^Package: .*/Package: $(PKGNAME)/" debian/control || \
		sed -i "" "s/^Package: .*/Package: $(PKGNAME)/" debian/control
	@cat debian/control
	tar czvf ../data.tar.gz opt usr
	cd debian && tar czvf ../../control.tar.gz *
	echo 2.0 > ../debian-binary
	ar -r ../$(PKGNAME)-$(PKGVERSION)-$(PKGARCH).deb ../debian-binary ../control.tar.gz ../data.tar.gz

cleanpackage:
	rm ../debian-binary
	rm ../control.tar.gz
	rm ../data.tar.gz

yggdrasilclone:
	@[ "$(BUILDDIR_YGG)" = "/tmp/yggdrasil" ] && git clone -b $(BRANCH) https://github.com/yggdrasil-network/yggdrasil-go $(BUILDDIR_YGG) || true

yggdrasilbuild-cross:
	@[ "$(PKGARCH)" = "mipsel" ] && (cd $(BUILDDIR_YGG) && GOOS='linux' GOARCH='mipsle' GOMIPS='softfloat' ./build) || true
	@[ "$(PKGARCH)" = "mips" ] && (cd $(BUILDDIR_YGG) && GOOS='linux' GOARCH='mips64' ./build) || true
	@[ "$(PKGARCH)" = "amd64" ] && (cd $(BUILDDIR_YGG) && GOOS='linux' GOARCH='amd64' ./build) || true
	@[ "$(PKGARCH)" = "i386" ] && (cd $(BUILDDIR_YGG) && GOOS='linux' GOARCH='386' ./build) || true

yggdrasilbuild-native:
	cd $(BUILDDIR_YGG) && GOOS='linux' ./build

yggdrasilstage:
	mkdir -p usr/local/bin
	cp $(BUILDDIR_YGG)/yggdrasil usr/local/bin/yggdrasil
	cp $(BUILDDIR_YGG)/yggdrasilctl usr/local/bin/yggdrasilctl

clean:
	rm -rf sbin/yggdrasil
	@[ "$(BUILDDIR_YGG)" = "/tmp/yggdrasil" ] && rm -rf $(BUILDDIR_YGG) || true

distclean:	clean
