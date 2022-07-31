# Maintainer: Erika <50500602+Enbika@users.noreply.github.com>

pkgname=sonic3air-git
pkgver=22.07.31.2.test.r0.g7b4d3aee
pkgrel=1
pkgdesc='A fan-made widescreen remaster of Sonic 3 & Knuckles.'
arch=('x86_64')
url='https://sonic3air.org/'
license=('GPL3')
depends=('opengl-driver' 'sdl2')
makedepends=('cmake' 'git' 'sed' 'glu' 'alsa-lib' 'xorg-server-xvfb')
optdepends=('discord: Discord rich presence support')
conflicts=(sonic3air sonic3air-bin)
provides=(sonic3air)

source=(
	'git+https://github.com/Eukaryot/sonic3air.git'
	"local://Sonic_Knuckles_wSonic3.bin"
	'sonic3air.desktop'
	'sonic3air.sh'
)

sha256sums=(
	'SKIP'
	'fa52ac946dfd576538d00aa858b790b9d81a1217e25aa5193693a4e57f4f89d9'
	'7c5568d01131935c189b3060ea0319cc0047c8a76c9152bf5d461e70f676eebd'
	'0ac3f4ea42157e6956f85e1e68e7bd456104fd264d10dd2be43e4c38421b3426'
)


pkgver() {
	cd "$srcdir/${pkgname%-git}"
	git describe --long --tags | sed 's/^v//;s/\([^-]*-g\)/r\1/;s/-/./g'
}

prepare() {
	cp -r "$srcdir/${pkgname%-git}/Oxygen/soncthrickles/build/_cmake" "$srcdir/${pkgname%-git}/Oxygen/soncthrickles/source/"
	cp -r "$srcdir/Sonic_Knuckles_wSonic3.bin" "$srcdir/${pkgname%-git}/Oxygen/soncthrickles/___internal/"
	# patch CMakeLists for mastering build
	sed -i 's;target_compile_definitions(Sonic3AIR PUBLIC ENDUSER);target_compile_definitions(Sonic3AIR PUBLIC);' "$srcdir/${pkgname%-git}/Oxygen/soncthrickles/source/_cmake/CMakeLists.txt"
}

build() {
	echo "Building master build..."
	cd "$srcdir/${pkgname%-git}/Oxygen/soncthrickles/source/_cmake"
	cmake -DCMAKE_BUILD_TYPE=Release -B build
	cmake --build build -j $(nproc)

	# run sonic3air_linux for mastering
	cd "$srcdir/${pkgname%-git}/Oxygen/soncthrickles"
	echo "Running script nativization..."
	xvfb-run -s "-screen 0 1920x1080x24 -nolisten local" \
	./sonic3air_linux -dumpcppdefinitions -nativize
	echo "Packing data files..."
	xvfb-run -s "-screen 0 1920x1080x24 -nolisten local" \
	./sonic3air_linux -pack

	# undo previous patch and rebuild with enduser flag set
	cd "$srcdir/${pkgname%-git}/Oxygen/soncthrickles/source/_cmake"
	sed -i 's;target_compile_definitions(Sonic3AIR PUBLIC);target_compile_definitions(Sonic3AIR PUBLIC ENDUSER);' "CMakeLists.txt"
	echo "Building enduser build..."
	cmake -DCMAKE_BUILD_TYPE=Release -B build
	cmake --build build -j $(nproc)
}

package() {
	install -Dm644 "$srcdir/${pkgname%-git}/LICENSE" "$pkgdir/usr/share/licenses/sonic3air/LICENSE"
	install -Dm644 sonic3air.desktop "$pkgdir/usr/share/applications/sonic3air.desktop"
	install -Dm755 sonic3air.sh "$pkgdir/usr/bin/sonic3air"
	
	# Build data packages and meta data
	cd "$srcdir/${pkgname%-git}/Oxygen/soncthrickles"
	install -Dm644 enginedata.bin "$pkgdir/opt/sonic3air/data/enginedata.bin"
	install -Dm644 gamedata.bin "$pkgdir/opt/sonic3air/data/gamedata.bin"
	install -Dm644 audiodata.bin "$pkgdir/opt/sonic3air/data/audiodata.bin"
	install -Dm644 audioremaster.bin "$pkgdir/opt/sonic3air/data/audioremaster.bin"
	install -Dm644 saves/scripts.bin "$pkgdir/opt/sonic3air/data/scripts.bin"
	install -Dm644 data/metadata.json "$pkgdir/opt/sonic3air/data/data/metadata.json"

	# Build the master installation
	cd "$srcdir/${pkgname%-git}/Oxygen/soncthrickles/_master_image_template"
	find . -type f -exec install -D {} "$pkgdir/opt/sonic3air/{}" \;

	cd "$srcdir/${pkgname%-git}/Oxygen/soncthrickles/"
	install -Dm644 data/images/icon.png "$pkgdir/opt/sonic3air/data/icon.png"
	install -Dm755 sonic3air_linux "$pkgdir/opt/sonic3air"
	install -Dm644 source/external/discord_game_sdk/lib/x86_64/libdiscord_game_sdk.so "$pkgdir/opt/sonic3air"
	install -Dm644 data/images/icon.png "$pkgdir/opt/sonic3air/data/icon.png"

	# Add Oxygen engine
	cd "$srcdir/${pkgname%-git}/Oxygen/oxygenengine/_master_image_template"
	find . -type f -exec install -D {} "$pkgdir/opt/sonic3air/bonus/oxygenengine/{}" \;
	cd "$srcdir/${pkgname%-git}/Oxygen/oxygenengine/data"
 	find . -type f -exec install -D {} "$pkgdir/opt/sonic3air/bonus/oxygenengine/data/{}" \;

	install -Dm755 "$srcdir/${pkgname%-git}/Oxygen/oxygenengine/oxygenapp_linux" "$pkgdir/opt/sonic3air/bonus/oxygenengine/oxygenapp_linux"

	# Complete S3AIR dev
	cd "$srcdir/${pkgname%-git}/Oxygen/soncthrickles/scripts/"
	find . -type f -exec install -D {} "$pkgdir/opt/sonic3air/bonus/sonic3air_dev/scripts/{}" \;
}
