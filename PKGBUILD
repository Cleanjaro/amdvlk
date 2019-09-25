# Maintainer:  David Spink <yorper_protonmail.com>
# Contributor: Christoph Haag <haagch@studi.informatik.uni-stuttgart.de>
# Contributor: Laurent Carlier <lordheavym@gmail.com>

# Read default.xml at https://github.com/GPUOpen-Drivers/AMDVLK
# for the correct commit strings for the current stable build information.

pkgbase=amdvlk
pkgname=($pkgbase lib32-$pkgbase)
xgl_commit=2cb5558b94c5dc839e093cb439057a1802426c8e
pal_commit=88d997710b4e405f3a8e3fd60a38afee9e3e77e2
llpc_commit=ec210a78b6a280b00fb1765dd588c3970b6dc818
spvgen_commit=2f31d1170e8a12a66168b23235638c4bbc43ecdc
llvm_commit=9bc5dd4450a6361faf5c5661056a7ee494fad830
metrohash_commit=2b6fee002db6cc92345b02aeee963ebaaf4c0e2f
cwpack_commit=b601c88aeca7a7b08becb3d32709de383c8ee428
pkgver=2019.Q3.6
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
        MetroHash-$pkgver.tar.gz::https://github.com/GPUOpen-Drivers/MetroHash/archive/${metrohash_commit}.tar.gz
        CWPack-$pkgver.tar.gz::https://github.com/GPUOpen-Drivers/CWPack/archive/${cwpack_commit}.tar.gz)

sha256sums=('1970d7cdf31e564c7a98737442fc7ed3593da24fb7e4dabe26ec120017aa0538'
            'fb9a6a497f488a3d9682b51ec9d615199b2f6770446b9cec47bd6a7c81278269'
            '063f3446339a42b08128b4acb5b74e846a0bb5ebd9d3aae3feec5011a1797f1d'
            '88ae1c7d465e6313c324e2802ffa024fc3e1ed588ac4b48170c736fea9181e93'
            'cc946ad2835e502aca904c5f87802a2004eaed4729cb5c1dc29a5258d1c1e401'
            'efbde2752044ec74d522c160899491105dbc77bb8a08ff64c274d2b94a6916d1'
            'a5c1e77efd593853ee93a8f168fb7826baae52ca56df1d46f9ccde3d4e1f6c12'
            '58ca397f33d62bcfecaecd89eb4ad466a6c33e1c619e5cf742822074f1f7d664')

prepare() {
  ln -sf ${srcdir}/AMDVLK-v-${pkgver} ${srcdir}/AMDVLK
  ln -sf ${srcdir}/xgl-${xgl_commit} ${srcdir}/xgl
  ln -sf ${srcdir}/pal-${pal_commit} ${srcdir}/pal
  ln -sf ${srcdir}/llpc-${llpc_commit} ${srcdir}/llpc
  ln -sf ${srcdir}/spvgen-${spvgen_commit} ${srcdir}/spvgen
  ln -sf ${srcdir}/llvm-${llvm_commit} ${srcdir}/llvm
  ln -sf ${srcdir}/MetroHash-${metrohash_commit} ${srcdir}/MetroHash
  ln -sf ${srcdir}/CWPack-${cwpack_commit} ${srcdir}/CWPack
  touch amdPalSettings.cfg

  #remove -Werror to build with gcc9 
  sed -i "s/-Werror//g" $srcdir/pal/shared/gpuopen/cmake/AMD.cmake
}

build() {
  export CFLAGS="$CFLAGS -fno-plt -mno-avx"
  export CXXFLAGS="$CXXFLAGS -fno-plt -mno-avx"
  export LDFLAGS="$LDFLAGS -z now"

  # 64-bit
  msg "building 64-bit xgl..."
  cd xgl
  cmake -H. -Bbuilds/Release64 \
    -DCMAKE_BUILD_TYPE=Release \
    -DBUILD_XLIB_XRANDR_SUPPORT=On \
    -DBUILD_WAYLAND_SUPPORT=On \
    -DXGL_METROHASH_PATH=${srcdir}/MetroHash \
    -DXGL_CWPACK_PATH=${srcdir}/CWPack \
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
    -DXGL_METROHASH_PATH=${srcdir}/MetroHash \
    -DXGL_CWPACK_PATH=${srcdir}/CWPack \
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
