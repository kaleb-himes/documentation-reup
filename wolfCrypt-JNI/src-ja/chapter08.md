#  使用方法

使用方法については、上記の [第6章](chapter06#supported-algorithms-and-classes) で指定されているクラスの Oracle/OpenJDK Javadoc に従ってください。 `java.security` ファイルで同じアルゴリズムを提供する他のプロバイダーよりも優先順位が低く設定されている場合は、 `wolfJCE`プロバイダーを明示的に要求する必要があることに注意してください。 たとえば、SHA-1のMessageDigestクラスでwolfJCEプロバイダーを使用するには、次のようにMessageDigestオブジェクトを作成します：

```
MessageDigest md = MessageDigest.getInstance(“SHA-1”, “wolfJCE”);
```

ご質問やご意見がございましたら、support@wolfssl.com までメールでお寄せください。
