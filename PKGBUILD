# Maintainer: Caleb Maclennan <caleb@alerque.com>
# Contributor: Eli Schwartz <eschwartz@archlinux.org>
# Contributor: Felix Yan <felixonmars@archlinux.org>
# Contributor: Michael Hansen <zrax0111 gmail com>

pkgbase=keybase
pkgname=(keybase kbfs keybase-gui)
pkgdesc='CLI tool for GPG with keybase.io'
pkgver=6.0.2
pkgrel=4
arch=(x86_64)
url=https://keybase.io
_url=https://github.com/keybase/client
license=(BSD)
# git is needed for yarn...
makedepends=(git
             go
             jq
             moreutils
             yarn)
_archive="$pkgbase-v$pkgver"
source=("$_url/releases/download/v$pkgver/$_archive.tar.xz"{,.sig}
        keybase-gui
        0001-Don-t-use-electron-to-build.patch
        https://github.com/keybase/client/commit/fbebbc9f1ba29e21ae6d3ee2edc21a7703d0019f.patch)
sha256sums=('22e5ae4d1f951ea9f3ffc3cb74de9b9f41b828b2c8a4e5cb6401de6fbccf497b'
            'SKIP'
            '7459a6846ff24c2bf7e6ab1ce31880829cf2692f23ffb3bf77e455f4de7ca34e'
            'dcff34a5676a929cd8f2cac1b0be98257c26a04e7935435c16e131bde486a1e4'
            '5a46d9433efb4244509d26fdf04340fb628de1d19a4dff6944510f9bba69d378')
validpgpkeys=('222B85B0F90BE2D24CFEB93F47484E50656D16C7') # Keybase.io Code Signing (v1) <code@keybase.io>

prepare() {
	ln -sf "${_archive/$pkgbase/client}" "$_archive"
	cd "$_archive"

	# update API certificate authority key
	patch -p1 -i ../fbebbc9f1ba29e21ae6d3ee2edc21a7703d0019f.patch

	export GOPATH="$srcdir/.gopath"
	mkdir -p "$GOPATH"/src/github.com/keybase
	ln -sf "$PWD" "$GOPATH"/src/github.com/keybase/client

	# Fix paths to run electron /path/to/app (or our minimal wrapper script).
	# Also wire up "hideWindow" when running as a service or via XDG autostart.
	sed -i 's@/opt/keybase/Keybase@/usr/bin/electron22 /usr/share/keybase-app@' \
		packaging/linux/systemd/keybase.gui.service

	# Don't let desktop launcher automatically start services on boot
	sed -i 's/run_keybase/keybase-gui/g' \
		packaging/linux/keybase.desktop

	patch -p1 -i ../0001-Don-t-use-electron-to-build.patch
	jq 'del(.devDependencies["electron"])' shared/package.json | sponge shared/package.json

	cd go
	go mod tidy
}

build() {
	cd "$_archive/go"

	# None of this should be necessary, but we duplicate the work of
	# makepkg.conf here since golang CGO can't be bothered to respect
	# standardized CFLAGS and we don't have native packaging integration for
	# either this or the go-specific flags. Must be done here to make sure
	# we're using debug or !buildflags.

	export CGO_CPPFLAGS="$CPPFLAGS"
	export CGO_CFLAGS="$CFLAGS"
	export CGO_CXXFLAGS="$CXXFLAGS"
	export CGO_LDFLAGS="$LDFLAGS"
	export GOFLAGS="-buildmode=pie -trimpath -ldflags=-linkmode=external -mod=readonly -modcacherw"

	export GOPATH="$srcdir/.gopath"
	go build -a -tags production -o ./bin/keybase github.com/keybase/client/go/keybase
	go build -a -tags production -o ./bin/kbnm github.com/keybase/client/go/kbnm
	go build -a -tags production -o ./bin/kbfsfuse github.com/keybase/client/go/kbfs/kbfsfuse
	go build -a -tags production -o ./bin/git-remote-keybase github.com/keybase/client/go/kbfs/kbfsgit/git-remote-keybase

	cd ../shared
	yarn install
	yarn run package --appVersion=$pkgver
}

package_keybase() {
	depends=(gnupg)
	optdepends=('kbfs: for the fuse-based fileystem and the encryption subcommand')

	cd "$_archive"

	install -Dm0755 -t "$pkgdir/usr/bin/" go/bin/{keybase,kbnm}

	# native messaging whitelists
	KBNM_INSTALL_ROOT=1 KBNM_INSTALL_OVERLAY="$pkgdir" "$pkgdir/usr/bin/kbnm" install
	# systemd activation
	install -Dm0644 -t "$pkgdir/usr/lib/systemd/user/" packaging/linux/systemd/keybase.service
	install -Dm0644 -t "$pkgdir/usr/share/licenses/$pkgname/" LICENSE
}

package_kbfs() {
	pkgdesc='The Keybase filesystem'
	url='https://keybase.io/docs/kbfs'
	depends=(fuse keybase)
	install=kbfs.install

	cd "$_archive"

	install -Dm0755 -t "$pkgdir/usr/bin/" go/bin/{kbfsfuse,git-remote-keybase}
	install -Dm0644 -t "$pkgdir/usr/share/licenses/$pkgname/" LICENSE
	# more systemd activation
	install -Dm0644 -t "$pkgdir/usr/lib/systemd/user/" packaging/linux/systemd/kbfs.service
}

package_keybase-gui() {
	pkgdesc='GUI frontend for GPG with keybase.io'
	depends=(electron22 keybase kbfs)

	cd "$_archive"

	install -Dm0644 -t "$pkgdir/usr/share/mime/packages/" packaging/linux/x-saltpack.xml
	install -Dm0644 -t "$pkgdir/usr/share/applications/" packaging/linux/keybase.desktop
	# more systemd activation
	install -Dm0644 -t "$pkgdir/usr/lib/systemd/user/" packaging/linux/systemd/keybase.gui.service
	# wrapper for electron
	install -Dm0755 -t "$pkgdir/usr/bin/" "$srcdir/keybase-gui"
	# Upstream launcher / wrapper / autostart service maker
	install -Dm0755 -t "$pkgdir/usr/bin/" packaging/linux/run_keybase

	install -Dm0644 -t "$pkgdir/usr/share/licenses/${pkgname}/" LICENSE

	for i in 16 32 128 256 512 ; do
		install -Dm0644 media/icons/Keybase.iconset/icon_${i}x${i}.png \
			"$pkgdir/usr/share/icons/hicolor/${i}x${i}/apps/keybase.png"
		install -Dm0644 media/icons/Saltpack.iconset/icon_${i}x${i}.png \
			"$pkgdir/usr/share/icons/hicolor/${i}x${i}/mimetypes/application-x-saltpack.png"
	done
	# ???
	install -Dm0644 media/icons/Keybase.iconset/icon_32x32@2x.png \
		"$pkgdir/usr/share/icons/hicolor/64x64/apps/keybase.png"
	install -Dm0644 media/icons/Keybase.iconset/icon_512x512@2x.png \
		"$pkgdir/usr/share/icons/hicolor/1024x1024/apps/keybase.png"

	# the app itself
	cd shared/desktop/build
	rm -rf desktop/sourcemaps/
	mkdir -p "$pkgdir/usr/share/keybase-app"
	cp -r * "$pkgdir/usr/share/keybase-app/"
}
