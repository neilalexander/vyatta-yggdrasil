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
BUILDNUMBER=$(shell git rev-list HEAD --count 2>/dev/null  | xargs printf "%04d" || echo 0)
REPOBUILDNUMBER=0.1.$(shell cd $(BUILDDIR_YGG) && git rev-list HEAD --count 2>/dev/null  | xargs printf "%04d" || echo 0)
VERSION=$(REPOBUILDNUMBER)-$(BUILDNUMBER)
BRANCH?=$(shell basename `git name-rev --name-only HEAD`)
ifneq ($(BRANCH),master)
BRANCHTAG=-$(BRANCH)
endif

all: cross

cross: clean build-cross package cleanpackage
native: clean build-native package cleanpackage

build-cross: yggdrasilclone yggdrasilbuild-cross yggdrasilstage
build-native: yggdrasilclone yggdrasilbuild-native yggdrasilstage

package:
	@sed -i "s/^Architecture: .*/Architecture: $(PKGARCH)/" debian/control || \
		sed -i "" "s/^Architecture: .*/Architecture: $(PKGARCH)/" debian/control
	@sed -i "s/^Version: .*/Version: $(VERSION)/" debian/control || \
		sed -i "" "s/^Version: .*/Version: $(VERSION)/" debian/control
	@if [ "$(BRANCH)" != "master" ]; then \
		sed -i "s/^Package: .*/Package: $(PACKAGE)$(BRANCHTAG)/" debian/control || \
			sed -i "" "s/^Package: .*/Package: $(PACKAGE)$(BRANCHTAG)/" debian/control; \
	fi
	@sed -i "/^Replaces/ d" debian/control || sed -i "" "/^Replaces/ d" debian/control
	@[ "$(BRANCH)" != "master" ] && (echo "Replaces: vyatta-yggdrasil" >> debian/control) || true
	tar czvf ../data.tar.gz opt
	cd debian && tar czvf ../../control.tar.gz *
	echo 2.0 > ../debian-binary
	ar -r ../$(PACKAGE)$(BRANCHTAG)-$(VERSION)-$(PKGARCH).deb ../debian-binary ../control.tar.gz ../data.tar.gz

cleanpackage:
	rm ../debian-binary
	rm ../control.tar.gz
	rm ../data.tar.gz

yggdrasilclone:
	@[ "$(BUILDDIR_YGG)" = "/tmp/yggdrasil" ] && git clone -b $(BRANCH) https://github.com/yggdrasil-network/yggdrasil-go $(BUILDDIR_YGG) || true
	touch $(BUILDDIR_YGG)/yggdrasil

yggdrasilbuild-cross:
	@[ "$(PKGARCH)" = "mipsel" ] && (cd $(BUILDDIR_YGG) && GOOS='linux' GOARCH='mipsle' ./build) || true
	@[ "$(PKGARCH)" = "mips" ] && (cd $(BUILDDIR_YGG) && GOOS='linux' GOARCH='mips64' ./build) || true

yggdrasilbuild-native:
	cd $(BUILDDIR_YGG) && GOOS='linux' ./build

yggdrasilstage:
	cp $(BUILDDIR_YGG)/yggdrasil opt/vyatta/sbin/yggdrasil

clean:
	rm -rf sbin/yggdrasil
	@[ "$(BUILDDIR_YGG)" = "/tmp/yggdrasil" ] && rm -rf $(BUILDDIR_YGG) || true

distclean:	clean
