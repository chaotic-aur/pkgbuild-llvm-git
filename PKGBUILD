#!/usr/bin/bash

# Based on LordHeavy's LLVM
pkgbase=llvm-git
pkgname=('lldb-git' 'lld-git' 'polly-git' 'compiler-rt-git' 'clang-git' 'llvm-ocaml-git' 'llvm-libs-git' 'llvm-git')
if [[ $CARCH == x86_64 ]]; then
    pkgname+=('lib32-llvm-libs-git' 'lib32-clang-git' 'lib32-llvm-git')
fi
pkgdesc='Low Level Virtual Machine (git version)'
pkgver=12.0.0_r376876.6f0f0220380f
pkgrel=1
groups=('chaotic-mesa-git')
arch=('x86_64' 'armv7h' 'aarch64')
url="https://llvm.org/"
license=('custom:Apache 2.0 with LLVM Exception')
makedepends=('git' 'cmake' 'ninja' 'libffi' 'libedit' 'ncurses' 'libxml2'
             'python-sphinx' 'ocaml' 'ocaml-ctypes' 'ocaml-findlib'
             'python-recommonmark' 'cuda' 'ocl-icd' 'opencl-headers'
             'swig' 'python' 'libunwind' 'python2' 'tensorflow')
if [[ $CARCH == x86_64 ]]; then
    makedepends+=('lib32-gcc-libs' 'lib32-libffi' 'lib32-libunwind'
                  'lib32-libxml2' 'lib32-zlib')
fi

source=("llvm-project::git+https://github.com/llvm/llvm-project.git"
        "llvm-config.h")

md5sums=('SKIP'
         '295c343dcd457dc534662f011d7cff1a')
sha512sums=('SKIP'
            '75e743dea28b280943b3cc7f8bbb871b57d110a7f2b9da2e6845c1c36bf170dd883fca54e463f5f49e0c3effe07fbd0db0f8cf5a12a2469d3f792af21a73fcdd')
options=('staticlibs' 'debug')

# NINJAFLAGS is an env var used to pass commandline options to ninja
# NOTE: It's your responbility to validate the value of $NINJAFLAGS. If unsure, don't set it.

_python2_optimize() {
    python2 -m compileall "$@"
    python2 -O -m compileall "$@"
}

_python3_optimize() {
    python -m compileall "$@"
    python -O -m compileall "$@"
    python -OO -m compileall "$@"
}

pkgver() {
    cd llvm-project/llvm

    # This will almost match the output of `llvm-config --version` when the
    # LLVM_APPEND_VC_REV cmake flag is turned on. The only difference is
    # dash being replaced with underscore because of Pacman requirements.
    local _pkgver=$(awk -F 'MAJOR |MINOR |PATCH |)' \
                        'BEGIN { ORS="." ; i=0 } \
             /set\(LLVM_VERSION_/ { print $2 ; i++ ; if (i==2) ORS="" } \
             END { print "\n" }' \
                        CMakeLists.txt)_r$(git rev-list --count HEAD).$(git rev-parse --short HEAD)
    echo "$_pkgver"

}

prepare() {
    cd llvm-project
    # llvm-project contains a lot of stuff, remove parts that aren't used by this package
    rm -rf debuginfo-tests libclc libcxx libcxxabi llgo openmp parallel-libs pstl libc

    rm -rf "$srcdir"/fakeinstall*
}

