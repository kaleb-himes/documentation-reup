# Building wolfEngine

## Getting wolfEngine Source Code

The most recent version of wolfEngine can be obtained directly from wolfSSL Inc. Contact [facts@wolfssl.com](mailto:facts@wolfssl.com) for more information.

## wolfEngine Package Structure

The general wolfEngine package is structured as follows:

```
certs/                        (Test certificates and keys, used with unit tests)
engine.conf              (Example OpenSSL config file using wolfEngine)
include/
    wolfengine/          (wolfEngine header files)
openssl_patches/
    1.0.2h/tests/        (patches for OpenSSL 1.0.2h test apps)
    1.1.1b/tests/        (patches for OpenSSL 1.1.1b test apps)
scripts/                      (wolfEngine test scripts)
src/                            (wolfEngine source files)
test/                           (wolfEngine test files)
user_settings.h         (EXAMPLE user_settings.h)
```
## OpenSSL Version Caveats

Depending on the version of OpenSSL being used with wolfEngine, there are several algorithms support caveats, including:

- SHA-3 support is only available with OpenSSL versions 1.1.1+
- EC_KEY_METHOD is only available with OpenSSL versions 1.1.1+

## Building on *nix

### Building OpenSSL

A pre-installed version of OpenSSL may be used with wolfEngine (barring algorithm caveats above), or OpenSSL can be recompiled for use with wolfEngine. General instructions for compiling OpenSSL on *nix-like platforms will be similar to the following. For complete and comprehensive OpenSSL build instructions, reference the OpenSSL INSTALL file and documentation.
```
git clone https://github.com/openssl/openssl.git
cd openssl
./config no-fips -shared
make
sudo make install
```

### Building wolfSSL

If using a FIPS-validated version of wolfSSL with wolfEngine, follow the build instructions provided with your specific FIPS validated source bundle and Security Policy. In addition to the correct “--enable-fips” configure option, wolfEngine will need wolfSSL to be compiled with “ **WOLFSSL_PUBLIC_MP** ” defined. For example, building the “wolfCrypt Linux FIPSv2” bundle on Linux:
```
cd wolfssl-X.X.X-commercial-fips-linuxv
./configure **--enable-fips=v2 CFLAGS=”-DWOLFSSL_PUBLIC_MP”**
make
./wolfcrypt/test/testwolfcrypt
< modify fips_test.c using verifyCore hash output from testwolfcrypt
>
make
./wolfcrypt/test/testwolfcrypt
< all algorithms should PASS >
sudo make install
```

To build non-FIPS wolfSSL for use with wolfEngine:
```
cd wolfssl-X.X.X

./configure --enable-cmac --enable-keygen --enable-sha --enable-des
--enable-aesctr --enable-aesccm --enable-x963kdf
CPPFLAGS="-DHAVE_AES_ECB -DWOLFSSL_AES_DIRECT -DWC_RSA_NO_PADDING
-DWOLFSSL_PUBLIC_MP -DECC_MIN_KEY_SZ=192 -DWOLFSSL_PSS_LONG_SALT
-DWOLFSSL_PSS_SALT_LEN_DISCOVER"

make
sudo make install
```

If cloning wolfSSL from GitHub, you will need to run the `autogen.sh` script before running `./configure`. This will generate the configure script:
```
./autogen.sh
```

### Building wolfEngine
When building wolfEngine on Linux or other *nix-like systems, use the autoconf system. To configure and compile wolfEngine run the following two commands from the wolfEngine root directory:
```
./configure
make
```

If building wolfEngine from GitHub, run autogen.sh before running configure:
```
./autogen.sh
```

Any number of build options can be appended to ./configure. For a list of available build options, please reference the “Build Options” section below or run the following command to see a list of available build options to pass to the ./configure script:
```
./configure  --help
```

wolfEngine will use the system default OpenSSL library installation unless changed with the “--with-openssl” configure option:
```
./configure --with-openssl=/usr/local/ssl
```

The custom OpenSSL installation location may also need to be added to your library search path. On Linux, `LD_LIBRARY_PATH` is used:
```
export LD_LIBRARY_PATH=/usr/local/ssl:$LD_LIBRARY_PATH
```

To build then install wolfEngine, run:
```
make
make install
```

You may need superuser privileges to install, in which case precede the command with sudo:
```
sudo make install
```

To test the build, run the built-in tests from the root wolfEngine directory:
```
./test/unit.test
```

Or use autoconf to run the tests:
```
make check
```

If you get an error like `error while loading shared libraries: libssl.so.3` then the library cannot be found. Use the `LD_LIBRARY_PATH` environment variable as described in the section above.

