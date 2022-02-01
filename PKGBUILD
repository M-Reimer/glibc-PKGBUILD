# Maintainer:  Bartłomiej Piotrowski <bpiotrowski@archlinux.org>
# Contributor: Allan McRae <allan@archlinux.org>

# toolchain build order: linux-api-headers->glibc->binutils->gcc->binutils->glibc
# NOTE: valgrind requires rebuilt with each major glibc version

#
# https://security.archlinux.org/package/glibc
#
# CVE-2021-27645  2021-02-25  OK
# CVE-2021-33574  2021-05-27  OK
# CVE-2021-35942  2021-06-25  OK
# CVE-2021-43396  2021-11-02  OK
# CVE-2021-3998   2022-01-13  OK
# CVE-2021-3999   2022-01-21  OK
#

pkgbase=glibc
pkgname=(glibc lib32-glibc)
pkgver=2.33
pkgrel=5
arch=(x86_64)
url='https://www.gnu.org/software/libc'
license=(GPL LGPL)
makedepends=(git gd lib32-gcc-libs python)
optdepends=('perl: for mtrace')
options=(!strip staticlibs !lto)
#_commit=3de512be7ea6053255afed6154db9ee31d4e557a
#source=(git+https://sourceware.org/git/glibc.git#commit=$_commit
source=(https://ftp.gnu.org/gnu/glibc/glibc-$pkgver.tar.xz{,.sig}
        locale.gen.txt
        locale-gen
        lib32-glibc.conf
        sdt.h sdt-config.h
        bz27343.patch
        0001-nptl_db-Support-different-libpthread-ld.so-load-orde.patch
        0002-nptl-Check-for-compatible-GDB-in-nptl-tst-pthread-gd.patch
        0003-nptl-Do-not-build-nptl-tst-pthread-gdb-attach-as-PIE.patch
        CVE-2021-27645.patch
        CVE-2021-33574_1.patch
        CVE-2021-33574_2.patch
        CVE-2021-35942.patch
        CVE-2021-43396.patch
        CVE-2021-3998.patch
        CVE-2021-3999.patch
       )
validpgpkeys=(7273542B39962DF7B299931416792B4EA25340F8 # Carlos O'Donell
              BC7C7372637EC10C57D7AA6579C43DFBF1CF2187) # Siddhesh Poyarekar
md5sums=('390bbd889c7e8e8a7041564cb6b27cca'
         'SKIP'
         '07ac979b6ab5eeb778d55f041529d623'
         '476e9113489f93b348b21e144b6a8fcf'
         '6e052f1cb693d5d3203f50f9d4e8c33b'
         '91fec3b7e75510ae2ac42533aa2e695e'
         '680df504c683640b02ed4a805797c0b2'
         'cfe57018d06bf748b8ca1779980fef33'
         '78f041fc66fee4ee372f13b00a99ff72'
         '9e418efa189c20053e887398df2253cf'
         '7a09f1693613897add1791e7aead19c9'
         '86eb9569f8fc87514ae289876f3272f1'
         '09ab4f43c81684597d505bbc36260573'
         '9a82016800594f133a01d27e11552329'
         'ba5b6bdc581b0c6af4961ff52a1be21b'
         'fe3cd79561ac6c22fa275b4775e99e78'
         '7aa2d117a6068266f7cbf8aa15dc698c'
         '9fa976739cbf3fe388b3e999a18c1af2')

prepare() {
  mkdir -p glibc-build lib32-glibc-build

  [[ -d glibc-$pkgver ]] && ln -s glibc-$pkgver glibc 
  cd glibc

  # commit c3479fb7939898ec22c655c383454d6e8b982a67
  patch -p1 -i "$srcdir"/bz27343.patch

  # nptl_db: Support different libpthread/ld.so load orders (bug 27744)
  patch -p1 -i "$srcdir"/0001-nptl_db-Support-different-libpthread-ld.so-load-orde.patch

  # nptl: Check for compatible GDB in nptl/tst-pthread-gdb-attach
  patch -p1 -i "$srcdir"/0002-nptl-Check-for-compatible-GDB-in-nptl-tst-pthread-gd.patch

  # nptl: Do not build nptl/tst-pthread-gdb-attach as PIE
  patch -p1 -i "$srcdir"/0003-nptl-Do-not-build-nptl-tst-pthread-gdb-attach-as-PIE.patch

  # Security patches
  patch -p1 -i "$srcdir"/CVE-2021-27645.patch
  patch -p1 -i "$srcdir"/CVE-2021-33574_1.patch
  patch -p1 -i "$srcdir"/CVE-2021-33574_2.patch
  patch -p1 -i "$srcdir"/CVE-2021-35942.patch
  patch -p1 -i "$srcdir"/CVE-2021-43396.patch
  patch -p1 -i "$srcdir"/CVE-2021-3998.patch
  patch -p1 -i "$srcdir"/CVE-2021-3999.patch
}

