# Maintainer: Warp Team <dev@warp.dev>
pkgname=warp-terminal-oss
pkgver=r1.0
pkgrel=1
pkgdesc="Warp, the Rust-based terminal for developers and teams (OSS build)"
arch=('x86_64' 'aarch64')
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
makedepends=(
	'cargo'
	'git'
)
optdepends=(
	'adwaita-cursors: for if there is no default cursor installed'
	'zenity: for file dialogs in Gnome'
	'kdialog: for file dialogs in KDE'
	'org.freedesktop.secrets: for securely storing passwords'
)
# Don't perform additional stripping of binaries.
options=('!strip')
source=("git+https://github.com/warpdotdev/warp.git")
sha256sums=('SKIP')

pkgver() {
	cd "$srcdir/warp"
	printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

prepare() {
	cd "$srcdir/warp"
	# Pin to the toolchain version required by the project.
	local _toolchain
	_toolchain="$(grep 'channel' rust-toolchain.toml | sed 's/.*= "\(.*\)"/\1/')"
	rustup toolchain install "$_toolchain" --profile minimal --no-self-update
}

build() {
	cd "$srcdir/warp"

	local _toolchain
	_toolchain="$(grep 'channel' rust-toolchain.toml | sed 's/.*= "\(.*\)"/\1/')"

	# The OSS channel does not ship Sentry crash reporting.
	# Features mirror script/linux/bundle for channel=oss, artifact=app.
	cargo +"$_toolchain" build \
		--release \
		--bin warp-oss \
		--features "release_bundle,gui,nld_improvements"

	# Prepare bundled resources (skills, font fallbacks, etc.).
	# SKIP_SETTINGS_SCHEMA and NO_LICENSES avoid requiring cargo-about and the
	# generate_settings_schema helper binary as additional makedepends.
	SKIP_SETTINGS_SCHEMA=1 NO_LICENSES=1 \
		./script/prepare_bundled_resources \
		"$srcdir/resources-out" \
		"oss" \
		"release"
}

package() {
	cd "$srcdir/warp"

	local _install_dir="/opt/warpdotdev/$pkgname"
	local _bundle_id="dev.warp.WarpOss"

	# Install the main binary.
	install -Dm755 "target/release/warp-oss" \
		"$pkgdir/$_install_dir/warp-oss"

	# Install bundled resources (skills, fonts, etc.).
	if [[ -d "$srcdir/resources-out" ]]; then
		cp -r "$srcdir/resources-out" "$pkgdir/$_install_dir/resources"
	fi

	# Install the wrapper shell script that launches the application and
	# respects user-supplied flags from the config file.
	install -Dm755 /dev/stdin "$pkgdir/usr/bin/$pkgname" <<-EOF
		#!/bin/bash
		XDG_CONFIG_HOME=\${XDG_CONFIG_HOME:-~/.config}
		if [[ -f "\$XDG_CONFIG_HOME/$pkgname-flags.conf" ]]; then
		    WARP_USER_FLAGS="\$(grep -v '^#' "\$XDG_CONFIG_HOME/$pkgname-flags.conf")"
		fi
		exec "$_install_dir/warp-oss" \$WARP_USER_FLAGS "\$@"
	EOF

	# Install the .desktop file.
	install -Dm644 "app/channels/oss/$_bundle_id.desktop" \
		"$pkgdir/usr/share/applications/$_bundle_id.desktop"

	# Install icons at all available sizes.
	local _size
	for _size in 16x16 32x32 64x64 128x128 256x256 512x512; do
		local _src="app/channels/oss/icon/no-padding/$_size.png"
		if [[ -f "$_src" ]]; then
			install -Dm644 "$_src" \
				"$pkgdir/usr/share/icons/hicolor/$_size/apps/$_bundle_id.png"
		fi
	done

	# Install licenses.
	install -Dm644 LICENSE-AGPL "$pkgdir/usr/share/licenses/$pkgname/LICENSE-AGPL"
	install -Dm644 LICENSE-MIT  "$pkgdir/usr/share/licenses/$pkgname/LICENSE-MIT"
}
