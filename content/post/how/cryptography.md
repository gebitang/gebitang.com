+++
title = "Encrypt & Decrypt"
description = "basic info about cryptography"
tags = [
    "crypt"
]
date = "2020-10-27"
topics = [
    "crypt"
]
toc = true
+++

一文入门[Cipher Suites: Ciphers, Algorithms and Negotiating Security Settings](https://www.thesslstore.com/blog/cipher-suites-algorithms-security-settings/)

[The Transport Layer Security (TLS) Protocol Version 1.2](https://tools.ietf.org/html/rfc5246), [Version 1.3](https://tools.ietf.org/html/rfc8446)

Cipher suites are named combinations of:

- Key Exchange Algorithms (RSA, DH, ECDH, DHE, ECDHE, PSK)
- Authentication/Digital Signature Algorithm (RSA, ECDSA, DSA)
- Bulk Encryption Algorithms (AES, CHACHA20, Camellia, ARIA)
- Message Authentication Code Algorithms (SHA-256, POLY1305)

DH for [Diffie–Hellman key exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange), [What is the Diffie–Hellman key exchange and how does it work?](https://www.comparitech.com/blog/information-security/diffie-hellman-key-exchange/), [“Diffie-Hellman Key Exchange” in plain English](https://security.stackexchange.com/questions/45963/diffie-hellman-key-exchange-in-plain-english)

- [What is RSA encryption and how does it work?](https://www.comparitech.com/blog/information-security/rsa-encryption/)
- [常见加密算法分类,用途,原理以及比较](https://blog.csdn.net/qq_21794823/article/details/53114819)
- [离散对数加密算法](https://blog.csdn.net/chen77716/article/details/7106485)
- [证明与计算(2): 离散对数问题(Discrete logarithm Problem, DLP)](https://www.cnblogs.com/math/p/discrete-log.html)

[Signal protocol 开源协议理解](https://blog.csdn.net/andylau00j/article/details/82870841)  
[【翻译】WhatsApp 加密概述（技术白皮书）](https://www.cnblogs.com/over140/p/8683171.html)  
[Signal协议与系统分析一：需求与特性篇](https://zhuanlan.zhihu.com/p/85366605)  
[Signal协议与系统分析二：三方密钥协商协议X3DH](https://zhuanlan.zhihu.com/p/85454061)  
[理解椭圆函数加密（初等内容介绍）](https://zhuanlan.zhihu.com/p/42983540)
[理解零知识证明和协议](https://zhuanlan.zhihu.com/p/43575662)

--- 

[official doc](https://signal.org/docs/)

[XEdDSA and VXEdDSA](https://signal.org/docs/specifications/xeddsa/)

This document describes how to create and verify EdDSA-compatible signatures using public key and private key formats initially defined for the X25519 and X448 elliptic curve Diffie-Hellman functions. This document also describes "VXEdDSA" which extends XEdDSA to make it a verifiable random function, or VRF.

[X3DH](https://signal.org/docs/specifications/x3dh/)

This document describes the "X3DH" (or "Extended Triple Diffie-Hellman") key agreement protocol. X3DH establishes a shared secret key between two parties who mutually authenticate each other based on public keys. X3DH provides forward secrecy and cryptographic deniability.

[Double Ratchet](https://signal.org/docs/specifications/doubleratchet/)

This document describes the Double Ratchet algorithm, which is used by two parties to exchange encrypted messages based on a shared secret key. The parties derive new keys for every Double Ratchet message so that earlier keys cannot be calculated from later ones. The parties also send Diffie-Hellman public values attached to their messages. The results of Diffie-Hellman calculations are mixed into the derived keys so that later keys cannot be calculated from earlier ones. These properties give some protection to earlier or later encrypted messages in case of a compromise of a party's keys.

[Sesame](https://signal.org/docs/specifications/sesame/)

This document describes the Sesame algorithm for managing message encryption sessions in an asynchronous and multi-device setting.
