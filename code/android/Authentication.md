# Android 7.1 Authentication
## 官方介绍
https://source.android.com/security/authentication/

## gateKeeper
### hal层实现
与TEE侧交互获取KEY、随机数，将明文发送给TEE进行签名

### enroll
version+user_id+flags+明文密码+salt（从TEE获取的随机数）等送到TEE生成一个32字节的签名，签名在应用层保存。

### verity
上层将enroll时生成的签名+尝试的明文密码发送到hal，采用enroll同样的方式请TEE将尝试的密码生成签名，HAL层判断两个签名是否相同。

### 问题
+ 加密密码存储在哪?
TEE只提供签名算法，存储是在android应用层操作的。
密码签名存储位置：`/data/system/locksettings.db*`，删掉可以清除密码
+ 为啥要向keystore添加AuthToken?
目前没看出有啥用，不通过TEE计算AuthToken的HMAC不影响解锁流程，删除addAuthToken代码也不影响解锁流程


## fingerprint
### 应用层逻辑
+ 注册：Settings.apk通过FingerPrintManager发起enroll
+ 解锁：SystemUI.apk通过KeyGuard发起指纹验证，KeyGuardUpdateMonitor调用FingerPrintManager发起authenticate

### hal层实现
所有工作都在TEE测完成，hal层负责发起交互和状态状态传递

### pre_enroll
向TEE获取挑战码

### enroll
TEE侧采集指纹，同时向上层通知采集的状态。用于引用指纹的id（fid)必须大于0。

### verity
TEE侧进行指纹匹配，如果验证成功通过CB通知上层

### 问题
+ 为啥要向keystore添加AuthToken?
目前没看出有啥用，不通过TEE计算AuthToken的HMAC不影响解锁流程，删除addAuthToken代码也不影响解锁流程
+ keystore addAuthToken失败返回6？
没有权限，确保fingerprintd作为system用户运行(root不行)