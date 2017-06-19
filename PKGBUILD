# $Id$
# Maintainer: Allan McRae <allan@archlinux.org>

# toolchain build order: linux-api-headers->glibc->binutils->gcc->binutils->glibc
# NOTE: valgrind requires rebuilt with each major glibc version

pkgname=glibc
pkgver=2.25
pkgrel=4
_commit=ccb4fd7a657b0fbc4890c98f4586d58a135fc583
pkgdesc='GNU C Library'
arch=(i686 x86_64)
url='http://www.gnu.org/software/libc'
license=(GPL LGPL)
groups=(base)
depends=('linux-api-headers>=4.10' tzdata filesystem)
makedepends=('gcc>=6' git gd)
optdepends=('gd: for memusagestat')
backup=(etc/gai.conf
        etc/locale.gen
        etc/nscd.conf)
options=(!strip staticlibs)
install=glibc.install
source=(git+https://sourceware.org/git/glibc.git#commit=${_commit}
        locale.gen.txt
        locale-gen
        CVE-2017-1000366-rtld-LD_AUDIT.patch
        CVE-2017-1000366-rtld-LD_PRELOAD.patch
        CVE-2017-1000366-rtld-LD_LIBRARY_PATH.patch
        cvs-vectorized-strcspn-guards.patch
        cvs-hwcap-AT_SECURE.patch)
md5sums=('SKIP'
         '07ac979b6ab5eeb778d55f041529d623'
         '476e9113489f93b348b21e144b6a8fcf'
         '519c6eda90310964d12c7c744ce09333'
         'ed0cf195055a66d0b6f6b409c08c3199'
         'ff9ff81f713bb1b062280e669a646114'
         'e78157b48300b2436b3d90059ef02976'
         'bd31ebc2514ad79cbabd25b611c0b2a1')

prepare() {
  mkdir glibc-build
  cd ${pkgname}
  patch -p1 < "${srcdir}/CVE-2017-1000366-rtld-LD_AUDIT.patch"
  patch -p1 < "${srcdir}/CVE-2017-1000366-rtld-LD_PRELOAD.patch"
  patch -p1 < "${srcdir}/CVE-2017-1000366-rtld-LD_LIBRARY_PATH.patch"
  patch -p1 < "${srcdir}/cvs-vectorized-strcspn-guards.patch"
  patch -p1 < "${srcdir}/cvs-hwcap-AT_SECURE.patch"
}

build() {
  cd glibc-build

  if [[ ${CARCH} = "i686" ]]; then
    # Hack to fix NPTL issues with Xen, only required on 32bit platforms
    export CFLAGS="${CFLAGS} -mno-tls-direct-seg-refs"
  fi

  echo "slibdir=/usr/lib" >> configparms
  echo "rtlddir=/usr/lib" >> configparms
  echo "sbindir=/usr/bin" >> configparms
  echo "rootsbindir=/usr/bin" >> configparms

  # remove fortify for building libraries
  CPPFLAGS=${CPPFLAGS/-D_FORTIFY_SOURCE=2/}

  ../${pkgname}/configure \
      --prefix=/usr \
      --libdir=/usr/lib \
      --libexecdir=/usr/lib \
      --with-headers=/usr/include \
      --with-bugurl=https://bugs.archlinux.org/ \
      --enable-add-ons \
      --enable-obsolete-rpc \
      --enable-kernel=2.6.32 \
      --enable-bind-now \
      --disable-profile \
      --enable-stackguard-randomization \
      --enable-stack-protector=strong \
      --enable-lock-elision \
      --enable-multi-arch \
      --disable-werror

  # build libraries with fortify disabled
  echo "build-programs=no" >> configparms
  make

  # re-enable fortify for programs
  sed -i "/build-programs=/s#no#yes#" configparms

  echo "CC += -D_FORTIFY_SOURCE=2" >> configparms
  echo "CXX += -D_FORTIFY_SOURCE=2" >> configparms
  make
}

check() {
  cd glibc-build

  # remove fortify in preparation to run test-suite
  sed -i '/FORTIFY/d' configparms

  # some failures are "expected"
  make check || true
}

package() {
  cd glibc-build

  install -dm755 ${pkgdir}/etc
  touch ${pkgdir}/etc/ld.so.conf

  make install_root=${pkgdir} install

  rm -f ${pkgdir}/etc/ld.so.{cache,conf}

  install -dm755 ${pkgdir}/usr/lib/{locale,systemd/system,tmpfiles.d}

  install -m644 ${srcdir}/${pkgname}/nscd/nscd.conf ${pkgdir}/etc/nscd.conf
  install -m644 ${srcdir}/${pkgname}/nscd/nscd.service ${pkgdir}/usr/lib/systemd/system
  install -m644 ${srcdir}/${pkgname}/nscd/nscd.tmpfiles ${pkgdir}/usr/lib/tmpfiles.d/nscd.conf

  install -m644 ${srcdir}/${pkgname}/posix/gai.conf ${pkgdir}/etc/gai.conf

  install -m755 ${srcdir}/locale-gen ${pkgdir}/usr/bin

  # create /etc/locale.gen
  install -m644 ${srcdir}/locale.gen.txt ${pkgdir}/etc/locale.gen
  sed -e '1,3d' -e 's|/| |g' -e 's|\\| |g' -e 's|^|#|g' \
    ${srcdir}/glibc/localedata/SUPPORTED >> ${pkgdir}/etc/locale.gen

  # Do not strip the following files for improved debugging support
  # ("improved" as in not breaking gdb and valgrind...):
  #   ld-${pkgver}.so
  #   libc-${pkgver}.so
  #   libpthread-${pkgver}.so
  #   libthread_db-1.0.so

  cd $pkgdir
  strip $STRIP_BINARIES usr/bin/{gencat,getconf,getent,iconv,iconvconfig} \
                        usr/bin/{ldconfig,locale,localedef,nscd,makedb} \
                        usr/bin/{pcprofiledump,pldd,rpcgen,sln,sprof} \
                        usr/lib/getconf/*
  if [[ $CARCH = "i686" ]]; then
    strip $STRIP_BINARIES usr/bin/lddlibc4
  fi

  strip $STRIP_STATIC usr/lib/lib{anl,BrokenLocale,c{,_nonshared},crypt}.a \
                      usr/lib/lib{dl,g,ieee,mcheck,nsl,pthread{,_nonshared}}.a \
                      usr/lib/lib{resolv,rpcsvc,rt,util}.a

  strip $STRIP_SHARED usr/lib/lib{anl,BrokenLocale,cidn,crypt}-${pkgver}.so \
                      usr/lib/libnss_{compat,db,dns,files,hesiod,nis,nisplus}-*.so \
                      usr/lib/lib{dl,m,nsl,resolv,rt,util}-${pkgver}.so \
                      usr/lib/lib{memusage,pcprofile,SegFault}.so \
                      usr/lib/{audit,gconv}/*.so || true

  if [[ $CARCH = "x86_64" ]]; then
    strip $STRIP_STATIC usr/lib/lib{m-${pkgver},mvec{,_nonshared}}.a
    strip $STRIP_SHARED usr/lib/libmvec-*.so
  fi
  
  if [[ $CARCH = "i686" ]]; then
    strip $STRIP_STATIC usr/lib/libm.a
  fi
}