build() {
  local _configure_flags=(
      --prefix=/usr
      --with-headers=/usr/include
      --with-bugurl=https://bugs.archlinux.org/
      --enable-add-ons
      --enable-bind-now
      --enable-cet
      --enable-kernel=4.4
      --enable-lock-elision
      --enable-multi-arch
      --enable-stack-protector=strong
      --enable-stackguard-randomization
      --enable-static-pie
      --enable-systemtap
      --disable-profile
      --disable-werror
  )

  cd "$srcdir/glibc-build"

  echo "slibdir=/usr/lib" >> configparms
  echo "rtlddir=/usr/lib" >> configparms
  echo "sbindir=/usr/bin" >> configparms
  echo "rootsbindir=/usr/bin" >> configparms

  # remove fortify for building libraries
  CPPFLAGS=${CPPFLAGS/-D_FORTIFY_SOURCE=2/} # Before and
  CFLAGS=${CFLAGS/,-D_FORTIFY_SOURCE=2/} # after https://github.com/archlinux/svntogit-packages/commit/a790c38
  CFLAGS=${CFLAGS/-Wp /} # and after https://github.com/archlinux/svntogit-packages/commit/0a33133

  #
  CFLAGS=${CFLAGS/-fno-plt/}
  CXXFLAGS=${CXXFLAGS/-fno-plt/}
  LDFLAGS=${LDFLAGS/,-z,now/}

  "$srcdir/glibc/configure" \
      --libdir=/usr/lib \
      --libexecdir=/usr/lib \
      ${_configure_flags[@]}

  # build libraries with fortify disabled
  echo "build-programs=no" >> configparms
  make

  # re-enable fortify for programs
  sed -i "/build-programs=/s#no#yes#" configparms

  echo "CC += -D_FORTIFY_SOURCE=2" >> configparms
  echo "CXX += -D_FORTIFY_SOURCE=2" >> configparms
  make

  # build info pages manually for reprducibility
  make info

  cd "$srcdir/lib32-glibc-build"
  export CC="gcc -m32 -mstackrealign"
  export CXX="g++ -m32 -mstackrealign"

  echo "slibdir=/usr/lib32" >> configparms
  echo "rtlddir=/usr/lib32" >> configparms
  echo "sbindir=/usr/bin" >> configparms
  echo "rootsbindir=/usr/bin" >> configparms

  # remove fortify for building libraries
  CPPFLAGS=${CPPFLAGS/-D_FORTIFY_SOURCE=2/} # Before and
  CFLAGS=${CFLAGS/,-D_FORTIFY_SOURCE=2/} # after https://github.com/archlinux/svntogit-packages/commit/a790c38
  CFLAGS=${CFLAGS/-Wp /} # and after https://github.com/archlinux/svntogit-packages/commit/0a33133

  CFLAGS=${CFLAGS/-fno-plt/}
  CXXFLAGS=${CXXFLAGS/-fno-plt/}

  "$srcdir/glibc/configure" \
      --host=i686-pc-linux-gnu \
      --libdir=/usr/lib32 \
      --libexecdir=/usr/lib32 \
      ${_configure_flags[@]}

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

package_glibc() {
  pkgdesc='GNU C Library'
  depends=('linux-api-headers>=4.10' tzdata filesystem)
  optdepends=('gd: for memusagestat')
  install=glibc.install
  backup=(etc/gai.conf
          etc/locale.gen
          etc/nscd.conf)

  install -dm755 "$pkgdir/etc"
  touch "$pkgdir/etc/ld.so.conf"

  make -C glibc-build install_root="$pkgdir" install
  rm -f "$pkgdir"/etc/ld.so.{cache,conf}

  # Shipped in tzdata
  rm -f "$pkgdir"/usr/bin/{tzselect,zdump,zic}

  cd glibc

  install -dm755 "$pkgdir"/usr/lib/{locale,systemd/system,tmpfiles.d}
  install -m644 nscd/nscd.conf "$pkgdir/etc/nscd.conf"
  install -m644 nscd/nscd.service "$pkgdir/usr/lib/systemd/system"
  install -m644 nscd/nscd.tmpfiles "$pkgdir/usr/lib/tmpfiles.d/nscd.conf"
  install -dm755 "$pkgdir/var/db/nscd"

  install -m644 posix/gai.conf "$pkgdir"/etc/gai.conf

  install -m755 "$srcdir/locale-gen" "$pkgdir/usr/bin"

  # Create /etc/locale.gen
  install -m644 "$srcdir/locale.gen.txt" "$pkgdir/etc/locale.gen"
  sed -e '1,3d' -e 's|/| |g' -e 's|\\| |g' -e 's|^|#|g' \
    "$srcdir/glibc/localedata/SUPPORTED" >> "$pkgdir/etc/locale.gen"

  if check_option 'debug' n; then
    find "$pkgdir"/usr/bin -type f -executable -exec strip $STRIP_BINARIES {} + 2> /dev/null || true
    find "$pkgdir"/usr/lib -name '*.a' -type f -exec strip $STRIP_STATIC {} + 2> /dev/null || true

    # Do not strip these for gdb and valgrind functionality, but strip the rest
    find "$pkgdir"/usr/lib \
      -not -name 'ld-*.so' \
      -not -name 'libc-*.so' \
      -not -name 'libpthread-*.so' \
      -not -name 'libthread_db-*.so' \
      -name '*-*.so' -type f -exec strip $STRIP_SHARED {} + 2> /dev/null || true
  fi

  # Provide tracing probes to libstdc++ for exceptions, possibly for other
  # libraries too. Useful for gdb's catch command.
  install -Dm644 "$srcdir/sdt.h" "$pkgdir/usr/include/sys/sdt.h"
  install -Dm644 "$srcdir/sdt-config.h" "$pkgdir/usr/include/sys/sdt-config.h"

  # Provided by libxcrypt; keep the old shared library for backwards compatibility
  rm -f "$pkgdir"/usr/include/crypt.h "$pkgdir"/usr/lib/libcrypt.{a,so}
}

package_lib32-glibc() {
  pkgdesc='GNU C Library (32-bit)'
  depends=("glibc=$pkgver")
  options+=('!emptydirs')

  cd lib32-glibc-build

  make install_root="$pkgdir" install
  rm -rf "$pkgdir"/{etc,sbin,usr/{bin,sbin,share},var}

  # We need to keep 32 bit specific header files
  find "$pkgdir/usr/include" -type f -not -name '*-32.h' -delete

  # Dynamic linker
  install -d "$pkgdir/usr/lib"
  ln -s ../lib32/ld-linux.so.2 "$pkgdir/usr/lib/"

  # Add lib32 paths to the default library search path
  install -Dm644 "$srcdir/lib32-glibc.conf" "$pkgdir/etc/ld.so.conf.d/lib32-glibc.conf"

  # Symlink /usr/lib32/locale to /usr/lib/locale
  ln -s ../lib/locale "$pkgdir/usr/lib32/locale"

  if check_option 'debug' n; then
    find "$pkgdir"/usr/lib32 -name '*.a' -type f -exec strip $STRIP_STATIC {} + 2> /dev/null || true
    find "$pkgdir"/usr/lib32 \
      -not -name 'ld-*.so' \
      -not -name 'libc-*.so' \
      -not -name 'libpthread-*.so' \
      -not -name 'libthread_db-*.so' \
      -name '*-*.so' -type f -exec strip $STRIP_SHARED {} + 2> /dev/null || true
  fi

  # Provided by lib32-libxcrypt; keep the old shared library for backwards compatibility
  rm -f "$pkgdir"/usr/lib32/libcrypt.{a,so}
}
