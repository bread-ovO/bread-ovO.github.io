### 什么是 JSON Web Token (JWT)?

JSON Web Token (JWT) 是一个开放标准 (RFC 7519)，它定义了一种紧凑且自包含的方式，用于在各方之间安全地传输 JSON 对象形式的信息 。由于 JWT 可以被签名（使用密钥的 HMAC 算法）或使用 RSA 或 ECDSA 的公钥/私钥对，因此这些信息可以被验证和信任 。

### JWT 的结构

JWT 在其紧凑形式中，由句点（`.`）分隔的三个部分组成：

1. **Header（头部）**
2. **Payload（载荷）**
3. **Signature（签名）**

因此，一个 JWT 通常如下所示：

`xxxxx.yyyyy.zzzzz`

#### 1. Header（头部）

头部通常由两部分组成：token 的类型（即 JWT）和所使用的签名算法，例如 HMAC SHA256 或 RSA 。例如：

```json

{   
	"alg": "HS256",   
	"typ": "JWT" 
}

```

然后，这个 JSON 被 Base64Url 编码，形成 JWT 的第一部分。

#### 2. Payload（载荷）

令牌的第二部分是载荷，它包含声明（claims）。声明是关于实体（通常是用户）以及其他数据的声明。有三种类型的声明：

- **Registered claims（注册声明）**：这是一组预定义的声明，它们不是强制性的，但建议使用，以提供一组有用的、可互操作的声明。其中一些包括：`iss` (issuer)、`exp` (expiration time)、`sub` (subject)、`aud` (audience) 等。
- **Public claims（公共声明）**：这些声明可以由使用 JWT 的人随意定义。但是为了避免冲突，它们应该在 IANA JSON Web Token Registry 中定义，或者定义为包含抗冲突命名空间的 URI。
- **Private claims（私有声明）**：这些是自定义声明，用于在各方之间共享信息，这些各方都同意使用它们，既不是注册声明也不是公共声明。

一个载荷示例可能如下所示：

json

`{   "sub": "1234567890",   "name": "John Doe",   "admin": true }`

然后，载荷被 Base64Url 编码，形成 JSON Web Token 的第二部分。请注意，对于已签名的令牌，此信息虽然受到篡改保护，但可以被任何人读取。除非加密，否则不要将秘密信息放在 JWT 的载荷或头部元素中。

#### 3. Signature（签名）

要创建签名部分，您必须获取编码的头部、编码的载荷、一个密钥以及头部中指定的算法，并对其进行签名。例如，如果你要想使用 HMAC SHA256 算法，则将按以下方式创建签名：

`HMACSHA256(   base64UrlEncode(header) + "." +   base64UrlEncode(payload),   secret)`

签名用于验证消息在传输过程中是否被更改，并且在使用私钥签名的令牌的情况下，还可以验证 JWT 的发送者是否是其声称的发送者 [2](https://jwt.io/introduction)。

### JWT 的工作原理

在身份验证中，当用户使用其凭据成功登录后，将返回一个 JSON Web Token 。由于令牌是凭据，因此必须格外小心以防止出现安全问题。

每当用户想要访问受保护的路由或资源时，用户代理应发送 JWT，通常在 `Authorization` 标头中使用 Bearer 模式。标头的内容应如下所示：

`Authorization: Bearer <token>`

服务器的受保护路由将检查 `Authorization` 标头中是否存在有效的 JWT，如果存在，则允许用户访问受保护的资源。如果 JWT 包含必要的数据，则可以减少查询数据库以进行某些操作的需求，但这可能并不总是这样。

### 何时应该使用 JSON Web Token？

以下是 JSON Web Token 有用的一些场景:

- **授权**：这是使用 JWT 最常见的场景。一旦用户登录，每个后续请求都将包含 JWT，允许用户访问该令牌允许的路由、服务和资源。单点登录 (Single Sign On) 是一项广泛使用 JWT 的功能，因为它开销小且易于在不同域中使用。
- **信息交换**：JSON Web Token 是一种安全地在各方之间传输信息的好方法。因为 JWT 可以被签名——例如，使用公钥/私钥对——您可以确保发送者是他们所说的那个人。此外，由于签名是使用头部和载荷计算的，因此您还可以验证内容是否已被篡改。

### 使用 PyJWT 库

PyyJWT 是一个流行的python 库，用于实现 JSON Web Tokens。

**安装**
```bash
pip3 install PyJWT
```

**示例**

以下是一个使用 python 的简单示例：

```python

#!/usr/bin/env python3  
import sys  
import time  
import jwt  
  
# Open PEM  
private_key = """your_private_key"""  
  
payload = {  
    'iat': int(time.time()) - 30,  
    'exp': int(time.time()) + 900,  
    'sub': 'your_sub'  
}  
headers = {  
    'kid': 'your_kid'  
}  
  
# Generate JWT  
encoded_jwt = jwt.encode(payload, private_key, algorithm='EdDSA', headers = headers)  
  
print(f"JWT:  {encoded_jwt}")

```



**为什么要使用 JWT 而不是 api**

1.JWT 是一个自包含的令牌，它包含了所有必要的信息，如用户身份、权限以及令牌的有效期。这意味着，服务器无需查询数据库或其他外部存储来验证令牌的有效性。

2.JWT 会被服务器签名，确保它在传输过程中没有被篡改。通过公私钥的方式，服务器可以验证 JWT 的完整性和来源，防止伪造。

3.JWT 通常包括一个过期时间（exp），确保令牌在一段时间后失效，从而减少了长期暴露的风险。相比之下，API Key 可能会被长期使用，增加了被滥用的风险。

4.JWT 可以在不同的服务和系统之间传递和验证，这使得它非常适合微服务架构。在这种架构下，每个服务都可以独立验证 JWT，无需集中式验证机制。