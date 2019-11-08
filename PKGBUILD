# Maintainer:  David Spink <yorper_protonmail.com>
# Contributor: Jonathon Fernyhough <jonathon_manjaro.org>
# Contributor: Christoph Haag <haagch@studi.informatik.uni-stuttgart.de>
# Contributor: Laurent Carlier <lordheavym@gmail.com>

# Read default.xml at https://github.com/GPUOpen-Drivers/AMDVLK
# for the correct commit strings for the current stable build information.

pkgbase=amdvlk
pkgname=($pkgbase lib32-$pkgbase)
xgl_commit=ef2f9c22455a79eea10c14e44fe371c003322ba1
pal_commit=76c5b997630e558158dbdd8ca24a120071068631
llpc_commit=2ba438ad4c592229a4ae14bd4368e318ac4f81eb
spvgen_commit=f1bc2ba988273c3724afffe72fe9cd933a022ce7
llvmproject_commit=2c15e55bc4b7171d6fa4bbb0cd9265bb8ad999b8
metrohash_commit=2b6fee002db6cc92345b02aeee963ebaaf4c0e2f
cwpack_commit=b601c88aeca7a7b08becb3d32709de383c8ee428
pkgver=2019.Q4.2
pkgrel=1
pkgdesc="AMD's standalone Vulkan driver"
arch=(x86_64)
url="https://github.com/GPUOpen-Drivers"
license=('MIT')
makedepends=('cmake' 'dri2proto' 'libdrm' 'lib32-libdrm' 'libxml2' 'lib32-libxml2'
             'libxrandr' 'ninja' 'python' 'wayland' 'xorg-server-devel')
source=(AMDVLK-$pkgver.tar.gz::https://github.com/GPUOpen-Drivers/AMDVLK/archive/v-${pkgver}.tar.gz
        xgl-${xgl_commit}.tar.gz::https://github.com/GPUOpen-Drivers/xgl/archive/${xgl_commit}.tar.gz
        pal-${pal_commit}.tar.gz::https://github.com/GPUOpen-Drivers/pal/archive/${pal_commit}.tar.gz
        llpc-${llpc_commit}.tar.gz::https://github.com/GPUOpen-Drivers/llpc/archive/${llpc_commit}.tar.gz
        spvgen-${spvgen_commit}.tar.gz::https://github.com/GPUOpen-Drivers/spvgen/archive/${spvgen_commit}.tar.gz
        llvm-project-${llvmproject_commit}.tar.gz::https://github.com/GPUOpen-Drivers/llvm-project/archive/${llvmproject_commit}.tar.gz
        MetroHash-${metrohash_commit}.tar.gz::https://github.com/GPUOpen-Drivers/MetroHash/archive/${metrohash_commit}.tar.gz
        CWPack-${cwpack_commit}.tar.gz::https://github.com/GPUOpen-Drivers/CWPack/archive/${cwpack_commit}.tar.gz)

sha256sums=('385c84e1a5cf6476d2a547418fdc41a10921c7a45be25dd0b7e7dfaee6cd63f8'
            '2dc052d20f36f9544036a6ad1b467d62b7274e42e276041cf6ee54be88a83496'
            'fb828be64ab7ee30793df82ef7b537c8e94be88b46a631235416c1e9e296d867'
            '6a5af19c2797261f12e956a01ad2579122f439b8c2c724449c40e19d21f756fa'
            '77a4cad8691960b825e86b624bb5433f5098b8a49cb0d758e17c6d59e25a9361'
            '601f4f32147ef8d676b2ca7ef21a58ae3b0d0fd9c23018083790fad4bf202e59'
            'e8ecf026584dd953e39c3abba2eb04d28b28ed4577482ee70265f0d421fef398'
            '58ca397f33d62bcfecaecd89eb4ad466a6c33e1c619e5cf742822074f1f7d664')

prepare() {
  ln -sf ${srcdir}/AMDVLK-v-${pkgver} ${srcdir}/AMDVLK
  ln -sf ${srcdir}/xgl-${xgl_commit} ${srcdir}/xgl
  ln -sf ${srcdir}/pal-${pal_commit} ${srcdir}/pal
  ln -sf ${srcdir}/llpc-${llpc_commit} ${srcdir}/llpc
  ln -sf ${srcdir}/spvgen-${spvgen_commit} ${srcdir}/spvgen
  ln -sf ${srcdir}/llvm-project-${llvmproject_commit} ${srcdir}/llvm-project
  ln -sf ${srcdir}/MetroHash-${metrohash_commit} ${srcdir}/MetroHash
  ln -sf ${srcdir}/CWPack-${cwpack_commit} ${srcdir}/CWPack
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
