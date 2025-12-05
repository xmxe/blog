---
title: Web项目中的安全设计
tags: 代码实战
img: https://picx.zhimg.com/v2-bb7faf486653cbbdf0cd7e10e784e2ee_1440w.jpg

---

## uid的生成和传输方案

**uid的使用场景**

uid作为重放攻击解决方案中计算签名的盐值、也是完整性校验中计算签名的盐值，同时也是用作对称加密的密钥（敏感数据加密传输），因此uid的生成和访问机制尤为重要。
本方案综合考虑了uid的传输安全、防止中间人劫持，可保证每个用户每个会话周期内的uid均不同，且无法伪造。
1. 用户首次访问系统后，请求服务端公钥（非对称加密算法生成的公私钥）
2. 服务端生成与用户会话相关的公私钥对（会话周期内有效），将公钥publickey_server发送给客户端（为防止攻击者分析并收集加固信息，服务端公钥随其他响应信息一起传回客户端，不单独发送）
3. 客户端接收响应后将publickey_server存储到localstorage中（加固相关信息存储到localstorage中而不是sessionstorage中，是为了防止客户端浏览器跨页面操作时获取不到相关参数）
4. 客户端生成公私钥对，用服务端公钥对客户端公钥进行加密`encryptpublickey_client=SM2.encrypt(publickey_client，publickey_server)`，并将加密后的客户端公钥发送到服务端
5. 服务端接收请求后使用服务端私钥对其解密，获取明文客户端公钥`publickey_client=SM2.decrypt(encryptpublickey_client，privatekey_server)`
6. 服务端生成与用户会话相关的随机数uid（会话周期有效），并使用客户端公钥对uid进行加密后发送到客户端，`encryptuid=SM2.encrypt(uid，publickey_client)`
7. 客户端接收响应数据后，用客户端私钥对其进行解密得到明文uid，`uid=SM2.decrypt(encryptuid，privatekey_client)`，并将uid本地localstorage中保存待后续使用。

注意：若系统使用SSL机制，则服务端生成uid后可直接返回给客户端，无需进行手动加解密过程。

详见下图所示
![](images/uid的生成和传输方案.png)

简明扼要前后端登录国密加解密:
1. 服务端根据用户会话生成密钥对
2. 客户端获取服务端公钥，保存至localStorage
3. 客户端生成密钥对，通过服务端公钥加密客户端公钥然后发送到服务端。
4. 服务端通过服务端私钥解密获取客户端公钥
5. 服务端生成uid(数据完整性校验、防重返时使用)，通过客户端公钥加密uid返回给客户端
6. 客户端通过客户端私钥解密uid，进行放重放、完整性校验使用。

**代码实现**

