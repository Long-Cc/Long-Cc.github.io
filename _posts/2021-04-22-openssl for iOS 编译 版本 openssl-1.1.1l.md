---
layout:     post
title:      openssl for iOS 编译 版本 openssl-1.1.1
subtitle:   版本 openssl-1.1.1 
date:       2021-09-25
author:     Belial
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - iOS
---

7.76.1适用，后面版本报错

```shell
#!/bin/bash -euo pipefail

readonly XCODE_DEV="$(xcode-select -p)"
export DEVROOT="${XCODE_DEV}/Toolchains/XcodeDefault.xctoolchain"
DFT_DIST_DIR=${HOME}/Desktop/libcurl-ios-dist
DIST_DIR=${DIST_DIR:-$DFT_DIST_DIR}

function check_curl_ver() {
echo "#include \"include/curl/curlver.h\"
#if LIBCURL_VERSION_MAJOR < 7 || LIBCURL_VERSION_MINOR < 55
#error Required curl 7.40.0+; See http://curl.haxx.se/docs/adv_20150108A.html
#error Supported minimal version is 7.55.0 for header file changes, see Issue #12 (https://github.com/sinofool/build-libcurl-ios/issues/12)
#endif"|gcc -c -o /dev/null -xc -||exit 9
}

function build_for_arch() {
  ARCH=$1
  HOST=$2
  SYSROOT=$3
  PREFIX=$4
  IPHONEOS_DEPLOYMENT_TARGET="6.0"
  export PATH="${DEVROOT}/usr/bin/:${PATH}"
  export CFLAGS="-DCURL_BUILD_IOS -arch ${ARCH} -pipe -Os -gdwarf-2 -isysroot ${SYSROOT} -miphoneos-version-min=${IPHONEOS_DEPLOYMENT_TARGET} -fembed-bitcode"
  export LDFLAGS="-arch ${ARCH} -isysroot ${SYSROOT}"
  ./configure --disable-shared --without-zlib --enable-static --enable-ipv6 ${SSL_FLAG} --host="${HOST}" --prefix=${PREFIX} && make -j8 && make install
}

if [ "${1:-''}" == "openssl" ]
then
  if [ ! -d ${HOME}/Desktop/openssl-ios-dist ]
  then
    echo "Please use https://github.com/sinofool/build-openssl-ios/ to build OpenSSL for iOS first"
    exit 8
  fi
  export SSL_FLAG=--with-ssl=${HOME}/Desktop/openssl-ios-dist
else
  check_curl_ver
  export SSL_FLAG=--with-darwinssl
fi

TMP_DIR=${HOME}/Desktop/tmp/build_libcurl_$$

build_for_arch i386 i386-apple-darwin ${XCODE_DEV}/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk ${TMP_DIR}/i386 || exit 1
build_for_arch x86_64 x86_64-apple-darwin ${XCODE_DEV}/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk ${TMP_DIR}/x86_64 || exit 2
build_for_arch arm64 arm-apple-darwin ${XCODE_DEV}/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk ${TMP_DIR}/arm64 || exit 3
build_for_arch armv7s armv7s-apple-darwin ${XCODE_DEV}/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk ${TMP_DIR}/armv7s || exit 4
build_for_arch armv7 armv7-apple-darwin ${XCODE_DEV}/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk ${TMP_DIR}/armv7 || exit 5

mkdir -p ${TMP_DIR}/lib/
${DEVROOT}/usr/bin/lipo \
  -arch x86_64 ${TMP_DIR}/x86_64/lib/libcurl.a \
  -arch armv7 ${TMP_DIR}/armv7/lib/libcurl.a \
  -arch armv7s ${TMP_DIR}/armv7s/lib/libcurl.a \
  -arch arm64 ${TMP_DIR}/arm64/lib/libcurl.a \
  -arch i386 ${TMP_DIR}/i386/lib/libcurl.a \
  -output ${TMP_DIR}/lib/libcurl.a -create

cp -r ${TMP_DIR}/arm64/include ${TMP_DIR}/

mkdir -p ${DIST_DIR}
cp -r ${TMP_DIR}/include ${TMP_DIR}/lib ${DIST_DIR}

# lipo -create /Users/clv/Downloads/gitCode/curl/build_libcurl_22752/arm64/lib/libcurl.a \
# /Users/clv/Downloads/gitCode/curl/build_libcurl_22752/armv7/lib/libcurl.a \
# /Users/clv/Downloads/gitCode/curl/build_libcurl_22752/armv7s/lib/libcurl.a \
# /Users/clv/Downloads/gitCode/curl/build_libcurl_22752/i386/lib/libcurl.a \
# /Users/clv/Downloads/gitCode/curl/build_libcurl_22752/x86_64/lib/libcurl.a \
# -output /Users/clv/Downloads/gitCode/curl/build_libcurl_22752/lib/libcurl.a
```

