---
layout: post
title:  "Cryptography_암호학 개요"
tags: "cryptography"
date:   2023-08-23 00:00:00
---

암호학은 우리의 현재와 미래에 밀접한 관련성을 가지는 분야입니다. 우리는 알게 모르게 암호학의 지식을 활용한 기술들을 사용하고 있으며 저는 앞으로도 더 많이 사용하게 될 것이라 생각합니다.

암호학은 정보의 기밀성, 무결성, 인증, 부인방지 등을 보장하기 위한 분야입니다. 여기서 기밀성은 민감한 정보가 외부에 노출되지 않도록 보호하는 원칙을 나타내며, 무결성은 데이터가 변조되지 않았음을 확신시켜줍니다. 인증은 통신 상대가 믿을 수 있는지를 확인하며, 부인방지는 메시지 송신자가 자신의 행동을 부인할 수 없도록 보장합니다. 

암호 시스템의 기호 표현으로는 C=Ek(P)가 있습니다. 여기서 P는 PlainText로 평문, 송신자가 수신자에 전송하고자 하는 정보입니다. C는 CipherText로 암호문이고 수신자가 받는 정보입니다. E는 Encryption 알고리즘을 의미하고 k는 키입니다. 그래서 C=Ek(P)는 평문P을 키k로 암호화(E)하여 암호문(C)를 얻는다는 것을 말합니다. 반대로 P=Dk(C)는 D는 Decryption 알고리즘으로 암호문C를 키k로 복호화(D)하여 평문(P)를 얻는다는 것을 의미합니다. 수신자는 받은 C를 복호화하여 정보P를 얻습니다. 

이때, 암화하 할 때의 키와 복호화 할 때의 키가 같다면 대칭키 암호 알고리즘이라고 하고 다르다면 비대칭키 암호 알고리즘이라고 말합니다.
<br>
<br>

1. [대칭키 암호](#대칭키-암호)
    
2. [비대칭키 암호](#비대칭키-암호)

3. [해시 함수](#해시-함수)
<br>
<br>
<br>

### **대칭키 암호**
대칭키 암호는 암호화와 복호화에 같은 비밀 키를 사용하는 알고리즘입니다. 주요 장점은 빠른 처리 속도와 간단한 구현이지만, 키 교환과 관리의 어려움이 있습니다. 하위 분류로는 크게 블록 암호, 스트림 암호, 해시 함수가 있습니다.

- ***블록 암호*** : 블록 암호는 데이터를 고정 크기의 블록 단위로 나누어 암호화하는 알고리즘입니다. 이러한 블록 단위로 데이터를 처리하기 때문에 블록 크기에 맞게 데이터를 패딩(padding)하여 처리하는 방식을 사용합니다. 대표적인 블록 암호로는 AES, DES, 3DES 등이 있습니다.

- ***스트림 암호*** : 스트림 암호는 데이터를 비트 단위로 처리하여 암호화하는 알고리즘입니다. 스트림 암호는 키 스트림을 생성하여 원본 데이터와 XOR 연산을 수행하여 암호화를 진행합니다.

<br>
<br>

### **비대칭키 암호**
비대칭키 암호는 암호화와 복호화에 서로 다른 두 개의 키를 사용하는 암호화 기술입니다. 이러한 키 중 하나는 공개키(Public Key)로 불리며, 누구나 알 수 있습니다. 다른 하나는 개인키(Private Key)로 불리며, 해당 키를 가진 사용자만이 알고 있어야 합니다. 공개키와 개인키는 서로 연결되어 있지만, 한 키로 암호화하면 다른 키로만 복호화할 수 있습니다.

비대칭키 암호의 주요 목적은 통신의 안전성과 무결성을 확보하며, 대칭키 암호의 키 교환 문제를 해결합니다. 그러나 대칭키 암호에 비해 계산 복잡성이 높아 처리 속도가 느리다는 단점이 있습니다. 대표적으로 RSA, ECC, Elgama 알고리즘 등이 있습니다. 

<br>
<br>

### **해시 함수**
해시 함수는 임의 길이의 입력 데이터를 고정 길이의 해시 값으로 매핑하는 함수입니다. 해시 함수는 데이터의 무결성을 확인하거나, 암호화되지 않은 비밀번호를 안전하게 저장하기 위해 사용될 수 있습니다. 일방향성이고 메시지 인증 코드과 디지털 서명에도 사용됩니다. 대표적인 해시 함수로는 SHA-256, MD5 등이 있습니다.

**(해시 함수는 정보의 무결성을 보장하지만, 기밀성은 보장하지 않습니다. 메시지 인증 코드는 무결성과 인증을 제공하고 디지털 서명은 무결성과 부인 방지, 인증을 제공합니다. 여기서 디지털 서명과 메시지 인증 코드의 차이점은 디지털 서명은 공개키 + 해시함수를 이용하는 반면 메시지 인증 코드는 대칭키 + 해시함수를 이용합니다. 디지털 서명은 공개키를 이용하기 때문에 속도가 느릴 수 있다는 단점이 있지만 메시지 인증 코드에 비해 더 높은 보안성을 가집니다.)**