```java
// 适配sm-crypto的SM2Util.java
import org.bouncycastle.crypto.engines.SM2Engine;
import org.bouncycastle.crypto.params.ECPrivateKeyParameters;
import org.bouncycastle.crypto.params.ECPublicKeyParameters;
import org.bouncycastle.crypto.params.ParametersWithRandom;
import org.bouncycastle.jce.provider.BouncyCastleProvider;
import org.bouncycastle.crypto.util.PrivateKeyFactory;
import org.bouncycastle.crypto.util.PublicKeyFactory;
import org.bouncycastle.util.encoders.Hex;

import java.nio.charset.StandardCharsets;
import java.security.*;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.Arrays;
import java.util.Base64;

public class SM2Util {

    static {
        Security.addProvider(new BouncyCastleProvider());
    }

    /**
     * 获取密钥对
     */
    public static KeyPair generateSM2KeyPair() throws Exception {
        KeyPairGenerator keyGen = KeyPairGenerator.getInstance("EC", "BC");
        keyGen.initialize(new java.security.spec.ECGenParameterSpec("sm2p256v1"));
        return keyGen.generateKeyPair();
    }
    /**
     * 获取未压缩公钥Hex（供前端使用）
     * 当需要将公钥发送给前端使用时调用此方法
     */
    public static String getUncompressedPubKeyHex(PublicKey pubKey) {
        java.security.interfaces.ECPublicKey ecPub = (java.security.interfaces.ECPublicKey) pubKey;
        java.security.spec.ECPoint w = ecPub.getW();
        return "04"
                + paddedHex(w.getAffineX().toByteArray())
                + paddedHex(w.getAffineY().toByteArray());
    }

    private static String paddedHex(byte[] bytes) {
        String s = new java.math.BigInteger(1, bytes).toString(16);
        while (s.length() < 64) s = "0" + s;
        return s;
    }

    /**
     * 获取私钥的PKCS#8 DER编码（Base64）—— 可直接硬编码复用
     */
    public static String getPrivateKeyPkcs8Base64(PrivateKey privateKey) {
        return Base64.getEncoder().encodeToString(privateKey.getEncoded());
    }

    /**
     * 从前端公钥Hex字符串加载公钥
     * 获取前端公钥加密数据调用此方法
     */
    public static PublicKey loadPublicKeyFromHex(String publicKeyHex) throws Exception {
        byte[] publicKeyBytes = Hex.decode(publicKeyHex);
        
        // 如果是04开头的未压缩格式，直接使用
        if (publicKeyBytes[0] == 0x04) {
            // 使用X509编码格式
            String x509Base64 = getX509PublicKeyBase64(publicKeyBytes);
            return loadPublicKeyFromBase64(x509Base64);
        }
        
        throw new IllegalArgumentException("不支持的公钥格式");
    }

    /**
     * 将未压缩公钥转换为X509格式
     */
    private static String getX509PublicKeyBase64(byte[] uncompressedPublicKey) throws Exception {
        // 这里需要将未压缩公钥转换为X509格式
        // 简化处理：直接使用BouncyCastle的KeyFactory
        KeyFactory keyFactory = KeyFactory.getInstance("EC", "BC");
        java.security.spec.ECPoint point = new java.security.spec.ECPoint(
            new java.math.BigInteger(1, Arrays.copyOfRange(uncompressedPublicKey, 1, 33)),
            new java.math.BigInteger(1, Arrays.copyOfRange(uncompressedPublicKey, 33, 65))
        );
        java.security.spec.ECParameterSpec ecSpec = new java.security.spec.ECParameterSpec(
            new java.security.spec.EllipticCurve(
                new java.security.spec.ECFieldFp(new java.math.BigInteger("FFFFFFFEFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF00000000FFFFFFFFFFFFFFFF", 16)),
                new java.math.BigInteger("FFFFFFFEFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF00000000FFFFFFFFFFFFFFFC", 16),
                new java.math.BigInteger("28E9FA9E9D9F5E344D5A9E4BCF6509A7F39789F515AB8F92DDBCBD414D940E93", 16)
            ),
            new java.security.spec.ECPoint(
                new java.math.BigInteger("32C4AE2C1F1981195F9904466A39C9948FE30BBFF2660BE1715A4589334C74C7", 16),
                new java.math.BigInteger("BC3736A2F4F6779C59BDCEE36B692153D0A9877CC62A474002DF32E52139F0A0", 16)
            ),
            new java.math.BigInteger("FFFFFFFEFFFFFFFFFFFFFFFFFFFFFFFF7203DF6B21C6052B53BBF40939D54123", 16),
            1
        );
        java.security.spec.ECPublicKeySpec pubKeySpec = new java.security.spec.ECPublicKeySpec(point, ecSpec);
        return Base64.getEncoder().encodeToString(keyFactory.generatePublic(pubKeySpec).getEncoded());
    }

    /**
     * 从Base64加载公钥
     */
    public static PublicKey loadPublicKeyFromBase64(String x509Base64) throws Exception {
        byte[] x509 = Base64.getDecoder().decode(x509Base64);
        X509EncodedKeySpec spec = new X509EncodedKeySpec(x509);
        KeyFactory keyFactory = KeyFactory.getInstance("EC", "BC");
        return keyFactory.generatePublic(spec);
    }

    /**
     * 使用前端公钥加密（返回C1C3C2格式Hex字符串，与前端sm-crypto兼容）
     */
    public static String encryptForFrontend(String plainText, PublicKey publicKey) throws Exception {
        // 创建公钥参数
        ECPublicKeyParameters pubKeyParam = (ECPublicKeyParameters)
                PublicKeyFactory.createKey(publicKey.getEncoded());

        // 加密必须使用ParametersWithRandom
        SecureRandom random = new SecureRandom();
        ParametersWithRandom params = new ParametersWithRandom(pubKeyParam,random);

        // 使用C1C2C3模式加密
        SM2Engine engine = new SM2Engine(SM2Engine.Mode.C1C2C3);
        engine.init(true, params);

        byte[] plainBytes = plainText.getBytes(StandardCharsets.UTF_8);
        byte[] cipherBytes = engine.processBlock(plainBytes, 0, plainBytes.length);

        // 转换为C1C3C2格式（前端期望的格式）
        byte[] c1c3c2 = convertC1C2C3ToC1C3C2(cipherBytes);
        return Hex.toHexString(c1c3c2);
    }

    /**
     * 使用服务端私钥解密前端发来的使用服务端公钥加密的数据
     */
    public static String decryptFromFrontend(String encryptedHex, PrivateKey privateKey) throws Exception {
        byte[] cipherBytes = Hex.decode(encryptedHex);
        // 根据第一个字节判断格式
        int firstByte = cipherBytes[0] & 0xFF;

        if (firstByte == 0x04) {
            System.out.println("检测到未压缩格式 (04开头) - 使用C1C3C2转C1C2C3");
            return decryptC1C3C2(cipherBytes, privateKey);
        } else if (firstByte == 0x02 || firstByte == 0x03) {
            System.out.println("检测到压缩格式 (" + String.format("%02x", firstByte) + "开头) - 直接解密");
            return decryptCompressed(cipherBytes, privateKey);
        } else {
            System.out.println("检测到未知格式，尝试自动处理");
            return tryAllFormats(cipherBytes, privateKey);
        }
    }

    /**
     * C1C3C2 → C1C2C3 转换
     */
    public static byte[] convertC1C3C2ToC1C2C3(byte[] c1c3c2) {
        int c1Len = 65; // 未压缩点：0x04 + 32x + 32y
        int c3Len = 32; // SM3 digest (32字节)
        int totalLen = c1c3c2.length;
        int c2Len = totalLen - c1Len - c3Len;

        // 验证长度
        if (totalLen < c1Len + c3Len) {
            throw new IllegalArgumentException("密文长度无效: " + totalLen + "，至少需要 " + (c1Len + c3Len) + " 字节");
        }

        // System.out.println("C1C3C2转换详情:");
        // System.out.println("  - 总长度: " + totalLen);
        // System.out.println("  - C1长度: " + c1Len);
        // System.out.println("  - C3长度: " + c3Len);
        // System.out.println("  - C2长度: " + c2Len);

        // 分割各部分
        byte[] c1 = Arrays.copyOfRange(c1c3c2, 0, c1Len);           // 公钥点
        byte[] c3 = Arrays.copyOfRange(c1c3c2, c1Len, c1Len + c3Len); // SM3哈希
        byte[] c2 = Arrays.copyOfRange(c1c3c2, c1Len + c3Len, totalLen); // 实际加密数据

        // 重组为 C1C2C3
        byte[] c1c2c3 = new byte[totalLen];
        System.arraycopy(c1, 0, c1c2c3, 0, c1Len);           // C1
        System.arraycopy(c2, 0, c1c2c3, c1Len, c2Len);       // C2
        System.arraycopy(c3, 0, c1c2c3, c1Len + c2Len, c3Len); // C3

        System.out.println("C1C3C2 → C1C2C3 转换完成");
        return c1c2c3;
    }

    /**
     * 反向转换：C1C2C3 → C1C3C2
     */
    public static byte[] convertC1C2C3ToC1C3C2(byte[] c1c2c3) {
        int c1Len = 65; // 未压缩点
        int c3Len = 32; // SM3 digest
        int totalLen = c1c2c3.length;
        int c2Len = totalLen - c1Len - c3Len;

        if (totalLen < c1Len + c3Len) {
            throw new IllegalArgumentException("密文长度无效: " + totalLen);
        }

        byte[] c1 = Arrays.copyOfRange(c1c2c3, 0, c1Len);
        byte[] c2 = Arrays.copyOfRange(c1c2c3, c1Len, c1Len + c2Len);
        byte[] c3 = Arrays.copyOfRange(c1c2c3, c1Len + c2Len, totalLen);

        byte[] c1c3c2 = new byte[totalLen];
        System.arraycopy(c1, 0, c1c3c2, 0, c1Len);
        System.arraycopy(c3, 0, c1c3c2, c1Len, c3Len);
        System.arraycopy(c2, 0, c1c3c2, c1Len + c3Len, c2Len);

        return c1c3c2;
    }

    /**
     * 未压缩格式解密（04开头）
     */
    private static String decryptC1C3C2(byte[] cipherBytes, PrivateKey privateKey) throws Exception {
        // C1C3C2 → C1C2C3
        byte[] c1c2c3 = convertC1C3C2ToC1C2C3(cipherBytes);

        ECPrivateKeyParameters privKeyParam = (ECPrivateKeyParameters)
                PrivateKeyFactory.createKey(privateKey.getEncoded());

        // 使用C1C2C3模式解密
        SM2Engine engine = new SM2Engine(SM2Engine.Mode.C1C2C3);
        engine.init(false, privKeyParam);

        byte[] plainBytes = engine.processBlock(c1c2c3, 0, c1c2c3.length);
        return new String(plainBytes, StandardCharsets.UTF_8);
    }

    /**
     * 压缩格式解密（02或03开头）
     */
    private static String decryptCompressed(byte[] cipherBytes, PrivateKey privateKey) throws Exception {
        ECPrivateKeyParameters privKeyParam = (ECPrivateKeyParameters) PrivateKeyFactory.createKey(privateKey.getEncoded());

        // 压缩格式通常直接使用C1C2C3模式
        SM2Engine engine = new SM2Engine(SM2Engine.Mode.C1C2C3);
        engine.init(false, privKeyParam);

        byte[] plainBytes = engine.processBlock(cipherBytes, 0, cipherBytes.length);
        return new String(plainBytes, StandardCharsets.UTF_8);
    }

    // 尝试所有可能的格式
    private static String tryAllFormats(byte[] cipherBytes, PrivateKey privateKey) {
        // 原有的尝试逻辑
        String[] methods = {
                "原始数据(C1C2C3)",
                "原始数据(C1C3C2)",
                "添加04前缀(C1C2C3)",
                "添加04前缀并转C1C2C3"
        };

        byte[][] testData = {
                cipherBytes, // 原始
                cipherBytes, // 原始（不同模式）
                addPrefix(cipherBytes, (byte) 0x04), // 加04
                convertC1C3C2ToC1C2C3(addPrefix(cipherBytes, (byte) 0x04)) // 加04并转换
        };

        SM2Engine.Mode[] modes = {
                SM2Engine.Mode.C1C2C3,
                SM2Engine.Mode.C1C3C2,
                SM2Engine.Mode.C1C2C3,
                SM2Engine.Mode.C1C2C3
        };

        for (int i = 0; i < methods.length; i++) {
            try {
                ECPrivateKeyParameters privKeyParam = (ECPrivateKeyParameters)
                        PrivateKeyFactory.createKey(privateKey.getEncoded());

                SM2Engine engine = new SM2Engine(modes[i]);
                engine.init(false, privKeyParam);

                byte[] plainBytes = engine.processBlock(testData[i], 0, testData[i].length);
                String result = new String(plainBytes, StandardCharsets.UTF_8);

                System.out.println("✓ " + methods[i] + " 解密成功: " + result);
                return result;
            } catch (Exception e) {
                e.printStackTrace();
                System.out.println("✗ " + methods[i] + " 失败: " + e.getMessage());
            }
        }
        throw new RuntimeException("所有解密尝试都失败");
    }

    private static byte[] addPrefix(byte[] data, byte prefix) {
        byte[] result = new byte[data.length + 1];
        result[0] = prefix;
        System.arraycopy(data, 0, result, 1, data.length);
        return result;
    }

    /**
     * 从Base64加载私钥
     */
    public static PrivateKey loadPrivateKeyFromBase64(String pkcs8Base64) throws Exception {
        byte[] pkcs8 = Base64.getDecoder().decode(pkcs8Base64);
        PKCS8EncodedKeySpec spec = new PKCS8EncodedKeySpec(pkcs8);
        KeyFactory keyFactory = KeyFactory.getInstance("EC", "BC");
        return keyFactory.generatePrivate(spec);
    }

    /**
     * 测试方法
     */
    public static void main(String[] args) throws Exception {
        // 服务端生成密钥对
        KeyPair keyPair = generateSM2KeyPair();
        // 发送前端需要的服务端密钥
        String frontPublicKey = getUncompressedPubKeyHex(keyPair.getPublic());
        // 后端私钥转换，易于保存
        String privateKeyBase64 = getPrivateKeyPkcs8Base64(keyPair.getPrivate());
        System.out.println("前端公钥: " + frontPublicKey);
        System.out.println("后端私钥: " + privateKeyBase64);

        // 测试数据
        String encryptedHex = "043aa90069a8ff7a4b4ab45a3a96e2bd6a3d6c0fd12f03d9332cff0c070c6a3baac1914697438318055fc1e72bc558720129d835fd6800c01420fdb83676f2f63cbaa165612f346e859c23b6b1b463bdc3be69406326570ab2df496890bfd55cd5a072cc26eea04bd83f2609a6";
        String privateKeyBase642 = "MIGTAgEAMBMGByqGSM49AgEGCCqBHM9VAYItBHkwdwIBAQQgDSd+qEfDfdciXKEODn9fN5c7CfO6VsHlL765JEnePnmgCgYIKoEcz1UBgi2hRANCAAQFxSaO5eXZtgUKiRq383CDUvanY+7kmvOD/tMTajbOO7nmEqYOij2GWo9uN3mNgus7uy18isn39jQU/SeSBjbr";

        try {
            PrivateKey privateKey = loadPrivateKeyFromBase64(privateKeyBase642);
            String decryptedText = decryptFromFrontend(encryptedHex, privateKey);
            System.out.println("解密成功! 结果: " + decryptedText);
        } catch (Exception e) {
            e.printStackTrace();
        }

        // 测试加密解密
        String plainText = "Hello, SM2加密测试!";
        System.out.println("原始文本: " + plainText);

        // 使用前端公钥加密
        PublicKey publicKey = loadPublicKeyFromHex(frontPublicKey);
        String encrypted = encryptForFrontend(plainText, publicKey);
        System.out.println("加密结果: " + encrypted);

        // 使用后端私钥解密
        PrivateKey privateKey = loadPrivateKeyFromBase64(privateKeyBase64);
        String decrypted = decryptFromFrontend(encrypted, privateKey);
        System.out.println("解密结果: " + decrypted);

        System.out.println("加解密验证: " + plainText.equals(decrypted));
    }

    // <script src="https://cdn.jsdelivr.net/npm/sm-crypto@0.3.2/dist/sm2.min.js"></script>
    // let currentPublicKey = "0405c5268ee5e5d9b6050a891ab7f3708352f6a763eee49af383fed3136a36ce3bb9e612a60e8a3d865a8f6e37798d82eb3bbb2d7c8ac9f7f63414fd27920636eb";
    // const encryptData = sm2.doEncrypt(plainText, currentPublicKey);
}
```

