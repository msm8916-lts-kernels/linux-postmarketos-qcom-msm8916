# Maintainer: Verugin Gleb <gleb@glebmail.xyz>
# Kernel config based on: arch/arm64/configs/msm8916_defconfig

# Original authors Minecrell <minecrell@minecrell.net> and Nikita Travkin <nikita@trvn.ru>, based on pmos package

_flavor="postmarketos-qcom-msm8916"
pkgname=linux-$_flavor
pkgver=6.12.43
pkgrel=2
pkgdesc="Mainline kernel fork for Qualcomm MSM8909/MSM8916/MSM8939 devices"
arch="aarch64 armv7"
url="https://github.com/GlebVerugin/msm8916-linux"
license="GPL-2.0-only"
options="!strip !check !tracedeps
	pmb:cross-native
	pmb:kconfigcheck-community
	pmb:kconfigcheck-uefi
	pmb:kconfigcheck-usb
"
makedepends="
	bison
	findutils
	flex
	gmp-dev
	mpc1-dev
	mpfr-dev
	openssl-dev
	perl
	postmarketos-installkernel
	python3
"
replaces="linux-postmarketos-qcom-msm8909 linux-postmarketos-qcom-msm8939"

# Architecture
case "$CARCH" in
	aarch64) _carch="arm64" ;;
	arm*)    _carch="arm" ;;
esac

# Source
_tag=v${pkgver//_/-}-msm8916
source="
	$pkgname-$_tag.tar.gz::$url/archive/$_tag.tar.gz
	config-$_flavor.aarch64
	config-$_flavor.armv7
"
builddir="$srcdir/msm8916-linux-${_tag#v}"

prepare() {
	default_prepare
	cp "$srcdir/config-$_flavor.$CARCH" .config
}

build() {
	unset LDFLAGS
	make ARCH="$_carch" CC="${CC:-gcc}" \
		KBUILD_BUILD_VERSION=$((pkgrel + 1 ))
}

package() {
	mkdir -p "$pkgdir"/boot

	_install_targets="modules_install dtbs_install"

	if [ -e "$builddir/arch/$_carch/boot/vmlinuz.efi" ]; then
		# ZBOOT EFI decompressor for EFI booting
		install -Dm644 "$builddir/arch/$_carch/boot/vmlinuz.efi" \
			"$pkgdir/boot/linux.efi"

		# Old GZIP'd kernel image for boot.img compatibility
		install -Dm644 "$builddir/arch/$_carch/boot/vmlinuz" \
			"$pkgdir/boot/vmlinuz"
	elif [ "$_carch" = "arm64" ]; then
		echo "WARNING: CONFIG_ZBOOT not enabled!"
		install -Dm644 "$builddir/arch/$_carch/boot/Image.gz" \
			"$pkgdir/boot/vmlinuz"
	else
		_install_targets="zinstall modules_install dtbs_install"
	fi

	make $_install_targets \
		ARCH="$_carch" \
		INSTALL_PATH="$pkgdir"/boot \
		INSTALL_MOD_PATH="$pkgdir" \
		INSTALL_MOD_STRIP=1 \
		INSTALL_DTBS_PATH="$pkgdir"/boot/dtbs
	rm -f "$pkgdir"/lib/modules/*/build "$pkgdir"/lib/modules/*/source

	install -D "$builddir"/include/config/kernel.release \
		"$pkgdir"/usr/share/kernel/$_flavor/kernel.release
}

sha512sums="
5ae4fd53a67bb582515af0f2d946b32954236346acb740d6ea4afb2c6aec22fba7bf235a7f5a8bd60ae4718721a1c6a8b5849bf21fb14d16a34f96312ee6c498  linux-postmarketos-qcom-msm8916-v6.12.43-msm8916.tar.gz
2935a10166e0e8840b962ffe2062dcc2c49e925496e9269741f40d21c2e3b56e02292818f66a5192e3ee07b50ebf763eaf7223cb08044c4f63707859d83e8f3f  config-postmarketos-qcom-msm8916.aarch64
3f64dbece4fe6c7c343162075a6abb1ffa34ca5a2857971c2051b88516569d468c5626b9f981bb4c2439e7b6649e61fae9c9cd3246ed2e0052ef180a1fb06336  config-postmarketos-qcom-msm8916.armv7
"
