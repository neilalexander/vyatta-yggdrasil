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

PACKAGE=vyatta-yggdrasil
DESTDIR=/opt/vyatta

TEMPLATECFGDIR=$DESTDIR/share/vyatta-cfg/template
TEMPLATEOPDIR=$DESTDIR/share/vyatta-op/template
SBINDIR=$DESTDIR/sbin/

BUILDDIR_YGG?=/tmp/yggdrasil

PKGARCH?=mipsel

# cd $(BUILDDIR_YGG) && sh contrib/semver/version.sh | cut -c 2-
# cd $(BUILDDIR_YGG) && sh contrib/semver/name.sh

all: cross

cross: clean build-cross package cleanpackage
native: clean build-native package cleanpackage

build-cross: yggdrasilclone yggdrasilbuild-cross yggdrasilconfbuild-cross yggdrasilstage
build-native: yggdrasilclone yggdrasilbuild-native yggdrasilconfbuild-native yggdrasilstage

package:
	@sed -i "s/^Architecture: .*/Architecture: $(PKGARCH)/" debian/control || \
		sed -i "" "s/^Architecture: .*/Architecture: $(PKGARCH)/" debian/control
	@sed -i "s/^Version: .*/Version: $(shell cd $(BUILDDIR_YGG) && sh contrib/semver/version.sh | cut -c 2-)/" debian/control || \
		sed -i "" "s/^Version: .*/Version: $(shell cd $(BUILDDIR_YGG) && sh contrib/semver/version.sh | cut -c 2-)/" debian/control
	@if [ "$(BRANCH)" != "master" ]; then \
		sed -i "s/^Package: .*/Package: vyatta-$(shell cd $(BUILDDIR_YGG) && sh contrib/semver/name.sh)/" debian/control || \
			sed -i "" "s/^Package: .*/Package: vyatta-$(shell cd $(BUILDDIR_YGG) && sh contrib/semver/name.sh)/" debian/control; \
	else \
		sed -i "s/^Package: .*/Package: vyatta-yggdrasil/" debian/control || \
			sed -i "" "s/^Package: .*/Package: vyatta-yggdrasil/" debian/control; \
	fi
	@sed -i "/^Replaces/ d" debian/control || sed -i "" "/^Replaces/ d" debian/control
	@[ "$(BRANCH)" != "master" ] && (echo "Replaces: vyatta-yggdrasil" >> debian/control) || true
	@cat debian/control
	tar czvf ../data.tar.gz opt
	cd debian && tar czvf ../../control.tar.gz *
	echo 2.0 > ../debian-binary
	ar -r ../vyatta-$(shell cd $(BUILDDIR_YGG) && sh contrib/semver/name.sh)-$(shell cd $(BUILDDIR_YGG) && sh contrib/semver/version.sh | cut -c 2-)-$(PKGARCH).deb ../debian-binary ../control.tar.gz ../data.tar.gz

cleanpackage:
	rm ../debian-binary
	rm ../control.tar.gz
	rm ../data.tar.gz

yggdrasilclone:
	@[ "$(BUILDDIR_YGG)" = "/tmp/yggdrasil" ] && git clone -b $(BRANCH) https://github.com/yggdrasil-network/yggdrasil-go $(BUILDDIR_YGG) || true
	touch $(BUILDDIR_YGG)/yggdrasil
	touch $(BUILDDIR_YGG)/yggdrasilctl
	touch $(BUILDDIR_YGG)/yggdrasilconf

yggdrasilbuild-cross:
	@[ "$(PKGARCH)" = "mipsel" ] && (cd $(BUILDDIR_YGG) && GOOS='linux' GOARCH='mipsle' ./build) || true
	@[ "$(PKGARCH)" = "mips" ] && (cd $(BUILDDIR_YGG) && GOOS='linux' GOARCH='mips64' ./build) || true

yggdrasilbuild-native:
	cd $(BUILDDIR_YGG) && GOOS='linux' ./build

yggdrasilconfbuild-cross:
	@[ "$(PKGARCH)" = "mipsel" ] && (cd $(BUILDDIR_YGG) && GOOS='linux' GOARCH='mipsle' GOPATH=$(BUILDDIR_YGG) go build contrib/config/yggdrasilconf.go) || true
	@[ "$(PKGARCH)" = "mips" ] && (cd $(BUILDDIR_YGG) && GOOS='linux' GOARCH='mips64' GOPATH=$(BUILDDIR_YGG) go build contrib/config/yggdrasilconf.go) || true

yggdrasilconfbuild-native:
	cd $(BUILDDIR_YGG) && GOOS='linux' GOARCH='mipsle' GOPATH=$(BUILDDIR_YGG) go build contrib/config/yggdrasilconf.go || true


yggdrasilstage:
	cp $(BUILDDIR_YGG)/yggdrasil opt/vyatta/sbin/yggdrasil
	cp $(BUILDDIR_YGG)/yggdrasilctl opt/vyatta/sbin/yggdrasilctl
	cp $(BUILDDIR_YGG)/yggdrasilconf opt/vyatta/sbin/yggdrasilconf

clean:
	rm -rf sbin/yggdrasil
	@[ "$(BUILDDIR_YGG)" = "/tmp/yggdrasil" ] && rm -rf $(BUILDDIR_YGG) || true

distclean:	clean
