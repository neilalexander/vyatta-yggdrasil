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
RELEASE=0.1

DESTDIR=/opt/vyatta

TEMPLATECFGDIR=$DESTDIR/share/vyatta-cfg/template
TEMPLATEOPDIR=$DESTDIR/share/vyatta-op/template
SBINDIR=$DESTDIR/sbin/

BUILDDIR_YGG=/tmp/yggdrasil

PKGARCH?=mipsel
BUILDNUMBER=$(shell git rev-list v$(RELEASE)..HEAD --count 2>/dev/null || echo 0)
REPOBUILDNUMBER=$(shell cd $(BUILDDIR_YGG) && git rev-list HEAD --count 2>/dev/null || echo 0)
ifeq ($(BUILDNUMBER),0)
VERSION=$(RELEASE)-$(shell echo $(REPOBUILDNUMBER) | xargs printf "%04d")
else
VERSION=$(RELEASE).$(shell echo $(BUILDNUMBER) | bc)-$(shell echo $(REPOBUILDNUMBER) | xargs printf "%04d")
endif

BRANCH?='master'
ifneq ($(BRANCH),master)
BRANCHTAG=-$(BRANCH)
endif

all: cross

cross: clean build-cross package cleanpackage
native: clean build-native package cleanpackage

build-cross: yggdrasilclone yggdrasilbuild-cross yggdrasilstage
build-native: yggdrasilclone yggdrasilbuild-native yggdrasilstage

package:
	sed -i "s/^Architecture: .*/Architecture: $(PKGARCH)/" debian/control || \
		sed -i "" "s/^Architecture: .*/Architecture: $(PKGARCH)/" debian/control
	sed -i "s/^Version: .*/Version: $(VERSION)$(BRANCHTAG)/" debian/control || \
		sed -i "" "s/^Version: .*/Version: $(VERSION)$(BRANCHTAG)/" debian/control
	tar czvf ../data.tar.gz opt
	cd debian && tar czvf ../../control.tar.gz *
	echo 2.0 > ../debian-binary
	ar -r ../$(PACKAGE)-$(VERSION)$(BRANCHTAG)-$(PKGARCH).deb ../debian-binary ../control.tar.gz ../data.tar.gz

cleanpackage:
	rm ../debian-binary
	rm ../control.tar.gz
	rm ../data.tar.gz

yggdrasilclone:
	git clone -b $(BRANCH) https://github.com/Arceliar/yggdrasil-go $(BUILDDIR_YGG)
	touch /tmp/yggdrasil/yggdrasil

yggdrasilbuild-cross:
	cd $(BUILDDIR_YGG) && GOOS='linux' GOARCH='mipsle' ./build

yggdrasilbuild-native:
	cd $(BUILDDIR_YGG) && GOOS='linux' ./build

yggdrasilstage:
	cp $(BUILDDIR_YGG)/yggdrasil opt/vyatta/sbin/yggdrasil

clean:
	rm -rf sbin/yggdrasil
	rm -rf $(BUILDDIR_YGG)

distclean:	clean
