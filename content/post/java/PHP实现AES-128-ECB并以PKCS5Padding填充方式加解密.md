---
title: PHP实现AES-128-ECB并以PKCS5Padding填充方式加解密
author: 要名俗气
type: post
date: 2024-06-29T03:36:33+00:00
url: /2024/php-aes-128-ecb-pkcs5padding
description: 工作中提供给外部调用的api需要进行请求的内容进行加密，加密使用的AES/ECB/PKCS5Padding这种加密方式，公司开发使用的是Java，但是对接企业有使用Java，Go或者php进行开发的，客户会要求提供一下demo，Java语言还好搞，本身就是Java开发的，直接提供就好了，但是php和Go由于公司没有使用者，在客户来询问的时候就没有demo可以...
image: https://images.iminling.com/app/hide.php?key=VDg3MjhNTkpJUzVSWUEyNGY0UHZCQkRvbXEwYXlpWVB5Q2ZuS3ZnTGUwTkxDVW1yTGd2L1N4cU5kVmZzQjIrcFQycG5UZXM9
categories:
  - PHP
tags:
  - AES
  - PHP
---
工作中提供给外部调用的api需要进行请求的内容进行加密，加密使用的AES/ECB/PKCS5Padding这种加密方式，公司开发使用的是Java，但是对接企业有使用Java，Go或者php进行开发的，客户会要求提供一下demo，Java语言还好搞，本身就是Java开发的，直接提供就好了，但是php和Go由于公司没有使用者，在客户来询问的时候就没有demo可以提供到，于是自己就研究了一下PHP的AES加密，并提供了一个demo。下边介绍一下具体的实现。

本文主要讲由Java转到PHP的实现，具体AES算法是怎么实现的不在本文的讨论范围，如果还不知道AES是什么自行搜索。

## Java实现

在这里有必要先说明一下Java的实现，因为只有知道Java是怎么实现的，才能在其他语言里去实现。先上代码吧：

```java
public class AesUtils {
    private static final String TRANSFORMATION = "AES/ECB/PKCS5Padding";
    private static final String ALGORITHM = "AES";
    public static String encrypt(String content, String key){
        try {
            Key secretKeySpec = secretKeySpec(key);
            Cipher cipher = Cipher.getInstance(TRANSFORMATION);// 创建密码器
            cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec);// 初始化
            byte[] result = cipher.doFinal(content.getBytes(StandardCharsets.UTF_8));
            return Base64.getEncoder().encodeToString(result);
        } catch (Exception e) {
        }
    }
    public static String decrypt(String content, String key) {
        try {
            Key keySpec = secretKeySpec(key);
            Cipher cipher = Cipher.getInstance(TRANSFORMATION);
            cipher.init(Cipher.DECRYPT_MODE, keySpec);
            byte[] result = cipher.doFinal(Base64.getDecoder().decode(content));
            return new String(result, StandardCharsets.UTF_8);
        } catch (Exception e) {
        }
    }
    private static Key secretKeySpec(String key) throws EncryptException {
        KeyGenerator generator = KeyGenerator.getInstance(ALGORITHM);
        SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG");
        secureRandom.setSeed(key.getBytes(StandardCharsets.UTF_8));
        generator.init(128, secureRandom);
        SecretKey secretKey = generator.generateKey();
        byte[] enCodeFormat = secretKey.getEncoded();
        return new SecretKeySpec(enCodeFormat, ALGORITHM);
    }
}
```

完整代码如上，整个代码也比较清晰，获取key然后对内容进行加解密。

**但是问题就出现在key获取上**，根据用户提供的key再使用RNG(Random Number Generator)算法：`SHA1PRNG`获取128位的key：`generator.init(128, secureRandom);`，由次可知完整的AES加密算法应该是AES-128-ECB并使用PKCS5Padding方式进行填充。

## PHP实现

通过阅读Java的代码发现完整算法是AES-128-ECB并使用PKCS5Padding方式进行填充，获取key的时候使用`SHA1PRNG`，但是PHP并没有这种方式，于是就去询问GPT，查看网上各种资料，最终找到了解决办法，PHP的完整代码如下：

```php
<?php
$plainText = isset($_GET['text']) ? $_GET['text'] : 'Hello, World!';
$key = isset($_GET['key']) ? $_GET['key'] : 'this_is_a_very_long_key_that_needs_to_be_shortened_or_hashed';

function encrypt($plainText, $key) {
    $key = substr(openssl_digest(openssl_digest($key, 'sha1', true), 'sha1', true), 0, 16);
    // openssl_encrypt 加密不同Mcrypt，对秘钥长度要求，超出16加密结果不变
    $data = openssl_encrypt($plainText, 'AES-128-ECB', $key, OPENSSL_RAW_DATA);
    //$data = strtolower(bin2hex($data));
    return base64_encode($data);
}

function decrypt($cipherText, $key) {
    $key = substr(openssl_digest(openssl_digest($key, 'sha1', true), 'sha1', true), 0, 16);
    $decrypted = openssl_decrypt(base64_decode($cipherText), 'AES-128-ECB', $key, OPENSSL_RAW_DATA);
    return $decrypted;
}

// 示例用法
$encrypted = encrypt($plainText, $key);
echo "加密结果: " . $encrypted . "\n";

$decrypted = decrypt($encrypted, $key);
echo "解密结果: " . $decrypted . "\n";
?>
```

如上，在php中是对Key进行两次sha1并对结果进行截取16位。

在PHP中，openssl\_encrypt和openssl\_decrypt函数会自动处理PKCS#5/PKCS#7填充（padding），不需要手动实现填充。参数中指明OPENSSL\_RAW\_DATA就可以了。

以上就是在PHP中实现Java中SHA1PRNG生成key并以AES-128-ECB加密且以PKCS#5方式进行填充的实现。记录一下。

