# Contributor: Anton Bazhenov <anton.bazhenov at gmail>
# Contributor: Lone_Wolf <lonewolf@xs4all.nl>
# Contributor: denton <e9203.00 gmail>
# Maintainer: Alex Ferguson <thoughtmonster at gmail>
pkgname=vulture
pkgver=2.3.67
pkgrel=3
pkgdesc='An isometric graphical interface for NetHack, SlashEM and UnNethack'
arch=('i686' 'x86_64')
url='http://old.darkarts.co.za/vulture'
license=('custom:NetHack General Public Licence')
depends=('hicolor-icon-theme' 'libpng' 'sdl_mixer' 'sdl_ttf')
makedepends=('bison' 'flex' 'p7zip')
conflicts=('vulture-nethack')
install=$pkgname.install
source=(http://old.darkarts.co.za/vulture/download/$pkgname-$pkgver.7z
		vulture.png
		png-fix.patch)
md5sums=('be1d0f07095d3385728821e8269a5723'
         '214218b48019b81d59120e4e84ce10ab'
         'e9fdadfecf48d68d0ee12fd55ad67c1b')

build() {
	cd "$srcdir/$pkgname-$pkgver"

	variants=(nethack slashem unnethack)
	for variant in "${variants[@]}"
	do
		sed -e 's|^/\* \(#define LINUX\) \*/|\1|' \
			-e 's|^/\* \(#define TIMED_DELAY\) \*/|\1|' \
			-e 's|^/\* \(#define VAR_PLAYGROUND.*\) \*/|\1|' \
			-e "/^#define VAR_PLAYGROUND/ s|/var/lib/games/nethack|/var/games/$pkgname/$variant|" -i $variant/include/unixconf.h

		sed -e '/^GAMEDIR/     s|$(PREFIX)/games/lib/$(GAME)dir|$(PREFIX)/share/$(GAME)|' \
			-e '/^VARDIR/      s|$(GAMEDIR)|$(PREFIX)/var/games/$(GAME)|' \
			-e '/^SHELLDIR/    s|$(PREFIX)/games|$(PREFIX)/bin|' \
			-e '/^GAMEUID/     s|games|root|' \
			-e '/^GAMEGRP/     s|bin|root|' \
			-e '/^GAMEPERM/    s|04755|0755|' -i $variant/sys/unix/Makefile.top

		sed -e "/^#  define HACKDIR/          s|/usr/games/lib/${variant}dir|/usr/share/$pkgname/$variant|" \
			-e '/^#define COMPRESS/           s|/usr/bin/compress|/bin/gzip|' \
			-e '/^#define COMPRESS_EXTENSION/ s|.Z|.gz|' -i $variant/include/config.h

		sed -e "/^#    define HACKDIR/         s|\.|/usr/share/$pkgname/$variant|" \
			-e '/^# define COMPRESS/           s|/usr/bin/compress|/bin/gzip|' \
			-e '/^# define COMPRESS_EXTENSION/ s|.Z|.gz|' -i $variant/include/config.h

		sed -e "/^HACK/    s|\$HACKDIR/nethack|/usr/bin/$pkgname-$variant|" \
			-e "/^HACKDIR/ s|/usr/games/lib/nethackdir|/usr/share/$pkgname/$variant|" -i $variant/sys/unix/nethack.sh
	done

	patch -Np0 -i "$srcdir/png-fix.patch"
	make home INSTPREFIX=$srcdir/$pkgname-$pkgver/build
}

package() {
	cd "$pkgdir"

	cp -a $srcdir/$pkgname-$pkgver/build/* .

	chgrp -R games var/games
	chmod -R 775   var/games

	mkdir -p usr/bin
	mkdir -p usr/share/$pkgname
	mkdir -p var/games/$pkgname

	variants=(nethack slashem unnethack)
	for variant in "${variants[@]}"
	do
		mv var/games/$pkgname-$variant var/games/$pkgname/$variant
		chmod 664 var/games/$pkgname/$variant/{logfile,perm,record}

		mv $pkgname-$variant usr/bin/$pkgname-$variant-start
		mv $pkgname-$variant-data/$pkgname-$variant usr/bin/$pkgname-$variant
		mv $pkgname-$variant-data/recover usr/bin/$pkgname-$variant-recover
		rm -f $pkgname-$variant-data/{*.{ico,png,nh},license}

		mv $pkgname-$variant-data usr/share/$pkgname/$variant
		install -Dm644 $startdir/$pkgname.png usr/share/icons/hicolor/48x48/apps/$pkgname-$variant.png
	done

	mkdir -p usr/share/applications
	install -Dm644 $srcdir/vulture-$pkgver/dist/linux/*.desktop usr/share/applications
	install -Dm644 $srcdir/$pkgname-$pkgver/LICENSE usr/share/licenses/$pkgname/LICENSE
}
