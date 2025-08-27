# 2	wolfSSLのポーティング

## データ型

**Q: どんな場合にこの章の内容が役立ちますか?**

A: プラットフォームに適したデータ型のサイズを設定することは常に重要です。

wolfSSL は、64ビット型を利用できることで速度面でメリットを得ています。プラットフォームの `sizeof(long)` と `sizeof(long long)` の結果と一致するように `SIZEOF_LONG` と `SIZEOF_LONG_LONG` を定義します。これは、`settings.h` または `user_settings.h` のカスタム定義に追加できます。たとえば、`MY_NEW_PLATFORM` のサンプル定義の下の `settings.h` では次のように示します。

```c
#ifdef MY_NEW_PLATFORM
    #define SIZEOF_LONG 4
    #define SIZEOF_LONG_LONG 8
...
#endif
```


wolfSSLとwolfCryptでは、`word32` と `word16` という2つの追加データ型を使用します。これらのデフォルトの型マッピングは次のとおりです。


```c
#ifndef WOLFSSL_TYPES
#ifndef byte
    typedef unsigned char byte
#endif
    typedef unsigned short word16;
    typedef unsigned int word32;
    typedef byte word24[3];
#endif
```


`word32` はコンパイラの32ビット型にマッピングされ、`word16` はコンパイラの16ビット型にマッピングされます。これらのデフォルトのマッピングがプラットフォームに適していない場合は、`settings.h` または `user_settings.h` で `WOLFSSL_TYPES` を定義し、word32およびword16に独自のカスタムtypedefを割り当てる必要があります。

wolfSSLのfastmathライブラリは、`fp_digit` および `fp_word` 型を使用します。デフォルトでは、これらはビルド構成に応じて `<wolfssl/wolfcrypt/tfm.h>` にマッピングされます。

`fp_word` は `fp_digit` の2倍のサイズである必要があります。デフォルトのケースがプラットフォームに当てはまらない場合は、`settings.h` または `user_settings.h` で `WOLFSSL_BIGINT_TYPES` を定義し、`fp_word` および `fp_digit` に独自のカスタムtypedefを割り当てる必要があります。

wolfSSLは、一部の操作において使用可能な場合は64ビット型を使用します。ビルド時には、`SIZEOF_LONG` および `SIZEOF_LONG_LONG` の設定に基づいて、`word64` の正しいデータ型を検出して設定しようとします。 真の64ビット型を持たない一部のプラットフォームでは、2つの32ビット型を合わせて使用するため、パフォーマンスが低下する可能性があります。コンパイル時に`NO_64BIT`を定義することで、64ビット型を使用しないこともできます。


## エンディアン

**Q: どんな場合にこの章の内容が役立ちますか?**

A: あなたのプラットフォームがビッグエンディアンの場合です。

お使いのプラットフォームはビッグエンディアンとリトルエンディアン、どちらでしょうか。wolfSSLはデフォルトでリトルエンディアンを使用しています。システムがビッグエンディアンの場合は、wolfSSL をビルドする際に `BIG_ENDIAN_ORDER` を定義します。例えば、`settings.h` で次のように設定できます。

```c
#ifdef MY_NEW_PLATFORM
    ...
    #define BIG_ENDIAN_ORDER
    ...
#endif
```


## writev

**Q: どんな場合にこの章の内容が役立ちますか?**

A: `<sys/uio.h>` を利用できない場合です。

デフォルトでは、wolfSSL APIは、`writev()` セマンティクスをシミュレートする `wolfSSL_writev()` をアプリケーションに提供します。 `<sys/uio.h>` ヘッダーが利用できないシステムでは、`NO_WRITEV` を定義してこの機能を除外します。


## ネットワークI/O

**Q: どんな場合にこの章の内容が役立ちますか?**

A: BSD スタイルのソケットAPIを利用できない、あるいはカスタムトランスポート層またはTCP/IPスタックを使用している、または静的バッファーのみを使用したい場合です。

wolfSSLはデフォルトでBSDスタイルのソケットインターフェイスを使用します。トランスポート層がBSDソケットインターフェイスを提供する場合、カスタムヘッダーが必要でない限り、wolfSSLはそのまま統合する必要があります。

