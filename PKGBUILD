# Maintainer : Anthony Caccia <acaccia at ulb dot ac dot be>
# Contributor: Thorsten Töpper <atsutane-tu@freethoughts.de>
# Contributor: Andrea Scarpino <andrea@archlinux.org>
# Contributor: Dale Blount <dale@archlinux.org>
# Contributor: Tom Newsom <Jeepster@gmx.co.uk>
# Contributor: Michal Krenek <mikos@sg1.cz>

_pkgbase=john
pkgname=john-cuda
pkgver=1.7.9
pkgrel=2
_jumbover=7
pkgdesc="John The Ripper - A fast password cracker - CUDA version"
provides=('john')
conflicts=('john')
arch=('x86_64')
url="http://www.openwall.com/$_pkgbase/"
license=('GPL2' 'custom')
depends=('openssl' 'opencl-nvidia')
makedepends=('cuda')
optdepends=("perl: for executing some of the scripts at /usr/share/john"
            "ruby: for executing some of the scripts at /usr/share/john"
            "python: for executing some of the scripts at /usr/share/john")
backup=('etc/john/john.conf')
install=john.install
source=(http://www.openwall.com/$_pkgbase/g/$_pkgbase-$pkgver.tar.bz2
        http://www.openwall.com/john/g/john-$pkgver-jumbo-$_jumbover.diff.gz
        ftp://ftp.kfki.hu/pub/packages/security/ssh/ossh/libdes-4.04b.tar.gz
        params.h.patch)
md5sums=('45f54fc59386ecd67daaef9f19781d93'
         'b953fcb7f743eeeb5f938a28c352b8ef'
         'c8d5c69f86c2eedb485583b0305284a1'
         'f69ed632eba8fb9e45847a4b4a323787')

build() {
	# jumbo patch
	cd ${srcdir}/$_pkgbase-$pkgver
	patch -p1 < ${srcdir}/$_pkgbase-$pkgver-jumbo-$_jumbover.diff
	cd ${srcdir}/john-$pkgver/src/

	# patch default params
	patch -p0 < ${srcdir}/params.h.patch
	if [ "$CARCH" == "x86_64" ]; then
	    sed -i 's|CFLAGS = -c -Wall -O2|CFLAGS = -c -Wall -O2 -march=x86-64 -DJOHN_SYSTEMWIDE=1|' Makefile
	    sed -i 's|^LDFLAGS =\(.*\)|LDFLAGS =\1 -lm -L/opt/cuda/lib64 /usr/lib/libstdc++.so.6|' Makefile
	    sed -i -e 's|-m486||g' Makefile
	  else sed -i 's|CFLAGS = -c -Wall -O2|CFLAGS = -c -Wall -O2 -march=i686 -DJOHN_SYSTEMWIDE=1|' Makefile
	fi
	sed -i 's|LIBS = -ldes|LIBS = -ldes -Ldes|' Makefile
#	sed -i 's|#include <des.h>|#include "des/des.h"|' KRB5_fmt.c
	sed -i 's|#include <des.h>|#include "des/des.h"|' KRB5_std.h

	# enable OMP
	sed -i 's|#OMPFLAGS = -fopenmp$|OMPFLAGS = -fopenmp|' Makefile

  # fix new versions of cuda
  sed -i 's|NVCC_FLAGS = -c -Xptxas -v -arch sm_10|NVCC_FLAGS = -c -Xptxas -v -arch sm_20|' Makefile
  
  # LDFLAGS and PATH
  echo $PATH | grep -q /opt/cuda/bin || PATH=/opt/cuda/bin:$PATH

	# build john
	make linux-x86-64-cuda -j10
}

package() {
	# config file
	sed -i 's|$JOHN/john.local.conf|/etc/john/john.local.conf|g' ${srcdir}/john-$pkgver/run/john.conf
	sed -i 's|$JOHN|/usr/share/john|g' ${srcdir}/john-$pkgver/run/john.conf
	install -Dm644 ${srcdir}/john-$pkgver/run/john.conf ${pkgdir}/etc/john/john.conf

	# docs
	install -d ${pkgdir}/usr/share/doc/john
	install -m644 ${srcdir}/john-$pkgver/doc/* ${pkgdir}/usr/share/doc/john/
	install -Dm644 ${srcdir}/john-$pkgver/doc/LICENSE ${pkgdir}/usr/share/licenses/$_pkgbase/LICENSE

	# install password list, charset files
	install -d ${pkgdir}/usr/share/john/
	install -m644 ${srcdir}/${_pkgbase}-${pkgver}/run/password.lst ${pkgdir}/usr/share/john/
	install -m644 ${srcdir}/${_pkgbase}-${pkgver}/run/dictionary.rfc2865 ${pkgdir}/usr/share/john/
	install -m644 ${srcdir}/${_pkgbase}-${pkgver}/run/stats ${pkgdir}/usr/share/john/
	install -m644 ${srcdir}/${_pkgbase}-${pkgver}/run/{all,alnum,alpha,digits,lanman}.chr \
 			${pkgdir}/usr/share/john/
	install -m644 ${srcdir}/${_pkgbase}-${pkgver}/run/{dumb16,dumb32,dynamic}.conf \
			${pkgdir}/usr/share/john/

	# install scripts
	john_scripts=(benchmark-unify \
		cracf2john.py \
		genincstats.rb \
		ldif2john.pl \
		lion2john-alt.pl \
		lion2john.pl \
		netntlm.pl \
		netscreen.py \
		odf2john.py \
		pass_gen.pl \
		radius2john.pl \
		sap2john.pl \
		sha-dump.pl \
		sha-test.pl \
		sipdump2john.py)
	for john_script in "${john_scripts[@]}"; do
		install -m755 ${srcdir}/${_pkgbase}-${pkgver}/run/${john_script} \
			${pkgdir}/usr/share/john
	done

	install -m644 ${srcdir}/${_pkgbase}-${pkgver}/run/dynamic.conf ${pkgdir}/etc/john/
	install -Dm644 ${srcdir}/${_pkgbase}-${pkgver}/run/john.bash_completion \
		${pkgdir}/etc/bash_completion.d/john

	# install binaries
	install -Dm755 ${srcdir}/john-$pkgver/run/john ${pkgdir}/usr/bin/john
	install -Dm755 ${srcdir}/john-$pkgver/run/calc_stat ${pkgdir}/usr/bin/calc_stat
	install -Dm755 ${srcdir}/john-$pkgver/run/genmkvpwd ${pkgdir}/usr/bin/genmkvpwd
	install -Dm755 ${srcdir}/john-$pkgver/run/mkvcalcproba ${pkgdir}/usr/bin/mkvcalcproba
	install -Dm755 ${srcdir}/john-$pkgver/run/relbench ${pkgdir}/usr/bin/relbench
	install -Dm755 ${srcdir}/john-$pkgver/run/tgtsnarf ${pkgdir}/usr/bin/tgtsnarf
	install -Dm755 ${srcdir}/john-$pkgver/run/mailer ${pkgdir}/usr/bin/john-mailer
	install -Dm755 ${srcdir}/john-$pkgver/run/raw2dyna ${pkgdir}/usr/bin/raw2dyna

	# create links
	cd ${pkgdir}/usr/bin
	ln -s john hccap2john
	ln -s john keepass2john
	ln -s john pdf2john
	ln -s john pwsafe2john
	ln -s john racf2john
	ln -s john rar2john
	ln -s john ssh2john
	ln -s john unafs
	ln -s john unique
	ln -s john unshadow
	ln -s john undrop
	ln -s john zip2john
}

# vim:set ts=2 sw=2 et:

