# wolfEngine のビルド

## wolfEngine のソースコードの取得

wolfEngine の最新バージョンは、wolfSSL Inc. から直接入手できます。詳細については、[facts@wolfssl.com](mailto:facts@wolfssl.com) までお問い合わせください。


## wolfEngine パッケージ構成

一般的な wolfEngine パッケージは次のように構成されています:

```
certs/               (ユニットテストで使用されるテスト用証明書、鍵)
engine.conf        　(wolfEngineを使用する場合のOpenSSLコンフィギュレーションファイルサンプル）
include/
    wolfengine/      (wolfEngineヘッダーファイル)
openssl_patches/
    1.0.2h/tests/    (OpenSSL 1.0.2h テストアプリ用パッチ)
    1.1.1b/tests/    (OpenSSL 1.1.1b テストアプリ用パッチ)
scripts/             (wolfEngine テストスクリプト)
src/                 (wolfEngine ソースファイル)
test/                (wolfEngine テストファイル)
user_settings.h      (user_settings.hサンプル)
```
## OpenSSL のバージョンに関する注意事項

wolfEngine で使用されている OpenSSL のバージョンに応じて、次のようないくつかのアルゴリズム サポートの注意事項があります:

- SHA-3 はOpenSSL versions 1.1.1以降でサポートされます
- EC_KEY_METHOD はOpenSSL versions 1.1.1以降でサポートされます

## *nix 上でのビルド


### OpenSSLをビルド

OpenSSL のプリインストールされたバージョンを wolfEngine で使用することも (上記のアルゴリズムの警告を除いて)、または OpenSSL を再コンパイルして wolfEngine で使用することもできます。 *nix のようなプラットフォームで OpenSSL をコンパイルするための一般的な手順は、次のようになります。 完全で包括的な OpenSSL のビルド手順については、OpenSSL INSTALL ファイルとドキュメントを参照してください。


```
git clone https://github.com/openssl/openssl.git
cd openssl
./config no-fips -shared
make
sudo make install
```

### wolfSSLをビルド

wolfEngine で wolfSSL の FIPS 検証済みバージョンを使用する場合は、特定の FIPS 検証済みソース バンドルとセキュリティ ポリシーで提供されるビルド手順に従ってください。 正しい「--enable-fips」設定オプションに加えて、wolfEngine は"**WOLFSSL_PUBLIC_MP**"が定義された状態で wolfSSL をコンパイルする必要があります。 たとえば、Linux で「wolfCrypt Linux FIPSv2」バンドルをビルドする場合:

```
cd wolfssl-X.X.X-commercial-fips-linuxv
./configure **--enable-fips=v2 CFLAGS=”-DWOLFSSL_PUBLIC_MP”**
make
./wolfcrypt/test/testwolfcrypt
#--< fips_test.c 内の verifyCore を hash output from testwolfcryptスクリプトが出力するハッシュ値に更新してください >--

make
./wolfcrypt/test/testwolfcrypt

#--< 全アルゴリズムでパスするはずです>--

sudo make install
```

wolfEngine で使用する非 FIPS wolfSSL をビルドするには:
```
cd wolfssl-X.X.X

./configure --enable-cmac --enable-keygen --enable-sha --enable-des --enable-aesctr --enable-aesccm --enable-x963kdf CPPFLAGS="-DHAVE_AES_ECB -DWOLFSSL_AES_DIRECT -DWC_RSA_NO_PADDING -DWOLFSSL_PUBLIC_MP -DECC_MIN_KEY_SZ=192 -DWOLFSSL_PSS_LONG_SALT -DWOLFSSL_PSS_SALT_LEN_DISCOVER"

make
sudo make install
```

GitHub から wolfSSL をクローンする場合、`./configure` を実行する前に `autogen.sh` スクリプトを実行する必要があります。 これにより、configure スクリプトが生成されます:

```
./autogen.sh
```

### wolfEngineをビルド

Linux またはその他の *nix ライクなシステムで wolfEngine をビルドする場合は、autoconf システムを使用してください。 wolfEngine を構成およびコンパイルするには、wolfEngine ルート ディレクトリから次の 2 つのコマンドを実行します：


```
./configure
make
```

GitHub から wolfEngine を取得してビルドする場合は、configure を実行する前に autogen.sh を実行します:
```
./autogen.sh
```

任意の数のビルドオプションを ./configure に追加できます。 利用可能なビルド オプションのリストについては、以下の「ビルド オプション」セクションを参照するか、次のコマンドを実行して、./configure スクリプトに渡す利用可能なビルド オプションのリストを表示してください：


```
./configure  --help
```

"--with-openssl"オプションで変更しない限り、wolfEngine はシステムのデフォルトの OpenSSL ライブラリのインストールを使用します:


```
./configure --with-openssl=/usr/local/ssl
```
カスタム OpenSSL のインストール場所も、ライブラリ検索パスに追加する必要となる場合があります。Linux では、`LD_LIBRARY_PATH` が使用されます:

```
export LD_LIBRARY_PATH=/usr/local/ssl:$LD_LIBRARY_PATH
```