wolfSSLはカスタムI/O抽象化レイヤーを提供し、ユーザーはwolfSSLのI/O機能をシステムに合わせて調整できます。詳細については、[5.1.2節](https://www.wolfssl.com/documentation/manuals/jp/wolfssl/chapter05.html#_2) をご参照ください。

具体的には `WOLFSSL_USER_IO` を定義し、wolfSSLデフォルトの `EmbedSend()` と `EmbedReceive()` をテンプレートとして使用し、独自のI/Oコールバック関数を記述します。これら2つの関数は `./src/io.c` にあります。

wolfSSLは、入力と出力に動的バッファを使用します。このバッファのデフォルトは0バイトです。バッファよりも大きいサイズの入力レコードを受信した場合、動的バッファが一時的にリクエストの処理に使用され、その後解放されます。

動的メモリを必要としない16kBの大きな静的バッファを使用する場合は、`LARGE_STATIC_BUFFERS` を定義することでこのオプションを使用できます。

動的バッファが使用され、ユーザーがバッファサイズよりも大きい `wolfSSL_write()` を要求した場合、最大 `MAX_RECORD_SIZE` までの動的ブロックを使用してデータを送信します。`RECORD_SIZE` で定義されている現在のバッファサイズの最大のチャンクでのみデータを送信したいユーザーは、`STATIC_CHUNKS_ONLY` を定義することでこれを実現できます。この定義を使用する場合、`RECORD_SIZE` はデフォルトで 128 バイトになります。


## ファイルシステム

**Q: どんな場合にこの章の内容が役立ちますか?**

A: ファイルシステムを利用できない、あるいは標準のファイルシステム機能を利用できない、または独自のファイルシステムを使用している場合です。

wolfSSLは、TLSセッションまたはコンテキストに鍵と証明書をロードするためにファイルシステムを使用します。wolfSSLでは、これらをメモリバッファからロードすることもできます。メモリバッファのみを使用する場合、ファイルシステムは必要ありません。

ライブラリをビルドするときに `NO_FILESYSTEM` を定義することで、wolfSSLによるファイルシステムの使用を無効にできます。つまり、証明書と鍵はファイルではなくメモリバッファからロードする必要があります。`settings.h` で、以下のように設定できます。

```c
#ifdef MY_NEW_PLATFORM
    ...
    #define NO_FILESYSTEM
    ...
#endif
```

テスト用の鍵と証明書バッファーは、`./wolfssl/certs_test.h` ヘッダーファイルにあります。これらは、`./certs` ディレクトリにある対応する証明書や鍵と一致します。

`certs_test.h` ヘッダーファイルは、必要に応じて `./gencertbuf.pl` スクリプトを使用して更新できます。`gencertbuf.pl` 内には、`fileList_1024` と `fileList_2048` の2つの配列があります。鍵サイズに応じて、追加の証明書またはキーをそれぞれの配列に追加できます。なお、DER形式である必要があります。上記の配列は、証明書・鍵ファイルの場所を目的のバッファー名にマップします。`gencertbuf.pl` を変更した後、wolfSSLルートディレクトリから実行すると、`./wolfssl/certs_test.h` の証明書・鍵のバッファーが更新されます。

```sh
./gencertbuf.pl
```

デフォルト以外のファイルシステムを使用する場合、ファイルシステム抽象化レイヤーは `./wolfssl/wolfcrypt/wc_port.h` にあります。ここには、EBSNET、FREESCALE_MQX、MICRIUM などのさまざまなプラットフォームのファイルシステムポートがあります。必要に応じて、プラットフォームのカスタム定義を追加できます。これにより、`XFILE`、`XFOPEN`、`XFSEEK` などを使用してファイルシステム関数を定義できます。たとえば、Micrium の µC/OS (MICRIUM) の `wc_port.h` のファイルシステムレイヤーは次のとおりです。

```c
#elif defined(MICRIUM)
#include <fs.h>
#define XFILE      FS_FILE*
#define XFOPEN     fs_fopen
#define XFSEEK     fs_fseek
#define XFTELL     fs_ftell
#define XREWIND    fs_rewind
#define XFREAD     fs_fread
#define XFCLOSE    fs_fclose
#define XSEEK_END  FS_SEEK_END
#define XBADFILE   NULL
```


## スレッド化

**Q: どんな場合にこの章の内容が役立ちますか?**

A: マルチスレッド環境でwolfSSLを使用したい、またはシングルスレッドモードでコンパイルしたい場合です。

wolfSSLをシングルスレッド環境でのみ使用する場合、wolfSSLをコンパイルするときに `SINGLE_THREADED` を定義することでwolfSSLミューテックスレイヤーを無効にできます。これにより、wolfSSLミューテックスレイヤーを移植する必要がなくなります。

wolfSSLをマルチスレッド環境で使用する場合、wolfSSLミューテックスレイヤーを新しい環境に移植する必要があります。ミューテックスレイヤーは `./wolfssl/wolfcrypt/wc_port.h` および `./wolfcrypt/src/wc_port.c` にあります。新しいシステムでは、`wc_port.h` に `wolfSSL_Mutex` を定義し、`wc_port.c` にミューテックス関数 (`wc_InitMutex`、`wc_FreeMutex`、`wc_LockMutex`、`wc_UnLockMutex`) を定義する必要があります。`wc_port.h` と `wc_port.c` を検索すると、既存のプラットフォームポートレイヤー (EBSNET、FREESCALE_MQX など) の例を参照できます。


## ランダムシード

**Q: どんな場合にこの章の内容が役立ちますか?**

A: `/dev/random` や `/dev/urandom` を利用できない、あるいはハードウェアRNGに統合する必要がある場合です。

デフォルトでは、wolfSSLは `/dev/urandom` または `/dev/random` を使用して RNG シードを生成します。wolfSSLをビルドするときに `NO_DEV_RANDOM` 定義を使用して、デフォルトの `GenerateSeed()` 関数を無効にすることができます。これが定義されている場合は、ターゲットプラットフォームに固有の `./wolfcrypt/src/random.c` にカスタム `GenerateSeed()` 関数を記述する必要があります。これにより、ハードウェアベースのランダムエントロピーソースが利用可能な場合は、wolfSSLのPRNGにシードを設定できます。

`GenerateSeed()` の記述方法の例については、`./wolfcrypt/src/random.c` にある wolfSSL の既存の `GenerateSeed()` 実装を参照してください。


## メモリー

**Q: どんな場合にこの章の内容が役立ちますか?**

A: 標準のメモリ関数が利用できない、あるいはオプションの数学ライブラリ間のメモリ使用量の違いに関心がある場合です。

wolfSSL本体はデフォルトで `malloc()` と `free()` の両方を使用します。旧来使用されてきた第1世代の整数演算ライブラリ(Normal Math)を使用する場合、wolfCryptは `realloc()` も使用します。

現在、整数演算ライブラリとして最初期に開発された第1世代のNormal Mathライブラリ、パブリックドメインのTFM(Tom's Fast Math)をベースに開発した第2世代のFast Mathライブラリ、SP(Single Precision)最適化を適用した第3世代のSP Mathライブラリが存在します。
このうち、第2世代以降のFast Math, SP Mathライブラリでは、暗号操作において動的メモリを使用しません。

メモリ使用量とスピードの観点から、私たちはSP Mathライブラリの使用を推奨しています。
wolfSSL 5.4.0以降ではデフォルトで採用しており、第1世代のNormal Mathライブラリは廃止予定です。
詳細は以下のページをご覧ください。

- [wolfSSLの新たな Multi-Precision 演算ライブラリ](https://wolfssl.jp/wolfblog/2021/02/03/multi-precision-math-library/)
- [wolfSSL Math Library Comparison Matrix](https://www.wolfssl.com/wolfssl-math-library-comparison-matrix/)
- [レガシー数学ライブラリを廃止します](https://wolfssl.jp/wolfblog/2023/03/13/deprecation-of-wolfssl-normal-math-library/)

wolfSSLのTLSレイヤーは依然としていくらかの動的メモリを使用するため、`malloc()` と `free()` は依然として必要です。

通常の `malloc()`、`free()`、および場合によっては `realloc()` 関数が使用できない場合は、`XMALLOC_USER` を定義し、ターゲット環境に固有の `./wolfssl/wolfcrypt/types.h` でカスタムメモリ関数フックを提供します。

`XMALLOC_USER` の使用の詳細については、wolfSSLマニュアルの[5.1.1.1節](https://www.wolfssl.com/documentation/manuals/jp/wolfssl/chapter05.html#_3) をご参照ください。


## 時間

**Q: どんな場合にこの章の内容が役立ちますか?**

A: 標準の時間関数 (`time()`、`gmtime()`) を利用できない、あるいはカスタムのクロックティック関数を指定する必要がある場合です。

デフォルトでは、wolfSSLは `./wolfcrypt/src/asn.c` で指定されているように、`time()`、`gmtime()`、および `ValidateDate()` を使用します。これらは、`XTIME`、`XGMTIME`、および `XVALIDATE_DATE` に抽象化されています。標準の時間関数と `time.h` を利用できない場合は、ユーザーは `USER_TIME` を定義できます。`USER_TIME` を定義した後、ユーザーは独自の `XTIME`、`XGMTIME`、および `XVALIDATE_DATE` 関数を定義できます。

wolfSSLは、クロックティック関数にデフォルトで `time(0)` を使用します。これは、`LowResTimer()` 関数内の `./src/internal.c` にあります。

`USER_TICKS` を定義すると、`time(0)` が必要ない場合にユーザーが独自のクロックティック関数を定義できるようになります。カスタム関数には秒単位の精度が必要ですが、エポックと相関させる必要はありません。参考として、`./src/internal.c` の `LowResTimer()` 関数を参照してください。


## C標準ライブラリ

**Q: どんな場合にこの章の内容が役立ちますか?**

A: C標準ライブラリを使用できない、あるいは独自のライブラリがある場合です。

wolfSSLは、開発者に高いレベルの移植性と柔軟性を提供するために、C標準ライブラリなしで構築できるようにしています。その場合、ユーザーはC標準の関数の代わりに使用したい関数をマップする必要があります。

第7章では、メモリ関数について説明しました。メモリ関数の抽象化に加えて、wolfSSLは文字列関数と数学関数も抽象化します。特定の関数は通常、`X<FUNC>` 形式の定義に抽象化されます。`<FUNC>` は抽象化される関数の名前です。

詳細については、wolfSSLマニュアルの[5.1節](https://www.wolfssl.com/documentation/manuals/jp/wolfssl/chapter05.html#_2)をお読みください。


## ロギング

**Q: どんな場合にこの章の内容が役立ちますか?**

A: デバッグメッセージを有効にしたいが、stderr を使用できない場合です。

デフォルトでは、wolfSSLはstderrを介してデバッグ出力を提供します。デバッグメッセージを有効にするには、wolfSSLを `DEBUG_WOLFSSL` を定義してコンパイルし、アプリケーションコードから `wolfSSL_Debugging_ON()` を呼び出す必要があります。同様に、アプリケーションから`wolfSSL_Debugging_OFF()` を呼び出すことで、wolfSSLデバッグメッセージをオフにすることもできます。

stderrを使用できない、あるいは別の出力ストリームや別の形式でデバッグメッセージを出力したい環境には、独自のコールバック関数を使用できるようにしています。

詳細については、wolfSSLマニュアルの[8.1節](https://www.wolfssl.com/documentation/manuals/jp/wolfssl/chapter08.html#_2)をお読みください。


## 公開鍵演算

**Q: どんな場合にこの章の内容が役立ちますか?**

A: wolfSSLで独自の公開鍵実装を使用したい場合です。

wolfSSLでは、SSL/TLSレイヤーが公開鍵操作を行う必要があるときに呼び出される、独自の公開鍵コールバックをユーザーが作成できます。 ユーザーはオプションで6つの機能を定義できます。

1. ECC 署名 コールバック
2. ECC 検証 コールバック
3. RSA 署名 コールバック
4. RSA 検証 コールバック
5. RSA 暗号化コールバック
6. RSA 復号 コールバック

詳細は、wolfSSLマニュアルの[6.4節](https://www.wolfssl.com/documentation/manuals/jp/wolfssl/chapter06.html#_5)を参照してください。


## アトミックレコードレイヤーの処理

**Q: どんな場合にこの章の内容が役立ちますか?**

A: レコードレイヤーの処理、特にMAC/暗号化および復号/検証操作を独自に実行したい場合です。

デフォルトでは、wolfSSLは暗号ライブラリwolfCryptを使用し、ユーザーに代わってレコードレイヤーの処理を行います。wolfSSLは、SSL/TLS接続中に MAC/暗号化および復号/検証機能をより細かく制御したいユーザーのために、アトミックレコード処理コールバックを提供します。

ユーザーは2つの関数を定義できます。

1. MAC/暗号化コールバック関数
2. 復号/検証コールバック関数

詳細は、wolfSSLマニュアルの[6.3節](https://www.wolfssl.com/documentation/manuals/jp/wolfssl/chapter06.html#_4)を参照してください。


## 機能

**Q: どんな場合にこの章の内容が役立ちますか?**

A: 特定の機能を無効にしたい場合です。

wolfSSLをビルドする際、マクロ定義を使用して特定の機能を無効化できます。
使用可能なマクロ定義については、wolfSSLマニュアルの[2章](https://www.wolfssl.com/documentation/manuals/jp/wolfssl/chapter02.html)をご覧ください。