## Building on WinCE

For full wolfEngine compatibility, ensure you have the following flags in your `user_settings.h` file for wolfCrypt:
```
#define WOLFSSL_CMAC
#define WOLFSSL_KEY_GEN
#undef NO_SHA
#undef NO_DES
#define WOLFSSL_AES_COUNTER
#define HAVE_AESCCM
#define HAVE_AES_ECB
#define WOLFSSL_AES_DIRECT
#define WC_RSA_NO_PADDING
#define WOLFSSL_PUBLIC_MP
#define ECC_MIN_KEY_SZ=192
```

Add wolfEngine flags to your `user_settings.h` file depending on which algorithms and features you want to use. You can find a list of wolfEngine user settings flags in the `user_settings.h` file in wolfEngine’s directory.

Build wcecompat, wolfCrypt and OpenSSL for Windows CE, and keep track of their paths.

In the wolfEngine directory, open the sources file and change the OpenSSL, wolfCrypt, and `user_settings.h` paths to the directories you are using. You will need to update the paths in the INCLUDES and TARGETLIBS sections.

Load the wolfEngine project in Visual Studio. Include either `bench.c`, or `unit.h` and `unit.c` depending on if you want to run the benchmark or unit tests.

Build the project, and you will end up with a wolfEngine.exe executable. You can run this executable with ` --help` to see a full list of options. You may need to run it with the  `--static` flag to use wolfEngine as a static engine.

## Build Options (./configure Options)

The following are options which may be appended to the `./configure` script to customize how the wolfEngine library is built.

By default, wolfEngine only builds a shared library, with building of a static library disabled. This speeds up build times by a factor of two. Either mode can be explicitly disabled or enabled if desired.

| Option               | Default Value | Description |
| :---------               | :---------------: | :-------------- |
|  --enable-static  | **Disabled** | Build static libraries   |
| --enable-shared | Enabled | Build shared libraries |
|  --enable-debug | **Disabled** | Enable wolfEngine debugging support |
|  --enable-coverage | **Disabled** | Build to generate code coverage stats |
| --enable-usersettings | **Disabled** | Use your own user_settings.h and do not add Makefile CFLAGS |
| --enable-dynamic-engine | Enabled | Enable loading wolfEngine as a dynamic engine |
| --enable-singlethreaded | **Disabled** | Enable wolfEngine single threaded |
| --enable-digest | Enabled | Enable use of wc_Hash API for digesting data |
| --enable-sha | Enabled | Enable SHA-1 |
| --enable-sha224 | Enabled | Enable SHA2-224 |
| --enable-sha256 | Enabled | Enable SHA2-256 |
| --enable-sha384 | Enabled | Enable SHA2-384 |
| --enable-sha512 | Enabled | Enable SHA2-512 |
| --enable-sha3 | Enabled | Enable SHA3 |
| --enable-sha3-224 | Enabled | Enable SHA3-224 |
| --enable-sha3-256 | Enabled | Enable SHA3-256 |
| --enable-sha3-384 | Enabled | Enable SHA3-384 |
| --enable-sha3-512 | Enabled | Enable SHA3-512 |
| --enable-cmac | Enabled | Enable CMAC |
| --enable-hmac | Enabled | Enable HMAC |
| --enable-des3cbc| Enabled | Enable 3DES-CBC |
| --enable-aesecb | Enabled | Enable AES-ECB |
| --enable-aescbc | Enabled | Enable AES-CBC |
| --enable-aesctr | Enabled | Enable AES-CTR |
| --enable-aesgcm | **Disabled** | Enable AES-GCM |
| --enable-aesccm | **Disabled** | Enable AES-CCM |
| --enable-rand | Enabled | Enable RAND |
| --enable-rsa | Enabled | Enable RSA |
| --enable-dh | Enabled | Enable DH |
| --enable-evp-pkey | Enabled | Enable EVP_PKEY APIs |
| --enable-ecc | Enabled | Enable ECC |
| --enable-ec-key | Enabled | Enable ECC using EC_KEY |
| --enable-ecdsa | Enabled | Enable ECDSA |
| --enable-ecdh | Enabled | Enable ECDH |
| --enable-eckg | Enabled | Enable EC Key Generation |
| --enable-p192 | Enabled | Enable EC Curve P-192 |
| --enable-p224 | Enabled | Enable EC Curve P-224 |
| --enable-p256 | Enabled | Enable EC Curve P-256 |
| --enable-p384 | Enabled | Enable EC Curve P-384 |
| --enable-p521 | Enabled | Enable EC Curve P-521 |
| --with-openssl=DIR |   | OpenSSL installation location to link against. If not set, use the system default library and include paths. |
| --enable-openssh | **Disabled** | Enables use with openssh |

