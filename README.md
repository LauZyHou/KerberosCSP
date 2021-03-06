# KerberosCSP
Kerberos formal verification in using CSP.

## 基本流程

```
AS() = 收到Client的验证请求 -> 在KDC数据库中查找用户主体名称 -> if 找到匹配的项 then 获取主体私钥userKey -> 生成会话密钥sessionKey1和票证TGT1,用userKey加密为resp -> 向Client回复resp -> AS();
															  else 回复错误消息 -> AS()

Client_Pre() = 向AS发送验证请求 -> 收到AS的resp -> if 正确消息 then 密钥解密resp拿到sessionKey1和TGT1 -> Client_Mid();
												  else Client_Pre();

Client_Mid() = 向TGS发送请求(服务主体名称,TGT1,使用sessionKey1加密的验证者) -> 从TGS收消息 -> if 正确消息(即加密后的sessionKey2和TGT2) -> 解密 -> Client_Post();
																						   else Client_Pre();

TGS() = 收到Client的请求 ->  解密TGT1得到sessionKey1,再用sessionKey1解密得到验证者,再验证服务主体
				-> (if 成功验证 then 为用户主体和服务器生成sessionKey2,TGT2 -> 使用sessionKey1加密sessionKey2和TGT2 -> 回复这些信息给Client
				   else  回复错误消息) -> TGS();

Client_Post() = 向AppServ发送TGT2和用sessionKey2加密的验证者 -> 收到AppServ的信息 -> if 信息是允许访问 -> Client_Use();
																				   else -> Client_Pre();

Client_Use() = 向AppServ发送访问的具体信息 -> 接受来自AppServ访问的具体信息 -> if 不是错误信息 then Client_Use() else Stop;


AppServ() = [收到Client的验证信息(TGT2和验证者) -> 用sessionKey2解密 -> if 验证通过 then 回复通过 -> AppServ()] | [收到Client的访问信息 -> (if 已验证 then 回复访问信息 else 回复错误) -> AppServ()];
```


# SSHCSP
SSH formal verification in using CSP.

## 基本流程

```
Client_Pre() = 向Server发送访问请求 -> 接收来自Server的消息 -> (if 没有找到公钥 -> Client_Pre() else 找到公钥收到加密质询 -> 解密质询 -> 向Server发送解密后的质询 -> Client_Post());


Client_Post() = 接收server的消息 -> (if 验证通过 then Client_use() else client_Pre());


Client_Use() = 向Server发送访问的具体信息 -> 接受来自Server访问的具体信息 -> (if 不是错误信息 then Client_Use() else Stop);


Server() = [收到Client的访问请求->在文件中找公钥是否存在-> (if 存在 then 发送加密的质询 else 回复错误) -> Server()] 
		  | [收到Client的解密后的质询 -> (和加密前的质询对比 -> if 一致 then 回复通过 else 回复错误) -> Server()]
		  | [收到Client的访问信息 -> (if 已验证 then 回复访问信息 else 回复错误) -> Server()];

```



