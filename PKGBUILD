# Maintainer:  David Spink <yorper_protonmail.com>
# Contributor: Christoph Haag <haagch@studi.informatik.uni-stuttgart.de>
# Contributor: Laurent Carlier <lordheavym@gmail.com>

# Read default.xml at https://github.com/GPUOpen-Drivers/AMDVLK
# for the correct commit strings for the current stable build information.

pkgbase=amdvlk
pkgname=($pkgbase lib32-$pkgbase)
xgl_commit=331558e93794068a786bf699d3fe23bb11bac021
pal_commit=68b57dba33a4d922e8f1ef1b3781c2f659ffbd1c
llpc_commit=4fa48ef1cf0f81eafdb56df91c2f2180d4865101
spvgen_commit=2f31d1170e8a12a66168b23235638c4bbc43ecdc
llvm_commit=9bc5dd4450a6361faf5c5661056a7ee494fad830
metrohash_commit=2b6fee002db6cc92345b02aeee963ebaaf4c0e2f
pkgver=2019.Q3.5
pkgrel=1
pkgdesc="AMD's standalone Vulkan driver"
arch=(x86_64)
url="https://github.com/GPUOpen-Drivers"
license=('MIT')
makedepends=('cmake' 'dri2proto' 'libdrm' 'lib32-libdrm' 'libxml2' 'lib32-libxml2'
             'libxrandr' 'ninja' 'python' 'wayland' 'xorg-server-devel')
