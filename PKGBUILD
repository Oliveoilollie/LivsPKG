# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Contributor: morwel
# Contributor: Angel Velasquez <angvp@archlinux.org>
# Contributor: Stéphane Gaudreault <stephane@archlinux.org>
# Contributor: Allan McRae <allan@archlinux.org>
# Contributor: Jason Chu <jason@archlinux.org>
# Contributor: Liv Rickey <rickeyollie@gmail.com> ~ new email coming soon

shopt -s extglob

pkgbase=python
pkgname=(python python-tests)
pkgver=3.13.0
pkgrel=1
_pybasever=${pkgver%.*}
pkgdesc="The Python programming language"
arch=('x86_64')
license=('PSF-2.0')
url="https://github.com/AcreetionOS-Linux/python.git"
depends=('bzip2' 'expat' 'gdbm' 'libffi' 'libnsl' 'libxcrypt' 'openssl' 'zlib' 'tzdata' 'mpdecimal')
makedepends=('tk' 'sqlite' 'bluez-libs' 'llvm' 'gdb' 'xorg-server-xvfb' 'ttf-font' 'git')
#source=("https://www.python.org/ftp/python/${pkgver%rc*}/Python-${pkgver}.tar.xz"{,.asc}
#        EXTERNALLY-MANAGED)
#sha512sums=('44a143c9b96b55b01885ec020c3364265bda55289615cd7d5071915b0d0178a6f35e7551a89090001fcb7f3172d38177a56bf8b8532b15c9dbc50295c9210152'
#            'SKIP'
#            '62a6fbfbaeaa3ba7c54e109d9c3b7f67e73bb21986da4c1fcc5d28cca83d71e0fcae28e1fc70ee8ddce7dea8cd0b64e18d1031dae3a2eae5eaa379c53efd53a0')
#validpgpkeys=('0D96DF4D4110E5C43FBFB17F2D347EA6AA65421D'  # Ned Deily (Python release signing key) <nad@python.org>
#              'E3FF2839C048B25C084DEBE9B26995E310250568'  # Łukasz Langa (GPG langa.pl) <lukasz@langa.pl>
#              'A035C8C19219BA821ECEA86B64E628F8D684696D'  # Pablo Galindo Salgado <pablogsal@gmail.com>
#              '7169605F62C751356D054A26A821E680E5FA6305') # Thomas Wouters <thomas@xs4all.nl>

prepare() {
  #sometimes the worst code is the best code

  
  echo "please make sure you dont have a folder named Python-${pkgver} if not git wont download the source and compile, you might have to remove the src dir till i have the time to fix it"

    # it wouldnt get the fucking source code ffs
  git clone https://github.com/AcreetionOS-Linux/python.git "Python-${pkgver}"

  cd Python-${pkgver}

  #change the python verison to 3.13
  git checkout 3.13

  # Ensure that we are using the system copy of various libraries (expat, libffi, and libmpdec),
  # rather than copies shipped in the tarball
  rm -r Modules/expat
  rm -r Modules/_decimal/libmpdec
}

build() {
  cd Python-${pkgver}

  # PGO should be done with -O3
  CFLAGS="${CFLAGS/-O2/-O3} -ffat-lto-objects"

  # Disable bundled pip & setuptools
  ./configure --prefix=/usr \
              --enable-shared \
              --with-computed-gotos \
              --enable-optimizations \
              --with-lto \
              --enable-ipv6 \
              --with-system-expat \
              --with-dbmliborder=gdbm:ndbm \
              --with-system-libmpdec \
              --enable-loadable-sqlite-extensions \
              --without-ensurepip \
              --with-tzpath=/usr/share/zoneinfo

  # Obtain next free server number for xvfb-run; this even works in a chroot environment.
  export servernum=99
  while ! xvfb-run -a -n "$servernum" /bin/true 2>/dev/null; do servernum=$((servernum+1)); done

  LC_CTYPE=en_US.UTF-8 xvfb-run -s "-screen 0 1920x1080x16 -ac +extension GLX" -a -n "$servernum" make EXTRA_CFLAGS="$CFLAGS"
}