## Build Defines

wolfEngine exposes several preprocessor defines that allow users to configure how wolfEngine is built. These are described in the table below.

| Define                                                      | Description |
| :---------------------------------------------- | :-------------- |
| WOLFENGINE_DEBUG | Build wolfEngine with debug symbols, optimization level, and debug logging. |
| WE_NO_DYNAMIC_ENGINE |  Do not build wolfEngine with dynamic engine support. Dynamic engines are ones that can be loaded into OpenSSL at runtime. |
| WE_SINGLE_THREADED | Build wolfEngine in single-threaded mode. This removes the need for locking around global resources used internally. |
| WE_USE_HASH | Enable digest algorithms using the wc_Hash API. |
| WE_HAVE_SHA1 | Enable SHA-1 digest algorithm. |
| WE_HAVE_SHA224 | Enable SHA-2 digest algorithm with digest size 224. |
| WE_HAVE_SHA256 | Enable SHA-2 digest algorithm with digest size 256. |
| WE_HAVE_SHA384 | Enable SHA-2 digest algorithm with digest size 384. |
| WE_HAVE_SHA512| Enable SHA-2 digest algorithm with digest size 512. |
| WE_SHA1_DIRECT | Enable the SHA-1 digest algorithm using the wc_Sha API. Incompatible with WE_USE_HASH. |
| WE_SHA224_DIRECT | Enable the SHA-2 224 digest algorithm using the wc_Sha224 API. Incompatible with WE_USE_HASH. |
| WE_SHA256_DIRECT | Enable the SHA-2 256 digest algorithm using the wc_Sha256 API. Incompatible with WE_USE_HASH. |
| WE_HAVE_SHA3_224 | Enable SHA-3 digest algorithm with digest size 224. Not available in OpenSSL 1.0.2. |
| WE_HAVE_SHA3_256 | Enable SHA-3 digest algorithm with digest size 256. Not available in OpenSSL 1.0.2. |
| WE_HAVE_SHA3_384 | Enable SHA-3 digest algorithm with digest size 384. Not available in OpenSSL 1.0.2. |
| WE_HAVE_SHA3_512 | Enable SHA-3 digest algorithm with digest size 512. Not available in OpenSSL 1.0.2. |
| WE_HAVE_EVP_PKEY | Enable functionality that uses the EVP_PKEY API. This includes things like RSA, DH, etc. |
| WE_HAVE_CMAC | Enable CMAC algorithm. |
| WE_HAVE_HMAC | Enable HMAC algorithm. |
| WE_HAVE_DES3CBC | Enable DES3-CBC algorithm. |
|WE_HAVE_AESECB | Enable AES algorithm with ECB mode. |
| WE_HAVE_AESCBC | Enable AES algorithm with CBC mode. |
| WE_HAVE_AESCTR | Enable AES algorithm with countee mode. |
| WE_HAVE_AESGCM | Enable AES algorithm with GCM mode. |
| WE_HAVE_AESCCM |Enable AES algorithm with CCM mode. |
| WE_HAVE_RANDOM | Enable wolfCrypt random implementation. |
| WE_HAVE_RSA | Enable RSA operations (e.g. sign, verify, key generation, etc.). |
| WE_HAVE_DH | Enable Diffie-Hellman operations (e.g. key generation, shared secret computation, etc.). |
| WE_HAVE_ECC | Enable support for elliptic curve cryptography. |
| WE_HAVE_EC_KEY | Enable support for EC_KEY_METHOD. Not available in OpenSSL 1.0.2. |
| WE_HAVE_ECDSA | Enable ECDSA algorithm. |
| WE_HAVE_ECDH | Enable EC Diffie-Hellman operations. |
| WE_HAVE_ECKEYGEN | Enable EC key generation. |
| WE_HAVE_EC_P192 | Enable EC curve P192. |
| WE_HAVE_EC_P224 | Enable EC curve P224. |
| WE_HAVE_EC_P256 | Enable EC curve P256. |
| WE_HAVE_EC_P384 | Enable EC curve P384. |
| WE_HAVE_EC_P512 | Enable EC curve P512. |
| WE_HAVE_DIGEST | Compile code in benchmark program and unit tests for use with digest algorithms. |
| WOLFENGINE_USER_SETTINGS | Read user-specified defines from user_settings.h. |

