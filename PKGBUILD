# Maintainer: leozeli <leozeli@users.noreply.github.com>
# Automatically updated by CI — do not edit pkgver/sha256sums manually.
pkgname=warp-terminal-oss
pkgver=r34.d0f045c
pkgrel=1
pkgdesc="Warp, the Rust-based terminal for developers and teams (OSS build)"
arch=('x86_64')
url="https://github.com/warpdotdev/warp"
license=('AGPL-3.0-only' 'MIT')
depends=(
	'curl'
	'default-cursors'
	'fontconfig'
	'libegl'
	'libx11'
	'libxcb'
	'libxcursor'
	'libxi'
	'libxkbcommon-x11'
	'opengl-driver'
	'xdg-utils'
	'zlib'
)
optdepends=(
	'adwaita-cursors: for if there is no default cursor installed'
	'zenity: for file dialogs in Gnome'
	'kdialog: for file dialogs in KDE'
	'org.freedesktop.secrets: for securely storing passwords'
)
options=('!strip')
source=("${pkgname}-${pkgver}-x86_64.tar.gz::https://github.com/leozeli/warp-terminal-oss-aur/releases/download/${pkgver}/${pkgname}-x86_64.tar.gz")
sha256sums=('43ffe294c63719e9f8b0b3e16c0412ba9850e845eba10eaa963dea0da2a0fad1')

package() {
	# The tarball contains opt/ and usr/share/ directory trees ready to install.
	cp -r "$srcdir/opt" "$pkgdir/"
	cp -r "$srcdir/usr" "$pkgdir/"

	# Install the wrapper shell script that respects user-supplied flags.
	local _install_dir="/opt/warpdotdev/$pkgname"
	install -Dm755 /dev/stdin "$pkgdir/usr/bin/$pkgname" <<-WRAPPER
		#!/bin/bash
		XDG_CONFIG_HOME=\${XDG_CONFIG_HOME:-~/.config}
		if [[ -f "\$XDG_CONFIG_HOME/$pkgname-flags.conf" ]]; then
		    WARP_USER_FLAGS="\$(grep -v '^#' "\$XDG_CONFIG_HOME/$pkgname-flags.conf")"
		fi
		exec "$_install_dir/warp-oss" \$WARP_USER_FLAGS "\$@"
	WRAPPER
}