check() {
  # test_tk: test_askcolor tkinter.test.test_tkinter.test_colorchooser.DefaultRootTest hangs
  # test_pyexpat: our `debug` implementation rewrites source location, which breaks the build-time
  #               only test test.test_pyexpat.HandlerExceptionTest as it cannot find source file in
  #               the to-be-installed debug package
  # test_socket: https://github.com/python/cpython/issues/79428
  # test_unittest: https://github.com/python/cpython/issues/108927
  # test_tkk: AssertionError: Tuples differ: (0,) != ('0',)
  # test_ssl: flaky tests issues

  #cd Python-${pkgver}

  # Obtain next free server number for xvfb-run; this even works in a chroot environment.
  export servernum=99
  while ! xvfb-run -a -n "$servernum" /bin/true 2>/dev/null; do servernum=$((servernum+1)); done

  LD_LIBRARY_PATH="${srcdir}/Python-${pkgver}":${LD_LIBRARY_PATH} \
  LC_CTYPE=en_US.UTF-8 xvfb-run -s "-screen 0 1920x1080x16 -ac +extension GLX" -a -n "$servernum" \
    "${srcdir}/Python-${pkgver}/python" -m test.regrtest -v -uall -x test_tk -x test_ttk -x test_ttk.test_widgets \
      -x test_tkinter -x test_pyexpat -x test_socket -x test_unittest -x test_ssl
}

package_python() {
  optdepends=('python-setuptools: for building Python packages using tooling that is usually bundled with Python'
              'python-pip: for installing Python packages using tooling that is usually bundled with Python'
              'python-pipx: for installing Python software not packaged on Arch Linux'
              'sqlite: for a default database integration'
              'xz: for lzma'
              'tk: for tkinter')
  provides=('python3' 'python-externally-managed')
  replaces=('python3' 'python-externally-managed')

  #cd Python-${pkgver}

  # Hack to avoid building again
  sed -i 's/^all:.*$/all: build_all/' Makefile

  # PGO should be done with -O3
  CFLAGS="${CFLAGS/-O2/-O3}"

  make DESTDIR="${pkgdir}" EXTRA_CFLAGS="$CFLAGS" install

  # Why are these not done by default...
  ln -s python3               "${pkgdir}"/usr/bin/python
  ln -s python3-config        "${pkgdir}"/usr/bin/python-config
  ln -s idle3                 "${pkgdir}"/usr/bin/idle
  ln -s pydoc3                "${pkgdir}"/usr/bin/pydoc
  ln -s python${_pybasever}.1 "${pkgdir}"/usr/share/man/man1/python.1

  # some useful "stuff" FS#46146
  install -dm755 "${pkgdir}"/usr/lib/python${_pybasever}/Tools/{i18n,scripts}
  install -m755 Tools/i18n/{msgfmt,pygettext}.py "${pkgdir}"/usr/lib/python${_pybasever}/Tools/i18n/
  install -m755 Tools/scripts/{README,*py} "${pkgdir}"/usr/lib/python${_pybasever}/Tools/scripts/

  # PEP668
  install -Dm644 "$srcdir"/EXTERNALLY-MANAGED -t "${pkgdir}/usr/lib/python${_pybasever}/"

  # Split tests
  cd "$pkgdir"/usr/lib/python*/
  rm -r {test,idlelib/idle_test}
}

package_python-tests() {
  pkgdesc="Regression tests packages for Python"
  depends=('python')

  cd Python-${pkgver}

  make DESTDIR="${pkgdir}" EXTRA_CFLAGS="$CFLAGS" libinstall
  cd "$pkgdir"/usr/lib/python*/
  rm -r !(test)

}