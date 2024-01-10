# `v2ray-core\proxy\vmess\aead\consts.go`

```
# 定义常量 KDFSaltConst_AuthIDEncryptionKey，用于身份验证 ID 加密的密钥
const KDFSaltConst_AuthIDEncryptionKey = "AES Auth ID Encryption"

# 定义常量 KDFSaltConst_AEADRespHeaderLenKey，用于响应头长度的密钥
const KDFSaltConst_AEADRespHeaderLenKey = "AEAD Resp Header Len Key"

# 定义常量 KDFSaltConst_AEADRespHeaderLenIV，用于响应头长度的初始化向量
const KDFSaltConst_AEADRespHeaderLenIV = "AEAD Resp Header Len IV"

# 定义常量 KDFSaltConst_AEADRespHeaderPayloadKey，用于响应头载荷的密钥
const KDFSaltConst_AEADRespHeaderPayloadKey = "AEAD Resp Header Key"

# 定义常量 KDFSaltConst_AEADRespHeaderPayloadIV，用于响应头载荷的初始化向量
const KDFSaltConst_AEADRespHeaderPayloadIV = "AEAD Resp Header IV"

# 定义常量 KDFSaltConst_VMessAEADKDF，用于 VMess 的 AEAD 密钥派生函数
const KDFSaltConst_VMessAEADKDF = "VMess AEAD KDF"

# 定义常量 KDFSaltConst_VMessHeaderPayloadAEADKey，用于 VMess 头部载荷的 AEAD 密钥
const KDFSaltConst_VMessHeaderPayloadAEADKey = "VMess Header AEAD Key"

# 定义常量 KDFSaltConst_VMessHeaderPayloadAEADIV，用于 VMess 头部载荷的 AEAD 初始化向量
const KDFSaltConst_VMessHeaderPayloadAEADIV = "VMess Header AEAD Nonce"

# 定义常量 KDFSaltConst_VMessHeaderPayloadLengthAEADKey，用于 VMess 头部载荷长度的 AEAD 密钥
const KDFSaltConst_VMessHeaderPayloadLengthAEADKey = "VMess Header AEAD Key_Length"

# 定义常量 KDFSaltConst_VMessHeaderPayloadLengthAEADIV，用于 VMess 头部载荷长度的 AEAD 初始化向量
const KDFSaltConst_VMessHeaderPayloadLengthAEADIV = "VMess Header AEAD Nonce_Length"
```