```js
// smutils.js
export function SM4Util() {
    // 和后端key一致
    this.secretKey="jjr9bez13evljita";
    // 当时用CBC模式的时候
    this.iv = "ZkR_SiNoSOFT=568";
    this.hexString = false;

    // ECB模式加密
    this.encryptData_ECB=function(plainText){
        try{
            const sm4 = new SM4();
            const ctx = new SM4_Context();
            ctx.isPadding = true;
            ctx.mode = sm4.SM4_ENCRYPT;
            const keyBytes = this.stringToByte(this.secretKey);
            sm4.sm4_setkey_enc(ctx, keyBytes);
            const encrypted = sm4.sm4_crypt_ecb(ctx, this.stringToByte(plainText));
            const cipherText = base64js.fromByteArray(encrypted);
            if (cipherText != null && cipherText.trim().length > 0)
            {
                cipherText.replace(/(\s*|\t|\r|\n)/g, "");
            }
            return cipherText;
        }catch (e){
            return null;
        }
    }
    //解密_ECB
    this.decryptData_ECB = function(cipherText) {
        try {
            let sm4 = new SM4();
            let ctx = new SM4_Context();
            ctx.isPadding = true;
            ctx.mode = sm4.SM4_ENCRYPT;
            let keyBytes = this.stringToByte(this.secretKey);
            sm4.sm4_setkey_dec(ctx, keyBytes);
            let decrypted = sm4.sm4_crypt_ecb(ctx, base64js.toByteArray(cipherText));
            return this.byteToString(decrypted);
        } catch (e) {
            return null;
        }
    }

    // CBC模式加密
    this.encryptData_CBC=function(plainText){
        try{
            const sm4 = new SM4();
            const ctx = new SM4_Context();
            ctx.isPadding = true;
            ctx.mode = sm4.SM4_ENCRYPT;

            const keyBytes = this.stringToByte(this.secretKey);
            const ivBytes = this.stringToByte(this.iv);

            sm4.sm4_setkey_enc(ctx, keyBytes);
            const encrypted = sm4.sm4_crypt_cbc(ctx, ivBytes, this.stringToByte(plainText));
            const cipherText = base64js.fromByteArray(encrypted);
            if (cipherText != null && cipherText.trim().length > 0)
            {
                cipherText.replace(/(\s*|\t|\r|\n)/g, "");
            }
            return cipherText;
        }
        catch ( e)
        {
            return null;
        }
    }
    //解密_CBC
    this.decryptData_CBC = function(cipherText) {
        try {
            let sm4 = new SM4();
            let ctx = new SM4_Context();
            ctx.isPadding = true;
            ctx.mode = sm4.SM4_ENCRYPT;
            let keyBytes = this.stringToByte(this.secretKey);
            let ivBytes = this.stringToByte(this.iv);
            sm4.sm4_setkey_dec(ctx, keyBytes);
            let decrypted = sm4.sm4_crypt_cbc(ctx, ivBytes, base64js.toByteArray(cipherText));
            return this.byteToString(decrypted);
        } catch (e) {
            return null;
        }
    }

    this.stringToByte=function(str) {
        const bytes = [];
        let len, c;
        len = str.length;
        for(let i = 0; i < len; i++) {
            c = str.charCodeAt(i);
            if(c >= 0x010000 && c <= 0x10FFFF) {
                bytes.push(((c >> 18) & 0x07) | 0xF0);
                bytes.push(((c >> 12) & 0x3F) | 0x80);
                bytes.push(((c >> 6) & 0x3F) | 0x80);
                bytes.push((c & 0x3F) | 0x80);
            } else if(c >= 0x000800 && c <= 0x00FFFF) {
                bytes.push(((c >> 12) & 0x0F) | 0xE0);
                bytes.push(((c >> 6) & 0x3F) | 0x80);
                bytes.push((c & 0x3F) | 0x80);
            } else if(c >= 0x000080 && c <= 0x0007FF) {
                bytes.push(((c >> 6) & 0x1F) | 0xC0);
                bytes.push((c & 0x3F) | 0x80);
            } else {
                bytes.push(c & 0xFF);
            }
        }
        return bytes;
    }


    this.byteToString=function(arr) {
        if(typeof arr === 'string') {
            return arr;
        }
        let str = '',
            _arr = arr;
        for(let i = 0; i < _arr.length; i++) {
            const one = _arr[i].toString(2),
                v = one.match(/^1+?(?=0)/);
            if(v && one.length === 8) {
                const bytesLength = v[0].length;
                let store = _arr[i].toString(2).slice(7 - bytesLength);
                for(let st = 1; st < bytesLength; st++) {
                    store += _arr[st + i].toString(2).slice(2);
                }
                str += String.fromCharCode(parseInt(store, 2));
                i += bytesLength - 1;
            } else {
                str += String.fromCharCode(_arr[i]);
            }
        }
        return str;
    }
}
```

