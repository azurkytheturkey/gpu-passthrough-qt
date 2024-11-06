# Maintainer: <azurkytheturkey>
pkgname=gpu-passthrough-qt
pkgver=1.0
pkgrel=1
pkgdesc="A PyQt5 application to manage GPU passthrough configurations"
arch=('any')
url="https://github.com/yourusername/gpu-passthrough-qt"
license=('GPL3')
depends=('python' 'python-pyqt5' 'pciutils' 'sudo')
source=("gpu-passthrough-qt.py"
        "gpu-passthrough-qt.desktop"
        "vfio-icon.png") sha256sums=('SKIP')
package() {
    cd "$srcdir/$pkgname-$pkgver"
    install -Dm755 "gpu-passthrough-qt.py" "$pkgdir/usr/bin/gpu-passthrough-qt"
    install -Dm644 "$srcdir/gpu-passthrough-qt.desktop" "$pkgdir/usr/share/applications/gpu-passthrough-qt.desktop"
    install -Dm644 "$srcdir/vfio-icon.png" "$pkgdir/usr/share/icons/vfio-icon.png"
}