wolfEngine をビルドしてインストールするには、以下を実行します:

```
make
make install
```

インストールにはスーパーユーザー権限が必要な場合があります。その場合は、コマンドの前に sudo を付けます:

```
sudo make install
```

ビルドをテストするには、ルート wolfEngine ディレクトリからビルトインテストを実行します:

```
./test/unit.test
```

または autoconf を使用してテストを実行します:

```
make check
```

`error while loading shared libraries: libssl.so.3` のようなエラーが発生した場合は、ライブラリが見つからなかった為です。上記のセクションで説明したように、`LD_LIBRARY_PATH` 環境変数を使用します：


## WinCE上でのビルド

wolfEngine との完全な互換性のために、wolfCrypt の `user_settings.h` ファイルに以下の定義があることを確認してください:


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

使用するアルゴリズムと機能に応じて、`user_settings.h` ファイルに wolfEngine フラグを追加します。 wolfEngine のディレクトリにある `user_settings.h` ファイルで、wolfEngine ユーザー設定フラグが参照できます。

Windows CE 用の wcecompat、wolfCrypt、および OpenSSL をビルドし、それらのパスを参照できるようにします。

wolfEngine ディレクトリでソースファイルを開き、OpenSSL、wolfCrypt、および `user_settings.h` パスを使用しているディレクトリに変更します。 INCLUDES セクションと TARGETLIBS セクションのパスを更新する必要があります。

Visual Studio で wolfEngine プロジェクトをロードします。 ベンチマークまたは単体テストを実行するかどうかに応じて、「bench.c」、または「unit.h」と「unit.c」のいずれかを含めます。

プロジェクトをビルドすると、wolfEngine.exe 実行可能ファイルが作成されます。 この実行可能ファイルを --help で実行すると、オプションの完全なリストが表示されます。 wolfEngine を静的エンジンとして使用するには、`--static` フラグを付けて実行する必要があります。


## ビルドオプション (./configure に指定するオプション)

以下は、wolfEngine ライブラリの構築方法をカスタマイズする目的で `./configure` スクリプトに追加できるオプションです。

デフォルトでは、wolfEngine は共有ライブラリのみを構築し、静的ライブラリの構築は無効になっています。 これにより、ビルド時間が 2 倍速くなります。 どちらのモードも、必要に応じて明示的に無効または有効にすることができます。


| オプション   | デフォルト | 意味 |
| :--------- | :---------------: | :-------------- |
| --enable-static   | **無効** | スタティックライブラリとしてビルド   |
| --enable-shared   | 有効     | 共有ライブラリとしてビルド |
| --enable-debug    | **無効** | wolfEngineのデバッグ出力を有効にする |
| --enable-coverage | **無効** | コードカバレッジレポートを作成する用ビルド |
| --enable-usersettings | **無効** | user_settings.h を使用しMakefileの CFLAGSを使用しない |
| --enable-dynamic-engine | 有効 | wolfEngine をダイナミックエンジンとしてロードする |
| --enable-singlethreaded | **無効** |  wolfEngineをシングルスレッド環境で使用する |
| --enable-digest | 有効 |  ダイジェストの生成にwc_Hash APIを使用する |
| --enable-sha | 有効 |  SHA-1 を有効にする|
| --enable-sha224 | 有効 |  SHA2-224 を有効にする|
| --enable-sha256 | 有効 |  SHA2-256 を有効にする|
| --enable-sha384 | 有効 |  SHA2-384 を有効にする|
| --enable-sha512 | 有効 |  SHA2-512 を有効にする|
| --enable-sha3 | 有効 |  SHA3 を有効にする|
| --enable-sha3-224 | 有効 |  SHA3-224 を有効にする|
| --enable-sha3-256 | 有効 |  SHA3-256 を有効にする|
| --enable-sha3-384 | 有効 |  SHA3-384 を有効にする|
| --enable-sha3-512 | 有効 |  SHA3-512 を有効にする|
| --enable-cmac | 有効 |  CMAC を有効にする|
| --enable-hmac | 有効 |  HMAC を有効にする|
| --enable-des3cbc| 有効 |  3DES-CBC を有効にする|
| --enable-aesecb | 有効 |  AES-ECB を有効にする|
| --enable-aescbc | 有効 |  AES-CBC を有効にする|
| --enable-aesctr | 有効 |  AES-CTR を有効にする|
| --enable-aesgcm | **無効** |  AES-GCM を有効にする|
| --enable-aesccm | **無効** |  AES-CCM を有効にする|
| --enable-rand | 有効 |  RAND を有効にする|
| --enable-rsa | 有効 |  RSA を有効にする|
| --enable-dh | 有効 |  DH を有効にする|
| --enable-evp-pkey | 有効 |  EVP_PKEY APIs を有効にする|
| --enable-ec-key | 有効 |  ECC using EC_KEY を有効にする|
| --enable-ecdsa | 有効 |  ECDSA を有効にする|
| --enable-ecdh | 有効 |  ECDH を有効にする|
| --enable-eckg | 有効 |  EC Key Generation を有効にする|
| --enable-p192 | 有効 |  EC Curve P-192 を有効にする|
| --enable-p224 | 有効 |  EC Curve P-224 を有効にする|
| --enable-p256 | 有効 |  EC Curve P-256 を有効にする|
| --enable-p384 | 有効 |  EC Curve P-384 を有効にする|
| --enable-p521 | 有効 |  EC Curve P-521 を有効にする|
| --with-openssl=DIR |   | OpenSSLのインストール場所を指定。指定しない場合はシステムのデフォルトライブラリパスとインクルードパスが使われます。|
| --enable-openssh | **無効** | OpenSSHと組み合わせて使用する |

