---
title: "Compile FFmpeg on Ubuntu 16.04"
tags:
- ffmpeg
- ubuntu
---

**shadowsocks and proxychains may be needed!**

## Get the Dependencies
---

Copy and paste the whole code box for each step. First install the dependencies:
```bash
sudo apt-get update
sudo apt-get -y install autoconf automake build-essential libass-dev libfreetype6-dev libtheora-dev libtool libvorbis-dev pkg-config texinfo zlib1g-dev nasm cmake mercurial libenca-dev
```

Now make a directory for the source files that will be downloaded later in this guide:

```bash
mkdir ~/ffmpeg_sources
```
## Compilation & Installation
---

You can compile `ffmpeg` to your liking. If you do not require certain encoders you may skip the relevant section and then remove the appropriate `./configure` option in FFmpeg. For example, if libopus is not needed, then skip that section and then remove `--enable-libopus` from the Install FFmpeg section.

This guide is designed to be non-intrusive and will create several directories in your home directory:

- `ffmpeg_sources` – Where the source files will be downloaded.
- `ffmpeg_build` – Where the files will be built and libraries installed.
- `bin` – Where the resulting binaries (`ffmpeg`, `ffplay`, `ffserver`, `x264`, and `yasm`) will be installed.

<!--more-->

### Yasm
---

An assembler for x86 optimizations used by x264 and FFmpeg. Highly recommended or your resulting build may be very slow.

If your repository offers a `yasm` package ≥ 1.2.0 then you can install that instead of compiling:

```bash
sudo apt-get install yasm
```

Otherwise you can compile:
```bash
cd ~/ffmpeg_sources
proxychains wget http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz
tar xvf yasm-1.3.0.tar.gz
cd yasm-1.3.0
./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin"
make
make install
make distclean
```

### libx264
---

