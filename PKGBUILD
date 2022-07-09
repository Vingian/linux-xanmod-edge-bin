_deb='https://github.com/xanmod/linux/releases/download/5.18.10-xanmod1/linux-image-5.18.10-xanmod1-x64v2_5.18.10-xanmod1-x64v2-0.git20220708.997a16a_amd64.deb'
_modules=('drivers/gpu/drm/i915/i915.ko')
pkgname=linux-xanmod-edge-bin
pkgbase=linux-xanmod-edge-bin
_kernel=$(echo "$_deb" | sed 's/^.*linux-image-\([^_]*\).*$/\1/')
pkgver=$(echo "$_kernel" | sed 's/^\([0-9.]*\).*$/\1/')
pkgrel=$(echo "$_kernel" | sed 's/^.*xanmod\([0-9.]*\).*$/\1/')
pkgdesc='Linux Xanmod - Latest Mainline (EDGE)'
url='http://www.xanmod.org/'
arch=(x86_64)
license=(GPL2)
options=('!strip')
depends=(coreutils kmod initramfs)
optdepends=('crda: to set the correct wireless channels of your country' 'linux-firmware: firmware images needed for some devices')
provides=(VIRTUALBOX-GUEST-MODULES WIREGUARD-MODULE KSMBD-MODULE NTFS3-MODULE)
if [ -z "$_modules" ] || [ -n "$_nopatchs" ]; then
  source=("$_deb")
  sha512sums=('SKIP')
else
  _kpatch=$(echo "$_deb" | sed 's/^.*\/download\/\([^\/]*\).*$/\1/')
  _major=$(echo "$_kernel" | sed 's/^\([0-9]*.[0-9]*\).*$/\1/')
  source=("$_deb" "headers.deb::${_deb//-image-/-headers-}" "https://cdn.kernel.org/pub/linux/kernel/v${_major%%.*}.x/linux-${_major}.tar.xz" "https://github.com/xanmod/linux/releases/download/${_kpatch}/patch-${_kpatch}.xz")
  sha512sums=('SKIP' 'SKIP' 'SKIP' 'SKIP')
  noextract=('headers.deb')
fi

prepare() {
  if [ -z "$_modules" ] || [ -n "$_nopatchs" ]; then return 0; fi

  mkdir -p headers
  bsdtar -xf headers.deb -C headers
  tar -xf headers/data.tar.xz -C "linux-$_major" --no-anchored --strip-components=4 Module.symvers

  local _compiler=gcc
  local _config=config_x86-64-v2

  pushd linux-$_major

  patch -Np1 -i "../patch-$_kpatch"

  local _patch
  for _patch in "$BUILDDIR/"*.patch
  do
    if [ -f "$_patch" ]
    then
      patch -Np1 < "$_patch"
    fi
  done

  scripts/setlocalversion --save-scmversion

  cp -vf "CONFIGS/xanmod/${_compiler}/${_config}" .config

  scripts/config --enable CONFIG_STACK_VALIDATION --enable CONFIG_IKCONFIG --enable CONFIG_IKCONFIG_PROC

  make LLVM=$_LLVM LLVM_IAS=$_LLVM olddefconfig

  make -s kernelrelease > version

  make prepare
  make modules_prepare
  local _module
  for _module in "${_modules[@]}"
  do
    make -C . M="${_module%/*}/"
  done

  popd
}

package() {
  pkgdesc='The Linux kernel and modules with Xanmod patches'

  tar -xf "$srcdir/data.tar.xz" -C "$pkgdir/"

  local modulesdir="usr/lib/modules/$_kernel"

  pushd "$pkgdir"
  rm -rf usr/*
  mv lib usr
  mv boot/vmlinuz-* "$modulesdir/vmlinuz"
  echo "${pkgbase%-*}" > "$modulesdir/pkgbase"
  find . -maxdepth 1 ! -name 'usr' -and ! -name '.' -exec rm -rf {} +

  if [ -z "$_nopatchs" ]; then
    local _module
    for _module in "${_modules[@]}"
    do
      mv -f "$modulesdir/kernel/$_module" "$modulesdir/kernel/${_module}.old"
      cp -f "$srcdir/linux-$_major/$_module" "$modulesdir/kernel/$_module"
    done
  fi

  popd
}