## ビルド用マクロ定義

wolfEngine は、ユーザーが wolfEngine の構築方法を設定できるようにするいくつかのプリプロセッサマクロを公開しています。 これらについては、次の表で説明します：


| マクロ定義  | 意味 |
| :---------------------------------------------- | :-------------- |
| WOLFENGINE_DEBUG | デバッグ シンボル、最適化レベル、デバッグ ロギングを使用して wolfEngine をビルドします |
| WE_NO_DYNAMIC_ENGINE |  wolfEngineをダイナミックエンジンとしてビルドしない。ダイナミックエンジンとはOpenSSLが実行時に動的にロードするエンジンです。 |
| WE_SINGLE_THREADED | wolfEngineをシングルスレッドモードでビルドする。このマクロ定義によりグローバルリソースの使用の排他用に内部的に使用するロック機構を取り除きます。|
| WE_USE_HASH | ハッシュアルゴリズムを wc_Hash APIを使って有効にする |
| WE_HAVE_SHA1   |  SHA-1 を有効にする |
| WE_HAVE_SHA224 |  SHA-2 224を有効にする |
| WE_HAVE_SHA256 |  SHA-2 256を有効にする |
| WE_HAVE_SHA384 |  SHA-2 384を有効にする |
| WE_HAVE_SHA512 |  SHA-2 512を有効にする |
| WE_SHA1_DIRECT | SHA-1 をwc_Sha APIを使って有効にする。WE_USE_HASHとはコンパチブルではない |
| WE_SHA224_DIRECT |  SHA-2 224 を wc_Sha224 APIを使って有効にする。WE_USE_HASHとはコンパチブルではない |
| WE_SHA256_DIRECT |  SHA-2 256 を wc_Sha256 APIを使って有効にする。WE_USE_HASHとはコンパチブルではない |
| WE_HAVE_SHA3_224 |  SHA-3  224を有効にする（OpenSSL 1.0.2では利用不可）|
| WE_HAVE_SHA3_256 |  SHA-3  256を有効にする（OpenSSL 1.0.2では利用不可）|
| WE_HAVE_SHA3_384 |  SHA-3  384を有効にする（OpenSSL 1.0.2では利用不可）|
| WE_HAVE_SHA3_512 |  SHA-3  512を有効にする（OpenSSL 1.0.2では利用不可）|
| WE_HAVE_EVP_PKEY | EVP_PKEY APIを使用する機能を有効にする（RSA, DH等も含む） |
| WE_HAVE_CMAC |  CMAC を有効にする |
| WE_HAVE_HMAC |  HMAC を有効にする |
| WE_HAVE_DES3CBC |  DES3-CBC を有効にする |
|WE_HAVE_AESECB |  AES-ECB を有効にする |
| WE_HAVE_AESCBC |  AES-CBC を有効にする |
| WE_HAVE_AESCTR |  AES-countee modeを有効にする |
| WE_HAVE_AESGCM |  AES-GCM を有効にする |
| WE_HAVE_AESCCM | AES-CCM を有効にする |
| WE_HAVE_RANDOM |  wolfCrypt の疑似乱数生成実装を有効にする |
| WE_HAVE_RSA |  RSA 操作 (すなわち 署名, 検証, 鍵生成等)を有効にする |
| WE_HAVE_DH |  Diffie-Hellman 操作 (すなわち 鍵生成, 共有シークレット計算等)を有効にする |
| WE_HAVE_ECC |  楕円曲線暗号を有効にする |
| WE_HAVE_EC_KEY | EC_KEY_METHODのサポートを有効にする（OpenSSL 1.0.2では利用不可） |
| WE_HAVE_ECDSA |  ECDSA を有効にする |
| WE_HAVE_ECDH |  EC Diffie-Hellman operationsを有効にする |
| WE_HAVE_ECKEYGEN |  EC key generationを有効にする |
| WE_HAVE_EC_P192 |  EC Curve P192を有効にする |
| WE_HAVE_EC_P224 |  EC Curve P224を有効にする |
| WE_HAVE_EC_P256 |  EC Curve P256を有効にする |
| WE_HAVE_EC_P384 |  EC Curve P384を有効にする |
| WE_HAVE_EC_P512 |  EC Curve P512を有効にする |
| WE_HAVE_DIGEST | ダイジェストアルゴリズムをベンチマークとユニットテストのコードに含めてコンパイルする |
| WOLFENGINE_USER_SETTINGS | ユーザーの指定した定義をuser_settings.hファイルから読み込む |

