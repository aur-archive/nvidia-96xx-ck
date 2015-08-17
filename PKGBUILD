# Maintainer: 3ED_0 <krzysztof1987 at gmail>
# Contributor: Maximilien Noal <noal dot maximilien at gmail dot com>
# Contributor: graysky <graysky AT archlnux.us>

pkgname=nvidia-96xx-ck
_kname=ck

pkgver=96.43.23
pkgrel=23
_kver_ge="3.0"
_kver_lt="3.20"

if [ -z "$_ppfix" ] && [[ $pkgname =~ ^nvidia-96xx(.*)$ ]]; then
  _ppfix="${BASH_REMATCH[1]}"
fi

pkgdesc="NVIDIA drivers for linux${_ppfix}, 96xx branch."
url="http://www.nvidia.com/"
arch=('i686' 'x86_64')
depends=("linux${_ppfix}>=${_kver_ge}"
         "linux${_ppfix}<${_kver_lt}"
         "nvidia-96xx-utils=${pkgver}")
makedepends=("linux${_ppfix}-headers>=${_kver_ge}"
             "linux${_ppfix}-headers<${_kver_lt}")
optdepends=('xorg-server1.12: latest compatible Xorg server')
provides=('nvidia-96xx')
conflicts=('nvidia'
           'nvidia-all'
           'nvidia-96xx-all'
           'nvidia-173xx'
           'nvidia-275xx'
           'nvidia-304xx'
           'nvidia-beta-all'
           'nvidia-beta')
license=('custom')
install='nvidia.install'
options=(!strip)
_npkg=0

if [ "$CARCH" = "x86_64" ]; then
  _arch=x86_64
else
  _arch=x86
fi

source=(http://download.nvidia.com/XFree86/Linux-${_arch}/${pkgver}/NVIDIA-Linux-${_arch}-${pkgver}-pkg${_npkg}.run
        nouveau_blacklist.conf
        173.14.36-37.patch
        173.14.37-38.patch
        linux-3.17.patch
        linux-3.19.patch)

sha512sums=('bed5726e57637481fe4e3c03a65ec14fe949f00860e729ebde408f4fd861d7bfdc296a78bc2f5d42e8b282db09f4bbde1e0545df7228fa20227080dc4b868ba7'
            'e6c5382b6f7dd10f350136d65a714de3ae337978c3ca88e63f016b4a42be87b422d2388fbf6d6a2feba15516fb9b409f6c5ff08968829e6fc2d1e8aeb9d3c508'
            '990aa39f8c4e3afdfc2070a2fe5e29f7e31647d622d5cbd80e8a93f9a33d7fb754f7de0b5b2c41fa4e75a0f0f5fc821a78249f9c61839c9f06fe9ab3922195b2'
            '7f4bd3737d3a02360372c6a54418a0f3e75bf84d0f1d013c985bd2a34b9738600b7c648c22d777c600929e3a2f049570c9a020a5f7cc1993d4031a275bc5e3bd'
            '23ca251d867b986900a4d952b829ff4ec573bac0b95374cb2bbbbf909a1b18a3a86d937409b7e60437a27886b31c2336bf835a5b05ee9c238d7d696eba8ba03a'
            'b89fd55957229ed6e4bb96ae539291b33dfb9a711d7a9180ed64fa3af52f605c1226c9754c5d1d3d1ea41c46b4ea7ce644f3503daf1a1418bc5155ecac27434e')

[ "$CARCH" = "x86_64" ] \
  && sha512sums[0]='6432968467df4f4675d04f73c8ba990a660290bda8526d484cf417c337a789d3f1dcf36acd20e34a6a332ccad29131f15ee626b30f4c84adf4cb6e37e7c234b7'

_extramodules="$(echo /usr/lib/modules/extramodules-[0-9.]*-${_kname}/version|cut -d/ -f5 2> /dev/null)"
_kernver="$(cat /usr/lib/modules/${_extramodules}/version 2> /dev/null)"

prepare() {
  msg2 "Cleaning..."
  test ! -d NVIDIA-Linux-${_arch}-${pkgver}-pkg${_npkg} \
    || rm -rf NVIDIA-Linux-${_arch}-${pkgver}-pkg${_npkg}

  msg2 "Extracting..."
  sh NVIDIA-Linux-${_arch}-${pkgver}-pkg${_npkg}.run --extract-only

  msg2 "Patching..."
  cd NVIDIA-Linux-${_arch}-${pkgver}-pkg${_npkg}/usr/src/nv
  patch -p1 -i "$srcdir"/173.14.36-37.patch
  patch -p1 -i "$srcdir"/173.14.37-38.patch

  # Probably this patch shouldn't work with older kernel versions
  if [[ $_kernver =~ ^3\.([0-9]+) ]]; then
    if   [ "${BASH_REMATCH[1]}" -ge "17" ]; then  # linux >= 3.17
      patch -p1 -i "$srcdir"/linux-3.17.patch

    elif [ "${BASH_REMATCH[1]}" -ge "19" ]; then  # linux >= 3.19
      patch -p1 -i "$srcdir"/linux-3.19.patch

    fi
  fi
}

build() {
  cd NVIDIA-Linux-${_arch}-${pkgver}-pkg${_npkg}/usr/src/nv
  make SYSSRC=/usr/lib/modules/${_kernver}/build module
}

package() {
  install -dm755 "$pkgdir/usr/lib/modprobe.d"
  install -m644 nouveau_blacklist.conf "$pkgdir/usr/lib/modprobe.d/nouveau_blacklist-${_kname}.conf"

  cd NVIDIA-Linux-${_arch}-${pkgver}-pkg${_npkg}
  if [ -n "$_ppfix" ]; then
    install -dm755 "$pkgdir/usr/share/licenses/${pkgname}/"
    install  -m644 LICENSE "$pkgdir/usr/share/licenses/${pkgname}/"
  fi

  cd usr/src/nv
  install -dm755 "$pkgdir/usr/lib/modules/${_extramodules}/"
  install  -m644 nvidia.ko "$pkgdir/usr/lib/modules/${_extramodules}/"
  gzip "${pkgdir}/usr/lib/modules/${_extramodules}/nvidia.ko"

  sed -i -e "s/EXTRAMODULES='.*'/EXTRAMODULES='${_extramodules}'/" "$startdir/nvidia.install"
}
