# Maintainer: Caleb Maclennan <caleb@alerque.com>
# Maintainer: Carlos Aznarán <caznaranl@uni.pe>
# Contributor: Luis Martinez <luis dot martinez at disroot dot org>

_pyname=deepl
pkgname=python-${_pyname}
pkgver=1.17.0
pkgrel=2
pkgdesc='Python library for the DeepL API'
arch=(any)
url="https://github.com/${_pyname}com/${_pyname}-python"
license=(MIT)
depends=(python-requests)
makedepends=(git 
             python-{build,installer,wheel}
             python-poetry-core)
optdepends=('python-keyring: for support storing API Key')
source=("git+$url.git?signed#tag=v$pkgver")
validpgpkeys=('D84A62C6CA4D3E3D6656083438FCACAAFD2C06B8') # Jan Ebbing <Jan.Ebbing@deepl.com>
sha256sums=('SKIP')

build() {
  cd "$_pyname-python"
  python -m build -wn
}

package() {
    cd "$_pyname-python"
    python -m installer -d ="$pkgdir" dist/*.whl
    install -Dm0644 -t "$pkgdir/usr/share/licenses/$pkgname/" LICENSE
}