source=(AMDVLK-$pkgver.tar.gz::https://github.com/GPUOpen-Drivers/AMDVLK/archive/v-${pkgver}.tar.gz
        xgl-$pkgver.tar.gz::https://github.com/GPUOpen-Drivers/xgl/archive/${xgl_commit}.tar.gz
        pal-$pkgver.tar.gz::https://github.com/GPUOpen-Drivers/pal/archive/${pal_commit}.tar.gz
        llpc-$pkgver.tar.gz::https://github.com/GPUOpen-Drivers/llpc/archive/${llpc_commit}.tar.gz
        spvgen-$pkgver.tar.gz::https://github.com/GPUOpen-Drivers/spvgen/archive/${spvgen_commit}.tar.gz
        llvm-$pkgver.tar.gz::https://github.com/GPUOpen-Drivers/llvm/archive/${llvm_commit}.tar.gz
        MetroHash-$pkgver.tar.gz::https://github.com/GPUOpen-Drivers/metrohash/archive/${metrohash_commit}.tar.gz)

sha256sums=('383d43ddcff3295bb8dc85bce2a376fbde9f2aa3535be9e4dbf67f745c40ff41'
            '939a2cf69d840e01da8b3e69f5ffe1f852f9d2919cdbc8aa4ade7cff7ac56906'
            '7648ca7761b588b6025f8fe16fcf4216bf7e1fe53c6568377f5cca98feca9627'
            'abe541ef6cd4fa3ca1eaab52412caa29e2adedec0fab40894aef88d33deee584'
            'cc946ad2835e502aca904c5f87802a2004eaed4729cb5c1dc29a5258d1c1e401'
            'efbde2752044ec74d522c160899491105dbc77bb8a08ff64c274d2b94a6916d1'
            'e8ecf026584dd953e39c3abba2eb04d28b28ed4577482ee70265f0d421fef398')

prepare() {
  ln -sf ${srcdir}/AMDVLK-v-${pkgver} ${srcdir}/AMDVLK
  ln -sf ${srcdir}/xgl-${xgl_commit} ${srcdir}/xgl
  ln -sf ${srcdir}/pal-${pal_commit} ${srcdir}/pal
  ln -sf ${srcdir}/llpc-${llpc_commit} ${srcdir}/llpc
  ln -sf ${srcdir}/spvgen-${spvgen_commit} ${srcdir}/spvgen
  ln -sf ${srcdir}/llvm-${llvm_commit} ${srcdir}/llvm
  ln -sf ${srcdir}/MetroHash-${metrohash_commit} ${srcdir}/metrohash
  touch amdPalSettings.cfg

  #remove -Werror to build with gcc9 
  sed -i "s/-Werror//g" $srcdir/pal/shared/gpuopen/cmake/AMD.cmake
}

build() {
  export CFLAGS="$CFLAGS -fno-plt"
  export CXXFLAGS="$CXXFLAGS -fno-plt"
  export LDFLAGS="$LDFLAGS -z now"

  # 64-bit
  msg "building 64-bit xgl..."
  cd xgl
  cmake -H. -Bbuilds/Release64 \
    -DCMAKE_BUILD_TYPE=Release \
    -DBUILD_XLIB_XRANDR_SUPPORT=On \
    -DBUILD_WAYLAND_SUPPORT=On \
    -DXGL_METROHASH_PATH=${srcdir}/metrohash \
    -G Ninja

  ninja -C builds/Release64
  msg "building 64-bit xgl finished!"

  # 32-bit
  msg "building 32-bit xgl..."
  export PKG_CONFIG_PATH="/usr/lib32/pkgconfig"
  cmake -H. -Bbuilds/Release \
    -DCMAKE_BUILD_TYPE=Release \
    -DBUILD_XLIB_XRANDR_SUPPORT=On \
    -DBUILD_WAYLAND_SUPPORT=On \
    -DCMAKE_C_FLAGS=-m32 \
    -DCMAKE_CXX_FLAGS=-m32 \
    -DLLVM_TARGET_ARCH:STRING=i686 \
    -DLLVM_DEFAULT_TARGET_TRIPLE="i686-pc-linux-gnu" \
    -DXGL_METROHASH_PATH=${srcdir}/metrohash \
    -G Ninja

  ninja -C builds/Release
  msg "building 32-bit xgl finished!"
}

package_amdvlk() {
  depends=('vulkan-icd-loader')
  provides=('vulkan-amdvlk' 'vulkan-driver')
  conflicts=('vulkan-amdvlk')

  install -m755 -d "${pkgdir}"/usr/lib
  install -m755 -d "${pkgdir}"/usr/share/vulkan/icd.d
  install -m755 -d "${pkgdir}"/usr/share/licenses/amdvlk
  install -m755 -d "${pkgdir}"/etc/amd

  install xgl/builds/Release64/icd/amdvlk64.so "${pkgdir}"/usr/lib/
  install AMDVLK/json/Redhat/amd_icd64.json "${pkgdir}"/usr/share/vulkan/icd.d/
  install AMDVLK/LICENSE.txt "${pkgdir}"/usr/share/licenses/amdvlk/
  install amdPalSettings.cfg "${pkgdir}"/etc/amd/

  sed -i "s/\/lib64/\/lib/g" "${pkgdir}"/usr/share/vulkan/icd.d/amd_icd64.json
}

package_lib32-amdvlk() {
  pkgdesc="AMD's standalone Vulkan driver (32-bit)"
  depends=('lib32-vulkan-icd-loader')
  provides=('lib32-vulkan-amdvlk' 'lib32-vulkan-driver')
  conflicts=('lib32-vulkan-amdvlk')

  install -m755 -d "${pkgdir}"/usr/lib32
  install -m755 -d "${pkgdir}"/usr/share/vulkan/icd.d
  install -m755 -d "${pkgdir}"/usr/share/licenses/lib32-amdvlk

  install xgl/builds/Release/icd/amdvlk32.so "${pkgdir}"/usr/lib32/
  install AMDVLK/json/Redhat/amd_icd32.json "${pkgdir}"/usr/share/vulkan/icd.d/
  install AMDVLK/LICENSE.txt "${pkgdir}"/usr/share/licenses/lib32-amdvlk/
  
  sed -i "s/\/lib/\/lib32/g" "${pkgdir}"/usr/share/vulkan/icd.d/amd_icd32.json
}