## 数据完整性校验

**描述**

数据传输完整性是指数据在传输过程中，不被未授权的篡改或在篡改后能够被迅速发现。

**防护建议**

1. 传输过程中数据做整体加密处理（该方案存在一定风险，若攻击者拥有系统访问权限，可在数据未解密的情况下直接篡改加密数据，进行提权操作，固未从根本上解决数据完整性问题，建议采用方案2）
2. 数据签名校验：建立客户端和服务端全流程校验和处理机制，有效解决数据在请求和响应传输过程中被篡改的问题，提高数据双向传输完整性，用以保障信息系统安全。为保证数据的完整性，约定uid为密钥（uid的生成和传输机制详见[uid的生成和传输方案](#uid的生成和传输方案)）用于签名计算。

系统应采用校验码技术或密码技术保证鉴别信息和重要业务数据等敏感信息在传输过程中的完整性。
    ①等保三级要求：系统鉴别信息和重要业务数据等敏感信息在传输过程中应采用国家密码主管部门要求的加密算法（SM3等）对其进行数据完整性校验
    ②等保二级要求：系统中的鉴别信息和重要业务数据等敏感信息在传输过程中应采用校验码技术或密码技术（MD5、SHA等）对其进行数据完整性校验

**请求传输完整性校验实现方案**

（1）客户端从localstorage中读取加固参数uid和时间差△timestamp（服务器时间与客户端时间的时间差），并计算请求发送时的服务端当前时间戳为:客户端当前时间+△timestamp。
（2）客户端使用散列算法对请求数据data和关键加固参数uid，timestamp进行签名计算，以国密算法SM3举例：`clientSM3=SM3(sort(data+salt+timestamp))`,sort的含义是将参数值按照字母字典排序，然后从小到大拼接成一个字符串。
（3）将签名clientSM3和时间戳以参数或自定义header的形式随请求发送给服务器。
（4）服务端收到请求，判断请求中是否存在clientSM3和时间戳参数，若不存在则请求无效，并向客户端返回请求无效；若存在，则读取并继续执行。
（5）服务端验证时间戳参数，服务端当前时间timestamp_now-timestamp是否大于60s或<-5s（默认一次HTTP请求从发出到到达服务器的时间是不会超过60s的，且不会小于0，考虑到计算机精度和多网关业务场景，故判断条件为>60s或<-5s）。若大于60s或<-5s，则请求无效；反之为有效请求，。
（6）服务端验证clientSM3参数，服务端根据请求的会话标识读取该用户的uid参数，并记录请求中的timestamp、请求数据data，调用签名生成算法，得到`serverSM3=SM3(sort(data+uid+timestamp))`，验证serverSM3是否等于clientSM3，若不一致，则请求无效，并向客户端返回数据请求被篡改；若一致说明数据未篡改，进行下一步业务处理。

![](images/请求传输完整性.png)

**响应传输完整性校验实现方案**

（1）服务端返回响应前根据会话标识读取该用户的uid参数，并获取当前服务端时间timestamp
（2）服务端使用散列算法对响应数据data和salt值进行签名计算，以国密SM3举例：`serverSM3=SM3(sort(data+uid+timestamp))`,sort的含义是将参数值按照字母字典排序，然后从小到大拼接成一个字符串。
（3）将签名serverSM3和timestamp以参数或自定义header的形式随响应发送给客户端。
（4）客户端收到响应后判断响应中是否存在serverSM3和timestamp参数，若不存在则响应无效，客户端不再解析该响应；若存在，则读取并继续执行。
（5）验证timestamp参数，客户端当前时间timestamp_now-timestamp是否大于60s或<-5s（默认一次HTTP响应从发出到到达客户端的时间是不会超过60s的，且不会小于0，考虑到计算机精度和多网关业务场景，故判断条件为>60s或<-5s）。若大于60s或<-5s，则响应无效；反之则继续执行。
（6）验证serverSM3参数，客户端读取该用户的盐值uid，并记录响应中的timestamp、响应数据data，调用签名生成算法，得到clientSM3=SM3(sort(data+salt+timestamp))，验证clientSM3是否等于serverSM3，若不一致，则响应无效，说明响应数据被篡改；若一致说明数据未篡改，响应正常，则客户端对响应解析并进行下一步处理。

![](images/响应传输完整性.png)

**简明扼要**

1、传输过程中数据加密处理
2、数据签名校验：为保证数据的完整性，约定uid为密钥（仅一次会话周期有效，且仅客户端和服务端知道，在传输中不可见）用于签名计算，客户端发送请求时，将请求内容与uid结合计算出签名；服务端接收到请求后，也按相同算法计算出签名。如果相等，则请求来自可信任客户端，且请求是完整的。
    ①客户端使用哈希算法对请求中数据和salt值进行签名计算（本方案中uid取为salt值）sign_client=md5(sort(data+uid)),sort的含义是将参数值按照字母字典排序，然后从小到大拼接成一个字符串。
    ②将签名sign_client以参数的形式发送给服务器
    ③服务器收到数据之后，根据用户会话读取uid，对数据使用相同的哈希算法进行签名计算sign_server=md5(sort(data+uid))
    ④验证sign_client和sign_server是否一致，若一致则说明数据没有经过篡改，执行响应操作，若不一致，则签名校验异常，数据经过篡改视为无效请求。

![](images/完整性校验流程.png)

**代码实现**

```java
// Spring拦截器实现
import java.net.URLDecoder;
import java.util.Arrays;
import java.util.Enumeration;
import java.util.Map;
import java.util.Objects;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.bouncycastle.crypto.digests.SM3Digest;
import org.bouncycastle.util.encoders.Hex;
import org.springframework.util.Assert;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

import cn.hutool.core.net.URLEncodeUtil;

/**
 * 数据完整性校验
 */
public class IntegralityCheckInterceptor extends HandlerInterceptorAdapter {// implements HandlerInterceptor

	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
		
		if (handler instanceof HandlerMethod) {
//            HandlerMethod handlerMethod = (HandlerMethod) handler;
		       String [] white ={"downLoadFromFTP"};
		        String path =request.getServletPath();
		        for (int i = 0; i < white.length; i++) {
		            if(path.indexOf(white[i])!=-1){
		            	return true;
		            }
		        }
        	//客户端时间+△timestamp，即发送请求时的服务端的当前时间
			try {
                Long timestamp = Long.valueOf(request.getHeader("timestamp"));
                // 客户端使用散列算法对请求数据data和关键加固参数uid，timestamp进行签名计算
                String clientSM3 = request.getHeader("clientSM3");
                if(Objects.nonNull(timestamp) && Objects.nonNull(clientSM3)){
                    // 失效时间
                    int seconds = 60;
                    // 服务器当前时间
                    long currentTime = System.currentTimeMillis();
                    long timeDiff = currentTime - timestamp;
                    // 默认一次HTTP请求从发出到到达服务器的时间是不会超过60s的，且不会小于0，考虑到计算机精度和多网关业务场景，故判断条件为>60s或<-5s
                    if(timeDiff > seconds * 1000 || timeDiff < -5 * 1000){
                        return false;
                    }

                    // 验证clientSM3参数
                    // 服务端根据请求的会话标识读取该用户的uid参数
                    String uid = (String) request.getSession().getAttribute("uid");
                    Assert.notNull(uid);

                    // 请求数据data
                    String data = "";
                    // 注意：put请求等json格式的接口无法获取参数
                    Map<String,String[]> paramMap = request.getParameterMap();
                    if(paramMap!=null && !paramMap.isEmpty()){
                        StringBuilder sb = new StringBuilder();
                        for(Map.Entry<String, String[]> entry : paramMap.entrySet()){
                            String key = entry.getKey();
                            String[] value = entry.getValue();
                            for(String paramValue : value){
                                sb.append(key);
                                sb.append("=");
                                String param = paramValue.replaceAll(" ","");
                                String deparam = null;
                                try {
                                    deparam = URLDecoder.decode(param,"UTF-8");
                                }catch (Exception e){
                                    e.printStackTrace();
                                    // 注意此块内容要符合业务逻辑，当有转义的时候需要怎么处理
                                    String fixed = param.replaceAll("%(?![0-9A-Fa-f]{2})","%25");
                                    deparam = URLDecoder.decode(fixed,"UTF-8");
                                }
                                // String deparam = URLDecoder.decode(param,"UTF-8");
                                if(deparam == null){
                                    return false;
                                }
                                sb.append(deparam);
                                sb.append("&");
                            }          			
                        }
                        // 去掉空格
                        String strParam = sb.substring(0, sb.length()-1).toString().replaceAll("\\s","");
                        data = URLEncodeUtil.encodeAll(strParam);
                    }	
                    // 调用签名算法serverSM3=SM3(sort(data+uid+timestamp))
                    char[] c = (data+uid+timestamp).toCharArray();
                    Arrays.sort(c);
                    String sort = String.valueOf(c);
                    String serverSM3 = generateSM3HASH(sort);
                    if(serverSM3.equalsIgnoreCase(clientSM3)){
                        return true;
                    }
                }
			}catch (Exception e){
                e.printStackTrace();
			}
			StringBuffer url = request.getRequestURL();
			return false;
        } else {	
            return super.preHandle(request, response, handler);
        }
	}
	
	//摘要计算
    public static String generateSM3HASH(String src) {
        byte[] md = new byte[32];
        byte[] msg1 = src.getBytes();
        //System.out.println(Util.byteToHex(msg1));
        SM3Digest sm3 = new SM3Digest();
        sm3.update(msg1, 0, msg1.length);
        sm3.doFinal(md, 0);
        String s = new String(Hex.encode(md));
        return s.toUpperCase();
    }
}

```

```js
// jQuery ajaxhook.min.js插件实现
// 接口拦截-数据完整性校验-防重放策略
(function () {
    // 字母字典排序
    const dictSort = (str) => {
        const strArr = String(str).split('');
        return strArr.sort().join('');
    };
    const randomStr = (len = 36) => {
        let randStr = function () {
            var array = new Uint32Array(1);
            window.crypto.getRandomValues(array);
            return  array[0].toString();
        };
        let str = randStr();
        do {
            str += randStr();
        } while (str.length < len)
        return str.substr(0, len);
    };
    ah.proxy({
        // 请求发起前进入
        onRequest: (config, handler) => {
            if (config.url==="update.do"){
                handler.next(config);
            }else if (config.url==="add.do"){
                handler.next(config);
            }else {
                let body = config.body || '';
                // 兼容formdata上传文件
                if (body?.constructor.name === 'FormData') {
                    const formDataStr = {};
                    body.forEach((val, key) => formDataStr[key] = val);
                    delete formDataStr.file;
                    body = Object.keys(formDataStr).map(it => `${it}=${formDataStr[it]}`).join('&');
                }
                const url = config.url;
                let getUrl = url.includes('?') ? url.split('?')[1] : '';
                let data = ''
                if(body){
                    if(getUrl){
                        body = body + "&" + getUrl;
                    }
                }else{
                    body += getUrl
                }
                if(body === '{}') body = ''
                let decodeURI = decodeURIComponent(body).replace(/\+/g,"")
                data = encodeURIComponent(decodeURI)
                const uid = localStorage.getItem('uid');
                // 获取时间差值
                const timestamp = localStorage.getItem('timestamp') || 0;
                // serverTime = now - 差值
                const serverTimestamp = new Date().getTime() - Number(timestamp);
                // 参数按字母字典sort排序
                const clientVal = dictSort(data + uid + serverTimestamp);
                // sm3加密
                const clientSM3 = sm3(clientVal);
                config.headers.clientSM3 = clientSM3;
                config.headers.timestamp = serverTimestamp;
                // 防重放策略参数
                const nonce_client = randomStr();
                const sign_client = sm3(serverTimestamp + nonce_client + uid);
                config.headers.nonce_client = nonce_client;
                config.headers.sign_client = sign_client;
                handler.next(config);
            }

        },
        onError: (err, handler) => {
            handler.next(err);
        },
        onResponse: (response, handler) => {
            handler.next(response);
        }
    })
})();

```
## 防重放攻击

**描述**

重放攻击（ReplayAttacks），是指攻击者发送一个目的主机已接收过的包，来达到欺骗系统的目的。

主机A要给服务器B发送数据请求，重放攻击可由发起者A或攻击者C发起。
※若由发起者A发起，A会恶意重复发送数据请求；
※若由攻击者C发起，攻击者可利用网络监听或者其他方式盗取用户数据请求，再重发该请求给服务器。

**防护建议**

1. 验证码机制
设置验证码，一次有效，每次请求更新新的验证码。适用于登录过程。优点：简单易实现。缺点：影响用户使用。
2. 随机数机制（挑战与应答的机制）
客户端请求服务器时，服务器生成一个随机数返回给客户端，客户端带上这个随机数访问服务器，服务器比对客户端的这个参数，若相同，说明正确，不是重放攻击。
方案漏洞：
将获取随机数的请求和正常数据请求放到请求集合中，并设置全局变量。将响应返回的随机数赋值给全局变量，然后再将全局变量的值赋值给数据请求从而可保证每个请求均带入有效的随机数，重放该请求集合，从而达到重放攻击的目的。

![](images/随机数防重放.png)

3. 时间戳+nonce+签名验证（推荐使用方案）
客户端发送请求时带上当前时间戳，并生成随机nonce，同时签名（签名是为了防止会话被劫持，时间戳和nonce参数被篡改)，服务端对请求时间戳、nonce及签名进行验证，一致则认为是有效请求，否则视为无效请求。