build() {
    CFLAGS+=' -I/usr/include/tensorflow'
    CXXFLAGS+=' -I/usr/include/tensorflow'
    CFLAGS+=' -fasynchronous-unwind-tables'
    CXXFLAGS+=' -fasynchronous-unwind-tables'
    LDFLAGS+=' -fasynchronous-unwind-tables'

    #  export CC="clang -v"
    #  export CXX="clang++ -v"

    cmake \
        -B _build64 \
        -S llvm-project/llvm  -G Ninja \
        -D CMAKE_C_FLAGS="$CFLAGS" \
        -D CMAKE_CXX_FLAGS="$CXXFLAGS" \
        -D CMAKE_BUILD_TYPE=Release \
        -D CMAKE_INSTALL_PREFIX=/usr \
        -D LLVM_APPEND_VC_REV=ON \
        -D LLVM_HOST_TRIPLE="$CHOST" \
        -D LLVM_ENABLE_RTTI=ON \
        -D LLVM_ENABLE_FFI=ON \
        -D LLVM_BINUTILS_INCDIR=/usr/include \
        -D FFI_INCLUDE_DIR="$(pkg-config --variable=includedir libffi )" \
        -D LLVM_BUILD_LLVM_DYLIB=ON \
        -D LLVM_LINK_LLVM_DYLIB=ON \
        -D LLVM_INSTALL_UTILS=ON \
        -D LLVM_BUILD_DOCS=OFF \
        -D LLVM_INCLUDE_DOCS=OFF \
        -D LLVM_ENABLE_DOXYGEN=OFF \
        -D CLANG_INCLUDE_DOCS=OFF \
        -D LLVM_EXPERIMENTAL_TARGETS_TO_BUILD=AVR \
        -D LLVM_ENABLE_SPHINX=OFF \
        -D SPHINX_OUTPUT_HTML=OFF \
        -D SPHINX_WARNINGS_AS_ERRORS=OFF \
        -D POLLY_ENABLE_GPGPU_CODEGEN=ON \
        -D LLVM_VERSION_SUFFIX="" \
        -D LLDB_USE_SYSTEM_SIX=1 \
        -D LLVM_ENABLE_PROJECTS="lldb;polly;compiler-rt;lld;clang-tools-extra;clang" \
        -D TENSORFLOW_C_LIB_PATH="/usr/" # Force on

    ninja -C _build64 LLVMgold all # ocaml_doc
    DESTDIR="$srcdir/fakeinstall_64" ninja -C _build64 install

    if [[ $CARCH == x86_64 ]]; then
        export PKG_CONFIG_PATH="/usr/lib32/pkgconfig"

        LIB32_CFLAGS="$CFLAGS"" -m32"
        LIB32_CXXFLAGS="$CXXFLAGS"" -m32"

        cmake \
            -B _build32 \
            -S llvm-project/llvm \
            -G Ninja \
            -D LLVM_ENABLE_PROJECTS="clang;clang-tools-extra;compiler-rt" \
            -D CMAKE_BUILD_TYPE=Release \
            -D CMAKE_INSTALL_PREFIX=/usr \
            -D LLVM_LIBDIR_SUFFIX=32 \
            -D CMAKE_C_FLAGS="$LIB32_CFLAGS" \
            -D CMAKE_CXX_FLAGS="$LIB32_CXXFLAGS" \
            -D LLVM_TARGET_ARCH:STRING=i686 \
            -D LLVM_HOST_TRIPLE="$CHOST" \
            -D LLVM_DEFAULT_TARGET_TRIPLE="i686-pc-linux-gnu" \
            -D LLVM_BINUTILS_INCDIR=/usr/include \
            -D LLVM_BUILD_LLVM_DYLIB=ON \
            -D LLVM_LINK_LLVM_DYLIB=ON \
            -D LLVM_ENABLE_RTTI=ON \
            -D LLVM_ENABLE_FFI=ON \
            -D LLVM_BUILD_TESTS=OFF \
            -D LLVM_BUILD_DOCS=OFF \
            -D LLVM_ENABLE_SPHINX=OFF \
            -D LLVM_ENABLE_DOXYGEN=OFF \
            -D FFI_INCLUDE_DIR="$(pkg-config --variable=includedir libffi)" \
            -D LLVM_BINUTILS_INCDIR=/usr/include \
            -D LLVM_APPEND_VC_REV=ON

        ninja -C _build32 all ocaml_doc LLVMgold
        DESTDIR="$srcdir/fakeinstall_32" ninja -C _build32 install
    fi
}

_fakeinstall_64 () {
    local src f dir
    for src; do
        f="${src#fakeinstall_64/}"
        dir="$pkgdir/${f%/*}"
        install -m755 -d "$dir"
        mv -v "$src" "$dir/"
    done
}

_fakeinstall_32 () {
    local src f dir
    for src; do
        f="${src#fakeinstall_32/}"
        dir="$pkgdir/${f%/*}"
        install -m755 -d "$dir"
        mv -v "$src" "$dir/"
    done
}

