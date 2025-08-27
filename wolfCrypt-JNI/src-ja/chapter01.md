# イントロダクション


JCE (Java Cryptography Extension) は、カスタムの暗号化サービス プロバイダーのインストールをサポートします。この仕組みにより、Java セキュリティ API によって使用される基本的な暗号化機能のサブセットを実装できます。

このマニュアルでは、wolfCrypt JCE プロバイダーの詳細と使用方法について説明します。 wolfCrypt JCE プロバイダー (wolfJCE) は、ネイティブの wolfCrypt 暗号化ライブラリーをラップして、Java Security API との互換性を確保します。 wolfCrypt JNI/JCEについては、[こちら](https://github.com/wolfSSL/wolfcrypt-jni)のGithubリポジトリを参照してください。

wolfcrypt-jni パッケージには、JCE プロバイダーとwolfCrypt JNI ラッパーの両方が含まれています。 JNI ラッパーは、必要に応じて単独で使用できます。
