# Mantainer: Sean Anderson <seanga2@gamil.com>
pkgname=proton
# Upstream version
_pkgver='3.16beta-20181031'
# Version for arch
#pkgver="${_pkgver//_/-}"
pkgver=3.16_20181031_beta
pkgrel=2
epoch=
pkgdesc="Compatibility tool for Steam Play based on Wine and additional components"
arch=('x86_64')
url="https://github.com/ValveSoftware/Proton/"
license=('BSD')
groups=()
depends=(
	'python2'
	'openvr-git'
	'wine-valve-git'
)
makedepends=(
	'vulkan-headers'
)
checkdepends=()
optdepends=()
provides=()
conflicts=()
replaces=()
backup=()
options=()
install=
changelog=
source=("https://github.com/ValveSoftware/Proton/archive/$pkgname-$_pkgver.tar.gz")
noextract=()
md5sums=('aa3ad16ebfc99296a2a280c58a692e9c')

prepare() {
	cd "Proton-$pkgname-$_pkgver"
	sed -i 's,wined3d-interop.h,wine/wined3d-interop.h,g' vrclient_x64/vrclient_x64/*
}

build() {
	cd "Proton-$pkgname-$_pkgver"

	export CXXFLAGS="$CXXFLAGS -Wno-attributes"
	export WINEMAKEFLAGS="--nosource-fix --nolower-include --nodlls --nomsvcrt --dll"
	export WINEMAKEFLAGS32="$WINEMAKEFLAGS --wine32"

	# The build script provided has so much cruft that it's easier to make everything manually
	mkdir -p build/lsteamclient.win32
	cp -a lsteamclient/* build/lsteamclient.win32
	cd build/lsteamclient.win32
	winemaker $WINEMAKEFLAGS32 -DSTEAM_API_EXPORTS .
	make
	cd ../..
	
	mkdir -p build/lsteamclient.win64
	cp -a lsteamclient/* build/lsteamclient.win64
	cd build/lsteamclient.win64
	winemaker $WINEMAKEFLAGS -DSTEAM_API_EXPORTS .
	make
	cd ../..

	# Currently depends on the custom bundled wine
	mkdir -p build/vrclient.win32
	cp -a vrclient_x64/* build/vrclient.win32
	rm -rf build/vrclient.win32/vrclient
	mv build/vrclient.win32/vrclient_x64 build/vrclient.win32/vrclient
	cd build/vrclient.win32/vrclient
	mv -u vrclient_x64.spec vrclient.spec
	winemaker -I.. $WINEMAKEFLAGS32 .
	CXXFLAGS="$CXXFLAGS --std=c++0x" make
	winebuild --dll --fake-module -E vrclient.spec -o vrclient.dll.fake
	cd ../../..

	mkdir -p build/vrclient.win64
	cp -a vrclient_x64/* build/vrclient.win64
	cd build/vrclient.win64/vrclient_x64
	winemaker -I.. $WINEMAKEFLAGS .
	CXXFLAGS="$CXXFLAGS --std=c++0x" make
	winebuild --dll --fake-module -E vrclient_x64.spec -o vrclient_x64.dll.fake
}

check() {
	cd "Proton-$pkgname-$_pkgver"
}

package() {
	cd "Proton-$pkgname-$_pkgver"
	
	install -d -m755 $pkgdir/usr/share/licenses/$pkgname
	install -m644 LICENSE $pkgdir/usr/share/licenses/$pkgname/LICENSE
	install -m644 LICENSE.proton $pkgdir/usr/share/licenses/$pkgname/LICENSE.proton
	install -m644 dist.LICENSE $pkgdir/usr/share/licenses/$pkgname/dist.LICENSE

	install -d -m755 $pkgdir/usr/share/doc/$pkgname
	install -m644 README.md $pkgdir/usr/share/doc/$pkgname/README.md

	install -d -m755 $pkgdir/usr/lib32/wine
	install -m755 build/lsteamclient.win32/lsteamclient.dll.so $pkgdir/usr/lib32/wine/
	install -m755 build/vrclient.win32/vrclient/vrclient.dll.so $pkgdir/usr/lib32/wine/

	install -d $pkgdir/usr/lib32/wine/fakedlls
	install -m644 build/vrclient.win32/vrclient/vrclient.dll.fake $pkgdir/usr/lib32/wine/fakedlls/vrclient.dll

	install -d -m755 $pkgdir/usr/lib/wine
	install -m755 build/lsteamclient.win64/lsteamclient.dll.so $pkgdir/usr/lib/wine/
	install -m755 build/vrclient.win64/vrclient_x64/vrclient_x64.dll.so $pkgdir/usr/lib/wine/

	install -d $pkgdir/usr/lib/wine/fakedlls
	install -m644 build/vrclient.win64/vrclient_x64/vrclient_x64.dll.fake $pkgdir/usr/lib/wine/fakedlls/vrclient_x64.dll
}