参数说明

|   参数    |  类型   |                             说明                             |
| :-------: | :-----: | :----------------------------------------------------------: |
|   sign    | String  | 请求加密签名，sign的计算结合了请求中的timestamp、nonce、uid以及**加密的消息体**（加入消息体计算sign，可保证请求数据完整性校验）。 |
| timestamp | Integer | 时间戳，首次访问系统从服务端获取，与nonce结合使用，用于防止请求重放攻击。 |
|   nonce   | String  |     随机数，与timestamp结合使用，用于防止请求重放攻击。      |
|    uid    |  Sting  | 用于计算签名和加密数据的key，与用户会话相关且生命周期与之一致，由服务端生成，经加密传输到客户端。 |

**详细解决方案**

1. 用户访问系统，向服务端请求当前系统时间
2. 客户端接收服务端返回的当前系统时间timestamp_server，并计算出与当前客户端时间的差值△timestamp，将其存入浏览器localstorage中。（加固相关信息存储到localstorage中而不是sessionstorage中，是为了防止客户端浏览器跨页面操作时获取不到相关参数）后续时间戳参数均带入客户端时间+△timestamp，即发送请求时的服务端的当前时间
3. 客户端生成仅一次有效的随机字符串nonce_client（防止60s内的攻击）
4. 客户端读取uid（uid由服务端生成，并与用户会话相关，后续完整性校验以及敏感信息传输中均使用到该uid，uid的生成和传输过程详见[uid的生成和传输方案](#uid的生成和传输方案)）
5. 客户端调用签名算法，`sign_client=hash(timestamp+nonce_client+uid)`，sign_client、timestamp、nonce_client三个参数（一般放到自定义header中）随请求发送到服务端（做签名计算是为了防止nonce和timestamp被篡改，uid字段不随请求发送防止中间人劫持）
6. 服务端收到请求后，读取参数值timestamp、nonce_client、sign_client,并对这三个参数做非空判断，若不为空则进行步骤7
7. 验证timestamp参数，服务端当前时间timestamp_now-timestamp是否大于60s或<-5s（默认一次HTTP请求从发出到到达服务器的时间是不会超过60s的，且不会小于0，考虑到计算机精度和多网关业务场景，故判断条件为>60s或<-5s）。若大于60s或<-5s，则请求无效；反之为有效请求，则进行步骤8
8. 验证nonce_client参数，nonce_client在服务端缓存中是否存在，如果存在则请求无效，视为重放攻击；若不存在则进行步骤9，并将nonce_client记录在服务端缓存中，失效时间一般设置为60s
9. 验证sign_client参数，服务端根据请求的会话标识读取该用户的uid参数，并记录前端传回的timestamp、nonce_client参数，调用签名生成算法，得到`sign_server=hash（timestamp+nonce_client+uid）`，验证sign_server是否等于sign_client，若一致，说明参数未被篡改，请求有效，进行请求业务处理流程；若不一致，说明参数被篡改，将请求丢弃。

防重放机制图示
![](images/防重放机制图.png)

**代码实现**

```java
// Spring拦截器实现
import java.util.Enumeration;
import java.util.Objects;
import java.util.concurrent.TimeUnit;

import com.google.common.cache.*;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.bouncycastle.crypto.digests.SM3Digest;
import org.bouncycastle.util.encoders.Hex;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

/**
 * 重放攻击漏洞
 */
public class ReplayAttackInterceptor extends HandlerInterceptorAdapter {
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
		if (handler instanceof HandlerMethod) {	  
//            HandlerMethod handlerMethod = (HandlerMethod) handler;
			// 白名单
			String [] white ={"/update","/add"};
			Enumeration<String> names = request.getParameterNames();
			String path =request.getServletPath();
			for (int i = 0; i < white.length; i++) {
				if(path.indexOf(white[i])!=-1){

					return true;
				}
			}
            if(!"GET".equalsIgnoreCase(request.getMethod())){
            	//客户端时间+△timestamp，即发送请求时的服务端的当前时间
    			Long timestamp = Long.valueOf(request.getHeader("timestamp"));
    			// 客户端生成的仅一次有效的随机字符串
    			String nonce_client = request.getHeader("nonce_client");
    			// 客户端签名算法
    			String sign_client = request.getHeader("sign_client");
    			
    			if(Objects.nonNull(timestamp) && Objects.nonNull(nonce_client) && Objects.nonNull(sign_client)){
    				// 失效时间
                	int seconds = 60;
                	// 服务器当前时间
                	long currentTime = System.currentTimeMillis();
                	long timeDiff = currentTime - timestamp;
                	// 默认一次HTTP请求从发出到到达服务器的时间是不会超过60s的，且不会小于0，考虑到计算机精度和多网关业务场景，故判断条件为>60s或<-5s
                	if(timeDiff > seconds*1000 || timeDiff < -5*1000){
                		return false;
                	}
                	// 验证nonce_client参数
                	if(Objects.nonNull(GuavaCache.get(nonce_client))){
                		return false;
                	}
                	GuavaCache.put(nonce_client, nonce_client);
                	// 验证sign_client参数
                	// 服务端根据请求的会话标识读取该用户的uid参数
                	String uid = (String)request.getSession().getAttribute("uid");
                	// 调用签名算法sign_server=hash（timestamp+nonce_client+uid）
                	String sign_server = generateSM3HASH(timestamp+nonce_client+uid);
                	
                	if(sign_server.equalsIgnoreCase(sign_client)) {
                		return true;
                	}
    			}
    			// StringBuffer url = request.getRequestURL();
    			return false;   
            }
        	return true;
            
        } else {
            return super.preHandle(request, response, handler);
        }
	}
	//摘要计算
    public static String generateSM3HASH(String src) {
        byte[] md = new byte[32];
        byte[] msg1 = src.getBytes();
        //System.out.println(Util.byteToHex(msg1));
        SM3Digest sm3 = new SM3Digest();
        sm3.update(msg1, 0, msg1.length);
        sm3.doFinal(md, 0);
        String s = new String(Hex.encode(md));
        return s.toUpperCase();
    }
    static class GuavaCache {
        // 缓存项最大数量
        private static final long GUAVA_CACHE_SIZE = 100000;
        // 缓存时间(秒)
        private static final long GUAVA_CACHE_SECONDS = 60;
        // 缓存操作对象
        private static LoadingCache<String, Object> GLOBAL_CACHE = null;
        
        static {
            GLOBAL_CACHE = loadCache(new CacheLoader<String, Object>() {
                @Override
                public String load(String key) {
                    // get键 值不存在时，加载新值到该键中
                    return null;
                }
            });
        }
        
        private static LoadingCache<String, Object> loadCache(CaceLoader<String, Object> cacheLoader) {
            LoadingCache<String, Object> cache = CacheBuilder.newBuilder()
                .maxmumSize(GUAVA_CACHE_SIZE)
                .expireAfterAccess(GUAVA_CACHE_SECONDS, TimeUnit.SECONDS)
                .expireAfterWrite(GUAVA_CACHE_SECONDS, TimeUnit.SECONDS)
                // 移除缓存
                .removalListener(new RemovalListener<String, Object>() {
                    @Override
                    public void onRemoval(RemovalNotification<String, Object> arg0) {
                        // 相关逻辑
                    }
                }).recordStats().build(cacheLoader);
            
            return cache;
        }
        
        /**
         * 添加缓存
         */
        public static void put(String key, Object value) {
            GLOBAL_CACHE.put(key,value);
        } 
        
        /**
         * 当get键值不存在时，加载CacheLoader load()方法 加载新值到改键中 当return null 时会抛异常
         */
        public static Object get(String key) {
            try{
                return GLOBAL_CACHE.get(key);
            }catch(Exception e){
                return null;
            }
        }
    }
}
```

## CORS

**描述**

前后端分离的项目，为了保证正常访问，需设置跨域，由于配置不当：	
1. CORS服务端的`Access-Control-Allow-Origin`设置为了\*，并且`Access-Control-Allow-Credentials`设置为false。
2. `Access-Control-Allow-Origin`设置不是固定的，而是根据用户跨域请求数据的Origin来定。不管`Access-Control-Allow-Credentials`设置为了true还是false，任何网站都可以发起请求，并读取对这些请求的响应。

上述两种情况任何一个网站都可以发送跨域请求来获得CORS服务端上的数据，从而造成跨域漏洞。

**防护建议**

1. 配置白名单，仅允许指定的源来跨域请求服务器端的资源，并且服务器会响应。
`Access-Control-Allow-Origin: https://xxxx.com`
```nginx
# 定义合法的Origin列表 解决跨域问题
map $http_origin $allowed_origin {
    default 0;
    "~……http://10\.37\.169\.111(:[0-9+])?$" $http_origin;
    "" 1;
    "null" 1; # 允许没有Origin的情况
}
location / {
    # ...其他配置
    if ($allowed_origin = 0) {
        return 403 "Invalid Origin";
    }
    add_header Access-Control-Allow-Origin $allowed_origin;
}
```
2. @CrossOrigin注解解决跨域问题，在需要允许跨域访问的接口或者类加上该注解就可以，如果用@RequestMapping，则要指定get，或post等才有起效。
`@CrossOrigin(origins="http://localhost:4000")`

## XSS

**描述**

XSS的全称是跨站脚本攻击。它是一种将恶意脚本注入到其他用户信任的网站中的攻击方式。

1. 反射型XSS
2. 存储型XSS
3. DOM型XSS

**代码实现**
```java
// 存储型xss
import lombok.extern.slf4j.Slf4j;
import org.springframework.util.StreamUtils;

import javax.servlet.ReadListener;
import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.io.BufferedReader;
import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.*;
import java.util.regex.Pattern;

@Slf4j
public class XssAndSqlHttpServletRequestWrapper extends HttpServletRequestWrapper {

    HttpServletRequest orgRequest = null;
    private Map<String, String[]> parameterMap;
    /**
     * 用于保存读取body中数据
      */
    private final byte[] body;

    /**
     * sql keywords pattern
     */
    private static String[] sqlKeywords = {"and", "or", "insert", "select", "delete", "update", "count", "chr", "mid", "master", "truncate", "char", "declare", "backup", "script", "exec"};


    /**
     * xss pattern
     */
    private static Pattern[] patterns = {
        // Avoid anything between script tags
        Pattern.compile("<[\r\n| | ]*script[\r\n| | ]*>(.*?)</[\r\n| | ]*script[\r\n| | ]*>", Pattern.CASE_INSENSITIVE),
        // Remove any lonesome <script ...> tag
        Pattern.compile("<[\r\n| | ]*script(.*?)>", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL),
        // Remove any lonesome </script> tag
        Pattern.compile("</[\r\n| | ]*script[\r\n| | ]*>", Pattern.CASE_INSENSITIVE),
        // Avoid anything in a src="http://www.xxx.com/xx/java/..." type of expression
        Pattern.compile("src[\r\n| | ]*=[\r\n| | ]*[\\\"|\\\'](.*?)[\\\"|\\\']", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL),
        // Avoid eval(...) expressions
        Pattern.compile("eval\\((.*?)\\)", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL),
        // Avoid e-xpression(...) expressions
        Pattern.compile("e-xpression\\((.*?)\\)", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL),
        // Avoid javaScript:... expressions
        Pattern.compile("javascript[\r\n| | ]*:[\r\n| | ]*", Pattern.CASE_INSENSITIVE),
        // Avoid vbScript:... expressions
        Pattern.compile("vbscript[\r\n| | ]*:[\r\n| | ]*", Pattern.CASE_INSENSITIVE),
        // Avoid onload= expressions
        Pattern.compile("onload(.*?)=", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL),
        // 特殊字符,不使用的特殊字符直接过滤掉，如果需要使用，不在这里配置，在下面转换为全角
        Pattern.compile("[|&;$%@',\"<>()+*\\r\\n\\\\]"),
        Pattern.compile("(0x0a)|(0x0d)"),
        // 关键字
        Pattern.compile(addSqlKeyWords(sqlKeywords), Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL)
    };

    private static String addSqlKeyWords(String[] map) {
        StringBuilder sb = new StringBuilder();
        for (String key: map) {
            sb.append("(").append(key).append(" )").append("|");
        }
        sb.deleteCharAt(sb.length() - 1);
        return sb.toString();
    }

    public XssAndSqlHttpServletRequestWrapper(HttpServletRequest request) throws IOException {
        super(request);
        orgRequest = request;
        parameterMap = request.getParameterMap();
        body = StreamUtils.copyToByteArray(request.getInputStream());
    }

    // 重写几个HttpServletRequestWrapper中的方法↓

    /**
     * 获取所有参数名
     *
     * @return 返回所有参数名
     */
    @Override
    public Enumeration<String> getParameterNames() {
        Vector<String> vector = new Vector<String>(parameterMap.keySet());
        return vector.elements();
    }

    /**
     * 覆盖getParameter方法，将参数名和参数值都做xss & sql过滤。<br/>
     * 如果需要获得原始的值，则通过super.getParameterValues(name)来获取<br/>
     * getParameterNames,getParameterValues和getParameterMap也可能需要覆盖
     */
    @Override
    public String getParameter(String name) {
        String[] results = parameterMap.get(name);
        if (results == null || results.length <= 0) {
            return null;
        } else {
            String value = results[0];
            if (value != null) {
                value = xssEncode(value);
            }
            return value;
        }
    }

    /**
     * 获取指定参数名的所有值的数组，如：checkbox的所有数据 接收数组变量 ，如checkobx类型
     */
    @Override
    public String[] getParameterValues(String name) {
        String[] results = parameterMap.get(name);
        if (results == null || results.length <= 0) {
            return null;
        } else {
            int length = results.length;
            for (int i = 0; i < length; i++) {
                results[i] = xssEncode(results[i]);
            }
            return results;
        }
    }

    /**
     * 覆盖getHeader方法，将参数名和参数值都做xss & sql过滤。<br/>
     * 如果需要获得原始的值，则通过super.getHeaders(name)来获取<br/>
     * getHeaderNames 也可能需要覆盖
     */
    @Override
    public String getHeader(String name) {

        String value = super.getHeader(name);
        if (value != null) {
            value = xssEncode(value);
        }
        return value;
    }

    /**
     * 将容易引起xss & sql漏洞的半角字符直接替换成全角字符
     *
     * @param s
     * @return
     */
    private static String xssEncode(String s) {
        if (s == null || s.isEmpty()) {
            return s;
        } else {
            s = stripXSSAndSql(s);
        }
        StringBuilder sb = new StringBuilder(s.length() + 16);
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            switch (c) {
                case '>':
                    // 转义大于号
                    sb.append("＞");
                    break;
                case '<':
                    // 转义小于号
                    sb.append("＜");
                    break;
                case '\'':
                    // 转义单引号
                    sb.append("＇");
                    break;
                case '\"':
                    // 转义双引号
                    sb.append("＂");
                    break;
                case '&':
                    // 转义&
                    sb.append("＆");
                    break;
                case '#':
                    // 转义#
                    sb.append("＃");
                    break;
                case '(':
                    // 转义#
                    sb.append("（");
                    break;
                case ')':
                    // 转义#
                    sb.append("）");
                    break;
                case ';':
                    // 转义#
                    sb.append("；");
                    break;
                default:
                    sb.append(c);
                    break;
            }
        }
        return sb.toString();
    }

    /**
     * 获取最原始的request
     *
     * @return
     */
    public HttpServletRequest getOrgRequest() {
        return orgRequest;
    }

    /**
     * 获取最原始的request的静态方法
     *
     * @return
     */
    public static HttpServletRequest getOrgRequest(HttpServletRequest req) {
        if (req instanceof XssAndSqlHttpServletRequestWrapper) {
            return ((XssAndSqlHttpServletRequestWrapper) req).getOrgRequest();
        }

        return req;
    }

    /**
     * 防止xss跨脚本攻击（替换，根据实际情况调整）
     */

    public static String stripXSSAndSql(String value) {
        if (value != null) {
            for (Pattern pattern : patterns) {
                value = pattern.matcher(value).replaceAll("");
            }
        }
        return value;
    }

    public boolean checkXSSAndSql(String value) {
        if (value != null) {
            for (Pattern pattern : patterns) {
                if (pattern.matcher(value).find()) {
                    return true;
                }
            }
        }
        return false;
    }

    public final boolean checkParameter() {
        Map<String, String[]> submitParams = new HashMap(parameterMap);
        Set<String> submitNames = submitParams.keySet();
        for (String submitName : submitNames) {
            Object submitValues = submitParams.get(submitName);
            if ((submitValues instanceof String)) {
                if (checkXSSAndSql((String) submitValues)) {
                    return true;
                }
            } else if ((submitValues instanceof String[])) {
                for (String submitValue : (String[]) submitValues) {
                    if (checkXSSAndSql(submitValue)) {
                        return true;
                    }
                }
            }
        }
        return false;
    }

    @Override
    public BufferedReader getReader() throws IOException {
        return new BufferedReader(new InputStreamReader(getInputStream()));
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {
        final ByteArrayInputStream bais = new ByteArrayInputStream(body);
        return new ServletInputStream() {

            @Override
            public int read() throws IOException {
                return bais.read();
            }

            @Override
            public boolean isFinished() {
                return false;
            }

            @Override
            public boolean isReady() {
                return false;
            }

            @Override
            public void setReadListener(ReadListener arg0) {}
        };
    }
}

// 过滤器中实现
@WebFilter("/*")  // 过滤所有请求
public class XSSFilter implements Filter {
    
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        // 初始化代码
    }
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                         FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        // 创建包装器
        HttpServletRequest wrappedRequest = new XssAndSqlHttpServletRequestWrapper(httpRequest);
        // 继续过滤器链，传递包装后的请求
        chain.doFilter(wrappedRequest, response);
    }
    
    @Override
    public void destroy() {
        // 清理代码
    }
}
```

## CSRF

**描述**

CSRF，全称Cross-site request forgery，即跨站请求伪造，是指利用受害者尚未失效的身份认证信息（cookie、会话等），诱骗其点击恶意链接或者访问包含攻击代码的页面，在受害人不知情的情况下以受害者的身份向（身份认证信息所对应的）服务器发送请求，从而完成非法操作（如转账、改密等）。

**防护建议**

1. 验证HTTP referer字段
在后端程序接收到请求时，应检查请求的来源地址（HTTP Referer）是否合法（利用白名单），如果请求来源不在指定的名单中，即视为CSRF。
```nginx
# 定义合法的Referer
# none:允许没有Referer头的请求。blocked:允许Referer头存在但值被防火墙或代理服务器修改、删除（值为空）的请求。server_names:允许Referer头与server_name中定义的其中一个域名匹配的请求。
# 如果配置不管用,可以看下location里面的配置
valid_referers none blocked server_names yoursite.com *.trusted-site.com;
if ($invalid_referer) {
    # 返回403禁止访问
    return 403;
}

# location块referer配置(后端代码没有referer配置的话 启用nginx referer配置)
set $allow_referer 1;

if($http_referer !~ "^http://10\.39\.169\.22(:[0-9]+)?") {
    set $allow_referer 0;
}

if($allow_referer = 0){
    return 403
}
```
2. 随机token
对于敏感的网页，可在请求参数或者请求header中增加不可伪造的token等信息用户防护CSRF攻击(可利用防重放攻击中的sign当作token)。

## SQL注入

**代码实现**

```java
import java.util.Enumeration;
import java.util.Locale;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.lang3.StringUtils;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;
/**
 * sql注入拦截
 */
public class SQLInterceptor implements HandlerInterceptor {

	@Override
	public void afterCompletion(HttpServletRequest arg0,
			HttpServletResponse arg1, Object arg2, Exception arg3)
    {
	
	}

	@Override
	public void postHandle(HttpServletRequest arg0, HttpServletResponse arg1,
			Object arg2, ModelAndView arg3)  {
		
	}

	public boolean preHandle(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2)  {
		Enumeration<String> names = arg0.getParameterNames();  
        while(names.hasMoreElements()){  
            String name = names.nextElement();  
            String[] values = arg0.getParameterValues(name);
            if(values!=null){
                for(String value: values){
                    //sql注入直接拦截
                    Locale.setDefault(Locale.ENGLISH);
                    if(judgeSQLInject(value.toLowerCase())){
                        return false;
                    }
                    //跨站xss清理
                    clearXss(value);
                }
            }
        }  
        return true;  
	}
	
	/**
	 * 校验sql注入字符串
	 */
    public boolean judgeSQLInject(String value){  
        if(value == null || StringUtils.isBlank(value)){
            return false;
        } 
        //insert|and|or|select|update|delete|drop|truncate|%20|=|--|;|'|%|#|+|,|//|/|\\|!=|(|)
//        String sqlStr = ">|<|script| insert | and | or | select | update | delete | drop | truncate |%20|-|--|'|%|+|//|/|\\|!=|(|)";
        String sqlStr = ">|<|script| insert | and | or | select | update | delete | drop | truncate |%20|--|%|+|//|\\|!=|(|)";  
        String[] sqlArr = sqlStr.split("\\|");  
        for(int i=0;i<sqlArr.length;i++){  
            if(value.indexOf(sqlArr[i])>-1){
            	if(!isSpecialParam(value)){

            		return true;	
            	}
            }  
        }  
        return false;  
    }
    
    /**
     * 针对用到的必要特殊参数字符在此判断
     */
    public boolean isSpecialParam(String value){
    	boolean isSpecial = false;//是否为特殊字符串
//    	Pattern pattern = Pattern.compile("[0-9]-[0-9]|.*image/octet-stream;base64.*|[0-9] [0-9]");
    	Pattern pattern = Pattern.compile(".*image/octet-stream;base64.*");
		Matcher matcher = pattern.matcher(value);
		if(matcher.find()){
			isSpecial = true;
		}
		return isSpecial;
    }
    
    /**
     * 处理跨站xss字符转义
     */
    private String clearXss(String value) {
        if (value == null || StringUtils.isBlank(value)) {
            return value;
        }
        value = value.replaceAll("<", "").replaceAll(">", "").replace("'","");
        value = value.replaceAll("eval\\((.*)\\)", "");
        value = value.replaceAll("[\\\"\\\'][\\s]*javascript:(.*)[\\\"\\\']",
                "\"\"");
        value = value.replace("script", "");
        return value;
    }
}
```



