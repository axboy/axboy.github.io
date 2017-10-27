---
layout: post
title: "java导入cer证书"
description: ""
categories: java
tags: java spring
# author: "炮灰哥"
---

### 前言

临近国庆、中秋连假，比年假都长，大半年了肯定回家一趟。然而发现比春运车票都难买，怎么办呢？刷咯。

我是在京东下单购买的车票，由京东刷，然后托管。

然后我在想，自己能不能实现一个刷票应用，购买暂且不做，暂无方案识别12306的验证码。那就先完成获取车票信息，短信提醒我去买票。

说干就干，我们平时买火车票到12306购买，总会出现证书错误的情况，浏览器上不导入证书也能购买，可是，java程序怎么办呢？这篇博客就来解决java证书问题。

### 项目说明

_Spring boot + groovy + mongodb_

未使用证书时，使用RestTemplate请求，错误信息关键词如下

```
unable to find valid certification path to requested target
```

### 解决

- 方案1，导入证书到java证书库

    导入方法如下

    ```sh
    # 添加证书,默认密码为changeit
    keytool -keystore "${JAVA_HOME}/jre/lib/security/cacerts" \
            -importcert -alias ca12306 \
            -file ./srca.cer

    # 查看证书
    keytool -list -alias ca12306 -keystore "${JAVA_HOME}/jre/lib/security/cacerts"

    #删除证书
    keytool -delete -alias ca12306 -keystore "${JAVA_HOME}/jre/lib/security/cacerts"
    ```

- 方案2，httpClient配置证书

    1. 下载网站的证书

    2. 利用jdk的toolkey工具，将证书转换成密钥的形式

        ```sh
        keytool -import -alias cacerts -file ./srca.cer \
            -keystore srca.keystore
        # 会提示输入keystore密码，随便输，但要记住，后面有用
        ```

    3. 配置httpClient、restTemplate

        spring的配置，我使用的是java配置，具体代码如下

    - application.yml

    ```
    cn.wazitang.train:
    srcaKeyStore: classpath:train/srca.keystore
    srcaPassword: 123456        # 填入生成keystore密钥时输入的密码
    ```

    - RestTemplateConf.groovy

    ```groovy
    package cn.wazitang.train.conf
    import ...

    @Configuration
    class RestTemplateConf {

        @Value('${cn.wazitang.train.srcaKeyStore}')
        private Resource srcaKeystore

        @Value('${cn.wazitang.train.srcaPassword}')
        private String srcaPassword

        @Bean
        HttpMessageConverters httpMessageConverters() {
            return new HttpMessageConverters(true, [
                    new StringHttpMessageConverter(Charset.forName("UTF-8"))
            ])
        }

        @Bean
        HttpClient trainHttpClient() {
            String password = srcaPassword
            KeyStore trustStore = KeyStore.getInstance(KeyStore.getDefaultType())
            InputStream is = srcaKeystore.getInputStream()
            trustStore.load(is, password.toCharArray())
            is.close()
            SSLContext sslContext = SSLContexts.custom()
                    .loadTrustMaterial(trustStore, new TrustSelfSignedStrategy())
                    .build()
            SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext)
            return HttpClients.custom().setSSLSocketFactory(sslsf).build()
        }

        @Bean
        ClientHttpRequestFactory trainClientHttpRequestFactory(HttpClient trainHttpClient) {
            return new HttpComponentsClientHttpRequestFactory(trainHttpClient)
        }

        /**
        * 配置了12306证书的restTemplate，用于向12306请求数据
        */
        @Bean
        @Scope("prototype")
        RestTemplate trainRestTemplate(
                RestTemplateBuilder restTemplateBuilder,
                ClientHttpRequestFactory trainClientHttpRequestFactory) {
            return restTemplateBuilder.requestFactory(trainClientHttpRequestFactory).build()
        }

        /**
        * 普通的restTemplate
        */
        @Bean
        @Scope("prototype")
        RestTemplate restTemplate(RestTemplateBuilder restTemplateBuilder) {
            return restTemplateBuilder.build()
        }
    }
    ```
    
### 测试

以获取站台信息为例，直接上代码吧

```groovy
package cn.wazitang.train
import ...

@RunWith(SpringRunner)
@SpringBootTest
class InitDatabase {

    @Autowired
    private RestTemplate restTemplate


    @Autowired
    private RestTemplate trainRestTemplate

    @Test
    void start() {
        //截取等号后面数据，并去除单引号
        getStationStr()
                .split("=")[1]
                .replaceAll("'", "")
                .replaceAll(";", "")
                .replaceFirst("@", "")
                .split("@")
                .each { data ->
            println data
        }
    }

    private String getStationStr() {
        String url = "https://kyfw.12306.cn/otn/resources/js/framework/station_name.js?station_version=1.9025"
        return trainRestTemplate.getForEntity(url, String).getBody()        # 把该行trainRestTemplate换成普通的restTemplate试试
    }
}
```

### 踩坑过程中参考的一些文章

- [添加证书到证书库](http://blog.csdn.net/u014783753/article/details/44619625)

- [cer证书转换](http://blog.csdn.net/liuxiao723846/article/details/52695549)

- [httpClient配置](http://www.cnblogs.com/grefr/p/6088044.html)