package_lldb-git() {
    _pythonver="$(python -V | sed 's/.* \([0-9]\).\([0-9]\).*/\1.\2/')"

    pkgdesc="Next generation, high-performance debugger (git version)"
    url="https://lldb.llvm.org/"
    depends=("clang-git=$pkgver-$pkgrel" "llvm-libs-git=$pkgver-$pkgrel")
    optdepends=('llvm-git: for the man file')
    provides=("lldb=$pkgver")
    replaces=('lldb-svn')
    conflicts=('lldb' 'lldb-svn')
    _fakeinstall_64 fakeinstall_64/usr/bin/lldb*
    _fakeinstall_64 fakeinstall_64/usr/include/lldb
    _fakeinstall_64 fakeinstall_64/usr/lib/liblldb*
    _fakeinstall_64 fakeinstall_64/usr/lib/python"$_pythonver"/site-packages/lldb
    # _fakeinstall_64 fakeinstall_64/usr/share/man/man1/lldb*
    install -Dm644 "$srcdir"/llvm-project/lldb/LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_lld-git(){
    pkgdesc="Linker from the LLVM project"
    url="https://lld.llvm.org"
    depends=("gcc-libs" "llvm-libs-git=$pkgver-$pkgrel")
    provides=("lld=$pkgver")
    replaces=('lld-svn')
    conflicts=('lld' 'lld-svn')

    _fakeinstall_64 fakeinstall_64/usr/include/lld
    _fakeinstall_64 fakeinstall_64/usr/lib/liblld*
    _fakeinstall_64 fakeinstall_64/usr/lib/cmake/lld/
    _fakeinstall_64 fakeinstall_64/usr/bin/lld*
    _fakeinstall_64 fakeinstall_64/usr/bin/*lld
    _fakeinstall_64 fakeinstall_64/usr/bin/wasm-ld
    install -Dm644 "$srcdir"/llvm-project/lld/docs/ld.lld.1 "$pkgdir/usr/share/man/man1"
    install -Dm644 "$srcdir"/llvm-project/lld/LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_polly-git() {
    pkgdesc="Polly is a high-level loop and data-locality optimizer and optimization infrastructure for LLVM"
    url="https://polly.llvm.org/"
    depends=("llvm-git=$pkgver-$pkgrel")
    provides=("polly=$pkgver")
    replaces=('polly-svn')
    conflicts=('polly' 'polly-svn')

    _fakeinstall_64 fakeinstall_64/usr/include/polly
    _fakeinstall_64 fakeinstall_64/usr/lib/cmake/polly/
    _fakeinstall_64 fakeinstall_64/usr/lib/*Polly*
    _fakeinstall_64 fakeinstall_64/usr/lib/libGPU*
    install -Dm644 "$srcdir"/llvm-project/polly/LICENSE.txt "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
    # _fakeinstall_64 fakeinstall_64/usr/share/man/man1/polly*
}

package_compiler-rt-git() {
    pkgdesc="Compiler runtime libraries for clang (git version)"
    url="https://compiler-rt.llvm.org/"
    depends=("llvm-git=$pkgver-$pkgrel" 'gcc-libs')
    provides=("compiler-rt=$pkgver")
    replaces=('compiler-rt-svn')
    conflicts=('compiler-rt' 'compiler-rt-svn' 'clang')

    local _llvmver=$(echo "$pkgver" | grep -Po "(^[\.\d]+)")

    _fakeinstall_64 fakeinstall_64/usr/lib/clang/"$_llvmver"/lib/linux
    _fakeinstall_64 fakeinstall_64/usr/lib/clang/"$_llvmver"/include/{sanitizer,xray}
    _fakeinstall_64 fakeinstall_64/usr/lib/clang/"$_llvmver"/share

    install -Dm644 "$srcdir"/llvm-project/clang/LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_clang-git() {
    _pythonver="$(python -V | sed 's/.* \([0-9]\).\([0-9]\).*/\1.\2/')"

    pkgdesc="C language family frontend for LLVM (git version)"
    url="http://clang.llvm.org/"
    depends=("llvm-git=$pkgver-$pkgrel" 'gcc' "python" 'python2')
    optdepends=('llvm-libs: for compiling with -flto')
    provides=("clang=$pkgver" "clang-analyzer=$pkgver" "clang-tools-extra=$pkgver")
    conflicts=('clang' 'clang-svn' 'clang-analyzer' 'clang-analyzer-svn' 'clang-tools-extra' 'clang-tools-extra-svn')
    replaces=('clang-svn' 'clang-analyzer-svn' 'clang-tools-extra-svn')

    _fakeinstall_64 fakeinstall_64/usr/bin/*clang*
    _fakeinstall_64 fakeinstall_64/usr/bin/{c-index-test,diagtool,find-all-symbols}
    _fakeinstall_64 fakeinstall_64/usr/bin/{hmaptool,modularize,scan-build,scan-view}
    _fakeinstall_64 fakeinstall_64/usr/bin/pp-trace

    _fakeinstall_64 fakeinstall_64/usr/include/clang*

    _fakeinstall_64 fakeinstall_64/usr/lib/clang
    _fakeinstall_64 fakeinstall_64/usr/lib/libclang*
    _fakeinstall_64 fakeinstall_64/usr/lib/libfindAllSymbols*
    _fakeinstall_64 fakeinstall_64/usr/lib/cmake/clang

    _fakeinstall_64 fakeinstall_64/usr/libexec

    _fakeinstall_64 fakeinstall_64/usr/share/clang
    _fakeinstall_64 fakeinstall_64/usr/share/scan{-build,-view}
    # _fakeinstall_64 fakeinstall_64/usr/share/man/man1/{clang,diagtool,extraclangtools,scan-build}*

    # Remove documentation sources
    rm -rf "$pkgdir"/usr/share/doc/clang{,-tools}/html/{_sources,.buildinfo}

    # Move analyzer scripts out of /usr/libexec
    mv "$pkgdir"/usr/libexec/{ccc,c++}-analyzer "$pkgdir/usr/lib/clang/"
    rmdir "$pkgdir/usr/libexec"
    sed -i 's|libexec|lib/clang|' "$pkgdir/usr/bin/scan-build"
    # Install Python bindings
    for _py in 2.7 "$_pythonver"; do
        install -d "$pkgdir/usr/lib/python$_py/site-packages"
        cp -a "$srcdir"/llvm-project/clang/bindings/python/clang "$pkgdir/usr/lib/python$_py/site-packages/"
        _python"${_py%%.*}"_optimize "$pkgdir/usr/lib/python$_py"
    done

    # Fix shebang in Python 2 script
    sed -i '1s|/usr/bin/env python$|&2|' \
        "$pkgdir"/usr/share/clang/run-find-all-symbols.py

    # Compile Python scripts
    _python2_optimize "$pkgdir/usr/share/clang"
    _python3_optimize "$pkgdir/usr/share" -x 'clang-include-fixer|run-find-all-symbols'

    install -Dm644 "$srcdir"/llvm-project/llvm/LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_llvm-libs-git() {
    pkgdesc="LLVM runtime libraries (git version)"
    depends=('gcc-libs' 'zlib' 'libffi' 'libedit' 'libxml2' 'ncurses' 'tensorflow')
    provides=("llvm-libs=$pkgver")
    replaces=('llvm-libs-svn')
    conflicts=('llvm-libs-svn' 'llvm-libs')

    _fakeinstall_64 fakeinstall_64/usr/lib/libLLVM-*.so*
    _fakeinstall_64 fakeinstall_64//usr/lib/libLTO.so*
    _fakeinstall_64 fakeinstall_64/usr/lib/libRemarks.so.*
    _fakeinstall_64 fakeinstall_64/usr/lib/LLVMgold.so
    install -d "$pkgdir/usr/lib/bfd-plugins"
    ln -s ../LLVMgold.so "$pkgdir/usr/lib/bfd-plugins/LLVMgold.so"

    install -Dm644 "$srcdir"/llvm-project/llvm/LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_llvm-ocaml-git() {
    _ocamlver="$(ocamlc --version)"

    pkgdesc="OCaml bindings for LLVM (git version)"
    depends=("llvm-git=$pkgver-$pkgrel" "ocaml=$_ocamlver" 'ocaml-ctypes')
    provides=("llvm-ocaml=$pkgver")
    replaces=('llvm-ocaml-svn')
    conflicts=('llvm-ocaml' 'llvm-ocaml-svn')

    _fakeinstall_64 fakeinstall_64/usr/lib/ocaml
    # _fakeinstall_64 fakeinstall_64/usr/share/doc/llvm/ocaml-html

    install -Dm644 "$srcdir"/llvm-project/llvm/LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"

}

package_llvm-git() {
    pkgdesc="Collection of modular and reusable compiler and toolchain technologies (git version)"
    depends=("llvm-libs-git=$pkgver" 'perl')
    optdepends=('python-setuptools: for using lit (LLVM Integrated Tester)')
    provides=("llvm=$pkgver")
    replaces=('llvm-svn')
    conflicts=('llvm' 'llvm-svn')

    _fakeinstall_64 fakeinstall_64/usr/bin
    _fakeinstall_64 fakeinstall_64/usr/include/llvm*
    _fakeinstall_64 fakeinstall_64/usr/lib
    _fakeinstall_64 fakeinstall_64/usr/share

    # Include lit for running lit-based tests in other projects
    pushd "$srcdir"/llvm-project/llvm/utils/lit
    python3 setup.py install --root="$pkgdir" -O1
    popd

    cp "$pkgdir"/usr/bin/lit "$pkgdir"/usr/bin/llvm-lit
    # Remove documentation sources
    rm -rf "$pkgdir"/usr/share/doc/llvm/html/{_sources,.buildinfo}
    # Remove libs which conflict with llvm-libs
    rm -f "$pkgdir"/usr/lib/{libLLVM,libLTO,LLVMgold,libRemarks}.so
    rm -f "$pkgdir"/usr/lib/python3.8/site-packages/six.py

    if [[ $CARCH == x86_64 ]]; then
        # Needed for multilib (https://bugs.archlinux.org/task/29951)
        # Header stub is taken from Fedora
        mv "$pkgdir/usr/include/llvm/Config/llvm-config"{,-64}.h
        cp "$srcdir/llvm-config.h" "$pkgdir/usr/include/llvm/Config/llvm-config.h"
    fi

    # make sure there are no files left to install
    find fakeinstall_64 -depth -print0 | xargs -0 rmdir

    install -Dm644 "$srcdir"/llvm-project/llvm/LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_lib32-llvm-libs-git() {
    pkgdesc='Low Level Virtual Machine library (runtime library)(32-bit)(git version)'
    depends=('lib32-libffi' 'lib32-zlib' 'lib32-libxml2' 'lib32-gcc-libs')
    provides=("lib32-llvm-libs=$pkgver")
    replaces=('lib32-llvm-libs-svn')
    conflicts=('lib32-llvm-libs-svn' 'lib32-llvm-libs')

    _fakeinstall_32 fakeinstall_32/usr/lib32/libLLVM-*.so
    _fakeinstall_32 fakeinstall_32/usr/lib32/libRemarks.so.*
    _fakeinstall_32 fakeinstall_32/usr/lib32/LLVMgold.so
    install -d "$pkgdir/usr/lib32/bfd-plugins"
    ln -s ../LLVMgold.so "$pkgdir/usr/lib32/bfd-plugins/LLVMgold.so"
    install -Dm644 "$srcdir"/llvm-project/llvm/LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_lib32-clang-git() {
    pkgdesc="C language family frontend for LLVM (32-bit)"
    depends=("lib32-llvm-libs-git=$pkgver-$pkgrel" "gcc-multilib")
    provides=("lib32-clang=$pkgver")
    replaces=('lib32-clang-svn')
    conflicts=('lib32-clang' 'lib32-clang-svn')

    _fakeinstall_32 fakeinstall_32/usr/lib32/clang
    _fakeinstall_32 fakeinstall_32/usr/lib32/cmake/clang/
    _fakeinstall_32 fakeinstall_32/usr/lib32/libclang*
    install -Dm644 "$srcdir"/llvm-project/clang/LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_lib32-llvm-git() {
    pkgdesc='Low Level Virtual Machine (32-bit)(git version)'
    depends=("lib32-llvm-libs-git=$pkgver-$pkgrel" 'llvm-git')
    provides=("lib32-llvm=$pkgver")
    replaces=('lib32-llvm-svn')
    conflicts=('lib32-llvm' 'lib32-llvm-svn')

    _fakeinstall_32 fakeinstall_32/usr/lib32
    # Remove libs which conflict with lib32-llvm-libs
    rm -f "$pkgdir"/usr/lib32/{libLLVM,libLTO,LLVMgold,libRemarks}.so

    _fakeinstall_32 fakeinstall_32/usr/bin/llvm-config
    mv -v "$pkgdir"/usr/bin/llvm-config "$pkgdir"/usr/lib32/llvm-config

    _fakeinstall_32 fakeinstall_32/usr/include/llvm/Config/llvm-config.h
    mv "$pkgdir"/usr/include/llvm/Config/llvm-config.h "$pkgdir"/usr/lib32/llvm-config-32.h

    rm -rf "$pkgdir"/usr/{bin/*,include,share/{doc,man,llvm,opt-viewer}}

    install -d "$pkgdir/usr/include/llvm/Config"
    mv "$pkgdir/usr/lib32/llvm-config-32.h" "$pkgdir/usr/include/llvm/Config/"

    mv -v "$pkgdir"/usr/lib32/llvm-config "$pkgdir"/usr/bin/llvm-config32

    # make sure there are no files left to install
    find fakeinstall_32 -depth -print0 | xargs -0 rm -r

    install -Dm644 "$srcdir"/llvm-project/llvm/LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
