# Maintainer: Eric Vidal <eric@obarun.org>
# based on the original https://git.archlinux.org/svntogit/packages.git/tree/trunk?h=packages/syslinux
# 						Maintainer: Tobias Powalowski <tpowa@archlinux.org>
# 						Maintainer: Thomas Bächler <thomas@archlinux.org>
# 						Maintainer: Anatol Pomozov <anatol.pomozov@gmail.com>
# 						Contributor: Keshav Amburay <(the ddoott ridikulus ddoott rat) (aatt) (gemmaeiil) (ddoott) (ccoomm)>

pkgname=syslinux
pkgver=6.03
_tag=syslinux-$pkgver
pkgrel=11
pkgdesc='Collection of boot loaders that boot from FAT, ext2/3/4 and btrfs filesystems, from CDs and via PXE'
url='http://www.syslinux.org/'
arch=(x86_64)
backup=(boot/syslinux/syslinux.cfg)
install=syslinux.install
license=(GPL2)
# syslinux build system is a mess of submakes that does not work with -jN
# efi32/com32 do not like Arch cflags/ldflags, though it would be nice to have the flags for userspace tools
options=(!makeflags !buildflags)
makedepends=(git python2 nasm upx asciidoc)
makedepends_x86_64=(lib32-glibc) # efi32 needs it
optdepends=('perl-crypt-passwdmd5: For md5pass'
            'perl-digest-sha1:     For sha1pass'
            'mtools:               For mkdiskimage and syslinux support'
            'gptfdisk:             For GPT support'
            'util-linux:           For isohybrid'
            'efibootmgr:           For EFI support'
            'dosfstools:           For EFI support')

# The syslinux-install_update script is maintained at https://gist.github.com/pyther/772138
# Script not yet updated for syslinux-efi
source=(git://git.kernel.org/pub/scm/boot/syslinux/syslinux.git#tag=$_tag
        syslinux.cfg
        syslinux-install_update
        splash.png
        btrfs-fix.patch::http://repo.or.cz/syslinux.git/patch/548386049cd41e887079cdb904d3954365eb28f3?hp=721a0af2f0ba111c31685c5f6c5481eb25346971
        gcc-fix-alignment.patch::http://repo.or.cz/syslinux.git/patch/e5f2b577ded109291c9632dacb6eaa621d8a59fe?hp=8dc6d758b564a1ccc44c3ae11f265d43628219ce
        dont-guess-alignment.patch::http://repo.or.cz/syslinux.git/patch/0cc9a99e560a2f52bcf052fd85b1efae35ee812f?hp=e5f2b577ded109291c9632dacb6eaa621d8a59fe
        kdb-230.patch::http://repo.or.cz/syslinux.git/patch/138e850fab106b5235178848b3e0d33e25f4d3a2
        correct_base_type.patch::http://repo.or.cz/syslinux.git/patch/83aad4f
        set_mode_base.patch::http://repo.or.cz/syslinux.git/patch/0a2dbb3
		fix_return_pointer.patch::http://repo.or.cz/syslinux.git/patch/8dc6d758b564a1ccc44c3ae11f265d43628219ce
		fix_infinite_loop_tests.patch
)
sha1sums=('SKIP'
          '223e1943eb922ecdda338b7faa991f3a7948168a'
          '2081d774731498f6bd598505a2a8c5d3b260cb00'
          '86320f7a18a8dbc913a2bfcb08b09ecd33f7ed30'
          '6ebf77bf028c928a6ef33dadeee7402b3113b6d3'
          'eaa9f5cd82b501f076ece4812d2d37f49d02caae'
          'c6a6e96e233b2f8105503725cd614abc1bac2845'
          'f50e0a92c65536ef73a84614d489e52d9d1db329'
          'e24bf5b1617bab4a3f46925c5a8ee6079f4686bf'
          '500c174fce91133d40024b28f6f5cedb144b84c2'
          'b3d2196aaec154749c5b796c6d9bfd605a918cf8'
          '7ecb02550666dfafeb0b22a67dcc34caa4b79767')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal

_targets='bios efi32 efi64'

prepare() {
  cd syslinux

  # FS#48253
  patch -p1 < ../gcc-fix-alignment.patch
  patch -p1 < ../dont-guess-alignment.patch

  # FS#48214
  patch -p1 < ../btrfs-fix.patch

  # FS#49046
  patch -p1 < ../kdb-230.patch

  # FS#53083
  patch -p1 < ../correct_base_type.patch
  patch -p1 < ../set_mode_base.patch
  
  # FS#49250
  patch -p1 < ../fix_return_pointer.patch

  # fix infinite loop in load_linux
  patch -p1 < ../fix_infinite_loop_tests.patch

  # do not swallow efi compilation output to make debugging easier
  sed 's|> /dev/null 2>&1||' -i efi/check-gnu-efi.sh

  # disable debug and development flags to reduce bootloader size
  truncate --size 0 mk/devel.mk
  
}

build() {
  cd syslinux
  export LDFLAGS+=--no-dynamic-linker  # workaround for binutils 2.28 http://www.syslinux.org/wiki/index.php?title=Building
  export EXTRA_CFLAGS=-fno-PIE   # to fix gpxe build
  make PYTHON=python2 $_targets
}

check() {
  cd syslinux
  make unittest
}

package() {
  cd syslinux
  make $_targets install INSTALLROOT="$pkgdir" SBINDIR=/usr/bin MANDIR=/usr/share/man AUXDIR=/usr/lib/syslinux

  rm -r "$pkgdir"/usr/lib/syslinux/{com32,dosutil,syslinux.com}
  install -D -m644 COPYING "$pkgdir"/usr/share/licenses/syslinux/COPYING
  install -d "$pkgdir"/usr/share/doc
  cp -ar doc "$pkgdir"/usr/share/doc/syslinux

  install -d "$pkgdir"/usr/lib/syslinux/bios
  mv "$pkgdir"/usr/lib/syslinux/{*.bin,*.c32,*.0,memdisk} "$pkgdir"/usr/lib/syslinux/bios 

  install -D -m0644 ../syslinux.cfg "$pkgdir"/boot/syslinux/syslinux.cfg
  install -D -m0644 ../splash.png "$pkgdir"/boot/syslinux/splash.png
  install -D -m0755 ../syslinux-install_update "$pkgdir"/usr/bin/syslinux-install_update
}