H.264 video encoder. See the [H.264 Encoding Guide](https://trac.ffmpeg.org/wiki/Encode/H.264) for more information and usage examples.

Requires `ffmpeg` to be configured with `--enable-gpl` `--enable-libx264`.

If your repository offers a `libx264-dev` package ≥ 0.118 then you can install that instead of compiling:
```bash
sudo apt-get install libx264-dev
```
Otherwise you can compile:
```bash
cd ~/ffmpeg_sources
proxychains wget http://download.videolan.org/pub/x264/snapshots/last_x264.tar.bz2
tar xvf last_x264.tar.bz2
cd x264-snapshot*
PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" --enable-static --disable-opencl
PATH="$HOME/bin:$PATH" make
make install
make distclean
```
### libx265
---
H.265/HEVC video encoder. See the [H.265 Encoding Guide](https://trac.ffmpeg.org/wiki/Encode/H.265) for more information and usage examples.
```bash
cd ~/ffmpeg_sources
proxychains hg clone https://bitbucket.org/multicoreware/x265
cd ~/ffmpeg_sources/x265/build/linux
PATH="$HOME/bin:$PATH" cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="$HOME/ffmpeg_build" -DENABLE_SHARED:bool=off ../../source
make
make install
make distclean
```
### libfdk-aac
---
AAC audio encoder. See the [AAC Audio Encoding Guide](https://trac.ffmpeg.org/wiki/Encode/AAC) for more information and usage examples.

Requires `ffmpeg` to be configured with `--enable-libfdk-aac` (and `--enable-nonfree` if you also included `--enable-gpl`).

If your repository offers a libfdk-aac-dev package then you can install that instead of compiling:
```bash
sudo apt-get install libfdk-aac-dev
```
Otherwise you can compile:

The error "error: Libtool library used but 'LIBTOOL' is undefined" after running "autoreconf -fiv" can be resolved by running the command: libtoolize.
```bash
cd ~/ffmpeg_sources
proxychains wget -O fdk-aac.tar.gz https://github.com/mstorsjo/fdk-aac/tarball/master
tar xzvf fdk-aac.tar.gz
cd mstorsjo-fdk-aac*
autoreconf -fiv
./configure --prefix="$HOME/ffmpeg_build" --disable-shared
make
make install
make distclean
```
### libmp3lame
---
MP3 audio encoder.

Requires `ffmpeg` to be configured with `--enable-libmp3lame`.

If your repository offers a `libmp3lame-dev` package ≥ 3.98.3 then you can install that instead of compiling:
```bash
sudo apt-get install libmp3lame-dev
```
Otherwise you can compile:
```bash
cd ~/ffmpeg_sources
proxychains wget http://downloads.sourceforge.net/project/lame/lame/3.99/lame-3.99.5.tar.gz
tar xzvf lame-3.99.5.tar.gz
cd lame-3.99.5
./configure --prefix="$HOME/ffmpeg_build" --enable-nasm --disable-shared
make
make install
make distclean
```
### libopus
---
Opus audio decoder and encoder.

Requires `ffmpeg` to be configured with `--enable-libopus`.

If your repository offers a `libopus-dev` package ≥ 1.1 then you can install that instead of compiling:
```bash
sudo apt-get install libopus-dev
```
Otherwise you can compile:
```bash
cd ~/ffmpeg_sources
proxychains wget http://downloads.xiph.org/releases/opus/opus-1.1.3.tar.gz
tar xzvf opus-1.1.3.tar.gz
cd opus-1.1.3
./configure --prefix="$HOME/ffmpeg_build" --disable-shared
make
make install
make clean
```
### libvpx
---
VP8/VP9 video encoder and decoder. See the [VP8 Video Encoding Guide](https://trac.ffmpeg.org/wiki/Encode/VP8) for more information and usage examples.

Requires `ffmpeg` to be configured with `--enable-libvpx`.
```bash
cd ~/ffmpeg_sources
proxychains wget http://storage.googleapis.com/downloads.webmproject.org/releases/webm/libvpx-1.6.0.tar.bz2
tar xjvf libvpx-1.6.0.tar.bz2
cd libvpx-1.6.0
PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --disable-examples --disable-unit-tests
PATH="$HOME/bin:$PATH" make
make install
make clean
```
### ffmpeg
---
```bash
cd ~/ffmpeg_sources
proxychains wget http://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2
tar xjvf ffmpeg-snapshot.tar.bz2
cd ffmpeg
PATH="$HOME/bin:$PATH" PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure \
  --prefix="$HOME/ffmpeg_build" \
  --pkg-config-flags="--static" \
  --extra-cflags="-I$HOME/ffmpeg_build/include" \
  --extra-ldflags="-L$HOME/ffmpeg_build/lib" \
  --bindir="$HOME/bin" \
  --enable-gpl \
  --enable-libass \
  --enable-libfdk-aac \
  --enable-libfreetype \
  --enable-libmp3lame \
  --enable-libopus \
  --enable-libtheora \
  --enable-libvorbis \
  --enable-libvpx \
  --enable-libx264 \
  --enable-libx265 \
  --enable-nonfree
PATH="$HOME/bin:$PATH" make
make install
make distclean
hash -r
```
build debug version:
```bash
cd ~/ffmpeg_sources
proxychains wget http://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2
tar xjvf ffmpeg-snapshot.tar.bz2
cd ffmpeg
PATH="$HOME/bin:$PATH" PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure \
  --prefix="$HOME/ffmpeg_build" \
  --pkg-config-flags="--static" \
  --extra-cflags="-I$HOME/ffmpeg_build/include" \
  --extra-ldflags="-L$HOME/ffmpeg_build/lib" \
  --bindir="$HOME/bin" \
  --enable-gpl \
  --enable-libass \
  --enable-libfdk-aac \
  --enable-libfreetype \
  --enable-libmp3lame \
  --enable-libopus \
  --enable-libtheora \
  --enable-libvorbis \
  --enable-libvpx \
  --enable-libx264 \
  --enable-libx265 \
  --enable-nonfree \
  --disable-yasm \
  --disable-mmx \
  --enable-debug=3 \
  --disable-optimizations \
  --disable-doc \
  --disable-htmlpages
PATH="$HOME/bin:$PATH" make
make install
make distclean
hash -r
```
### Conclusion
---
Installation is now complete and `ffmpeg` is now ready for use. Your newly compiled FFmpeg programs are in `~/bin`.

# All in one script
---
```bash
#!/bin/bash

sudo apt-get update
sudo apt-get -y install autoconf automake build-essential libass-dev libfreetype6-dev libtheora-dev libtool libvorbis-dev pkg-config texinfo zlib1g-dev nasm cmake mercurial

mkdir ~/ffmpeg_sources
mkdir ~/ffmpeg_build

#Yasm
cd ~/ffmpeg_sources
proxychains wget http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz
tar xvf yasm-1.3.0.tar.gz
cd yasm-1.3.0
./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin"
make
make install
make distclean

#libx264
cd ~/ffmpeg_sources
proxychains wget http://download.videolan.org/pub/x264/snapshots/last_x264.tar.bz2
tar xvf last_x264.tar.bz2
cd x264-snapshot*
PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" --enable-static --disable-opencl
PATH="$HOME/bin:$PATH" make
make install
make distclean

#libx265
cd ~/ffmpeg_sources
proxychains hg clone https://bitbucket.org/multicoreware/x265
cd ~/ffmpeg_sources/x265/build/linux
PATH="$HOME/bin:$PATH" cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="$HOME/ffmpeg_build" -DENABLE_SHARED:bool=off ../../source
make
make install
make distclean

#libfdk-aac
cd ~/ffmpeg_sources
proxychains wget -O fdk-aac.tar.gz https://github.com/mstorsjo/fdk-aac/tarball/master
tar xzvf fdk-aac.tar.gz
cd mstorsjo-fdk-aac*
autoreconf -fiv
./configure --prefix="$HOME/ffmpeg_build" --disable-shared
make
make install
make distclean

#libmp3lame
cd ~/ffmpeg_sources
proxychains wget http://downloads.sourceforge.net/project/lame/lame/3.99/lame-3.99.5.tar.gz
tar xzvf lame-3.99.5.tar.gz
cd lame-3.99.5
./configure --prefix="$HOME/ffmpeg_build" --enable-nasm --disable-shared
make
make install
make distclean

#libopus
cd ~/ffmpeg_sources
proxychains wget http://downloads.xiph.org/releases/opus/opus-1.1.3.tar.gz
tar xzvf opus-1.1.3.tar.gz
cd opus-1.1.3
./configure --prefix="$HOME/ffmpeg_build" --disable-shared
make
make install
make clean

#libvpx
cd ~/ffmpeg_sources
proxychains wget http://storage.googleapis.com/downloads.webmproject.org/releases/webm/libvpx-1.6.0.tar.bz2
tar xjvf libvpx-1.6.0.tar.bz2
cd libvpx-1.6.0
PATH="$HOME/bin:$PATH" ./configure --prefix="$HOME/ffmpeg_build" --disable-examples --disable-unit-tests
PATH="$HOME/bin:$PATH" make
make install
make clean

#ffmpeg
cd ~/ffmpeg_sources
proxychains wget http://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2
tar xjvf ffmpeg-snapshot.tar.bz2
cd ffmpeg
PATH="$HOME/bin:$PATH" PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure \
  --prefix="$HOME/ffmpeg_build" \
  --pkg-config-flags="--static" \
  --extra-cflags="-I$HOME/ffmpeg_build/include" \
  --extra-ldflags="-L$HOME/ffmpeg_build/lib" \
  --bindir="$HOME/bin" \
  --enable-gpl \
  --enable-libass \
  --enable-libfdk-aac \
  --enable-libfreetype \
  --enable-libmp3lame \
  --enable-libopus \
  --enable-libtheora \
  --enable-libvorbis \
  --enable-libvpx \
  --enable-libx264 \
  --enable-libx265 \
  --enable-nonfree \
  --disable-yasm \
  --disable-mmx \
  --enable-debug=3 \
  --disable-optimizations \
  --disable-doc \
  --disable-htmlpages
PATH="$HOME/bin:$PATH" make
make install
make distclean
hash -r
```
[build-all.sh](/uploads/76b44c826a02c4246c7b5699fbd7cd6e/build-all.sh)