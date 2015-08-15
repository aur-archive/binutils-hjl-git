#Maintainer: Jens Staal <staal1978@gmail.com>

pkgname=binutils-hjl-git
_pkgname=binutils
pkgver=83598.819843c
pkgrel=2
pkgdesc="Binutils with 'hjl' patches needed to build Linux with LTO"
arch=('i686' 'x86_64')
url="http://www.gnu.org/software/binutils/"
license=('GPL')
groups=('base-devel')
depends=('glibc>=2.17' 'zlib')
makedepends=('git' 'autoconf')
checkdepends=('dejagnu' 'bc')
options=('!libtool')
install=binutils.install

provides=("binutils=2.26" 'gdb')
conflict=('binutils' 'gdb')
replaces=('binutils' 'gdb')

source=("$_pkgname::git://sourceware.org/git/binutils-gdb.git")
md5sums=('SKIP')

pkgver() {
    cd "${srcdir}/${_pkgname}"
    echo $(git rev-list --count HEAD).$(git rev-parse --short HEAD)
}


prepare() {
  cd "${srcdir}/${_pkgname}"
  # hack! - libiberty configure tests for header files using "$CPP $CPPFLAGS"
  sed -i "/ac_cpp=/s/\$CPPFLAGS/\$CPPFLAGS -O2/" libiberty/configure

  #get patches to apply...
  rm -rf "${srcdir}/work"
  cp -r "${srcdir}/${_pkgname}" "${srcdir}/work"
  cd "${srcdir}/work"
  
  #https://github.com/andikleen/linux-misc/commit/664de370c2f8f3ae4468ce1e6ed7c80884019c17#commitcomment-10732286
  #3
  git checkout -b users/hjl/linux/master origin/users/hjl/linux/master
  #4
  rm -f -r configure autom4te.cache
  #5
  autoconf_version=$(autoconf -V | grep "autoconf" | tr ' ' '\n' | tail -1) 
  sed -i "s/2.64/${autoconf_version}/g" ./config/override.m4
  #6
  autoreconf -vfi
  #7
  /bin/sh patches/README      # this applies the needed 'hjl' patches
  
  rm -rf ${srcdir}/binutils-build
  mkdir ${srcdir}/binutils-build
}

build() {
  cd ${srcdir}/binutils-build
  
  CFLAGS="$CFLAGS -fPIC"
  
  "${srcdir}/work"/configure --prefix=/usr \
    --with-lib-path=/usr/lib:/usr/local/lib \
    --with-bugurl=https://bugs.archlinux.org/ \
    --enable-threads --enable-shared --with-pic \
    --enable-ld=default --enable-gold --enable-plugins \
    --enable-lto --disable-werror

  # check the host environment and makes sure all the necessary tools are available
  make configure-host

  make tooldir=/usr
}

#check() {
#  cd ${srcdir}/binutils-build

  # unset LDFLAGS as testsuite makes assumptions about which ones are active
  # ignore failures in gold testsuite...
#  make -k LDFLAGS="" check || true
#}

package() {
  cd ${srcdir}/binutils-build
  make prefix=${pkgdir}/usr tooldir=${pkgdir}/usr install

  # Remove unwanted files
  rm ${pkgdir}/usr/share/man/man1/{dlltool,nlmconv,windres,windmc}*

  # No shared linking to these files outside binutils
  rm ${pkgdir}/usr/lib/lib{bfd,opcodes}.so
} 
