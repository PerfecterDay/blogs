## 系统表
system_role 角色表
system_role_menu 角色菜单表
system_menu 菜单表
system_user_role 用户角色表

system_users 用户表， `status` tinyint NOT NULL DEFAULT 1 COMMENT '帐号状态（0正常 1停用）'


bizlog-sdk

## 测试信息
长安链归集地址：
认购：0xD1C02F8d906fCFfdc34c19c531758A7Fd13A5bb2
赎回：0x85e411be787300333926B8C69e61905199b45b40

eth归集地址：
认购：0xF73D131e1B888fcBc729b73cDC36823e1Bb89a81
赎回：0x9aF4Bf7E6bAc7F10FE984EdD386717C62b6745BE




0x2e6ec24a628b06aea25abe1c3b867e03bcc1a5e8 长安链账户地址
0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 eth归集地址 公链


issue   0x2e6ec24a628b06aea25abe1c3b867e03bcc1a5e8 ---> 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266

        0xD1C02F8d906fCFfdc34c19c531758A7Fd13A5bb2 ---> 0xF73D131e1B888fcBc729b73cDC36823e1Bb89a81

redeem  0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 ---> 0x2e6ec24a628b06aea25abe1c3b867e03bcc1a5e8
        0x9aF4Bf7E6bAc7F10FE984EdD386717C62b6745BE ---> 0x85e411be787300333926B8C69e61905199b45b40

yzl123 admin123
xd123  admin123
dq123  admin123
xy123  admin123

### 测试环境机器
shiyukun
Shiyukun5713$

gjsyShare@20210422
10.187.225.201:2883 数据库


### SDK
String abiJson = new String(Files.readAllBytes(
        Paths.get("chain-client-config/abi/contracts/contracts-eth/HKDCAccessManager.abi")
));

// 2. 合约地址
String contractAddress = "0x0165878A594ca255338adfa4d48449f69242Eb8F";

// 3. 创建合约调用器
ContractInETH caller = new ContractInETH(contractAddress, abiJson);


String privateKey = readResourceToString("config/crypto-config/node1/admin/admin1/admin1.key");
String publicKey = readResourceToString("config/crypto-config/node1/admin/admin1/admin1.pem");
User user = new User("public", privateKey.getBytes(), "".getBytes(), publicKey.getBytes(), AuthType.Public.getMsg());