openssl for iOS 编译 版本 openssl-1.1.1l<br />[openssl-build.sh](https://www.yuque.com/attachments/yuque/0/2023/sh/12433510/1675400729951-a7c4358c-cf10-4356-b540-6e65c8ab0117.sh?_lake_card=%7B%22src%22%3A%22https%3A%2F%2Fwww.yuque.com%2Fattachments%2Fyuque%2F0%2F2023%2Fsh%2F12433510%2F1675400729951-a7c4358c-cf10-4356-b540-6e65c8ab0117.sh%22%2C%22name%22%3A%22openssl-build.sh%22%2C%22size%22%3A474%2C%22ext%22%3A%22sh%22%2C%22source%22%3A%22%22%2C%22status%22%3A%22done%22%2C%22download%22%3Atrue%2C%22taskId%22%3A%22u57c8f326-de16-42ee-aec6-0436472643a%22%2C%22taskType%22%3A%22transfer%22%2C%22type%22%3A%22text%2Fx-sh%22%2C%22mode%22%3A%22title%22%2C%22id%22%3A%22udd12dc4e%22%2C%22card%22%3A%22file%22%7D)
[openssl-build-phase2.sh](https://www.yuque.com/attachments/yuque/0/2023/sh/12433510/1675400729952-a4d672cc-5c43-431d-a131-2b6e135090d0.sh?_lake_card=%7B%22src%22%3A%22https%3A%2F%2Fwww.yuque.com%2Fattachments%2Fyuque%2F0%2F2023%2Fsh%2F12433510%2F1675400729952-a4d672cc-5c43-431d-a131-2b6e135090d0.sh%22%2C%22name%22%3A%22openssl-build-phase2.sh%22%2C%22size%22%3A12506%2C%22ext%22%3A%22sh%22%2C%22source%22%3A%22%22%2C%22status%22%3A%22done%22%2C%22download%22%3Atrue%2C%22taskId%22%3A%22u74c0e851-d2b6-4f5d-8263-3539452f4b0%22%2C%22taskType%22%3A%22transfer%22%2C%22type%22%3A%22text%2Fx-sh%22%2C%22mode%22%3A%22title%22%2C%22id%22%3A%22u0ef8afad%22%2C%22card%22%3A%22file%22%7D)
[openssl-build-phase1.sh](https://www.yuque.com/attachments/yuque/0/2023/sh/12433510/1675400729974-f8623638-51db-4187-86ca-108aa84cae9d.sh?_lake_card=%7B%22src%22%3A%22https%3A%2F%2Fwww.yuque.com%2Fattachments%2Fyuque%2F0%2F2023%2Fsh%2F12433510%2F1675400729974-f8623638-51db-4187-86ca-108aa84cae9d.sh%22%2C%22name%22%3A%22openssl-build-phase1.sh%22%2C%22size%22%3A18975%2C%22ext%22%3A%22sh%22%2C%22source%22%3A%22%22%2C%22status%22%3A%22done%22%2C%22download%22%3Atrue%2C%22taskId%22%3A%22ud7a2a075-9009-4c34-bf91-95ca9099f37%22%2C%22taskType%22%3A%22transfer%22%2C%22type%22%3A%22text%2Fx-sh%22%2C%22mode%22%3A%22title%22%2C%22id%22%3A%22u4f85437c%22%2C%22card%22%3A%22file%22%7D)
