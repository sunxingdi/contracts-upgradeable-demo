# 升级合约学习总结

### 1. Call与Delegatecall原理区别

合约B ——call——> 合约C： 执行C的函数，修改C的状态变量。

合约B ——Delegatecall——> 合约C： 执行C的函数，修改B的状态变量。

<br>

### 2. 代理模式
基于Delegatecall的原理，可以构造智能合约的代理模式：

用户 ——call——> 代理合约 ——Delegatecall——> 逻辑合约

在代理合约里的fallback函数里执行Delegatecall调用逻辑合约中的业务函数。

代理模式将合约数据和逻辑分开，分别保存在不同合约中。

数据（状态变量）存储在代理合约中，而逻辑（函数）保存在逻辑合约中。

要求代理合约和逻辑合约中的状态变量存储槽对应，否则存在漏洞。

<br>

### 3. 普通可升级代理（存在漏洞风险）
基于代理模式，可以构造普通可升级的合约，当需要升级合约的逻辑时，只需要将代理合约指向新的逻辑合约即可。

在普通可升级合约中，存在3个实体：代理合约，逻辑合约，管理合约（或管理员地址）。
- 升级函数（修改目标逻辑合约地址的函数）位于代理合约里。
- 管理员可以执行代理合约里的升级函数，也可以执行逻辑合约里的业务函数。
- 用户不可以执行代理合约里的升级函数，只可以执行逻辑合约里的业务函数。

方案描述：
- 代理合约里的升级函数upgrade()：只有管理员能执行。
- 代理合约里的回调函数fallback()：所有人都能执行，回调函数里执行delegatecall。


>**存在函数选择器冲突漏洞风险**：
>如果逻辑合约里的业务函数和代理合约里的升级函数选择器冲突，管理员在执行业务函数时，可能误调用升级函数，升级为黑洞合约。

<br>

### 4. 透明代理模式（方案一，高gas）
透明代理模式可以解决普通可升级合约中的选择器冲突风险。

解决方法：
- 代理合约里的升级函数upgrade()：只有管理员能执行。
- 代理合约里的回调函数fallback()：不能由管理员执行。

>**存在gas费高的问题**：
>回调函数中新增是否为管理员的高频检查，产生高gas费。

<br>

### 5. UUPS通用可升级代理（方案二，推荐）
把代理合约里的升级函数迁移至逻辑合约中，如果存在函数选择器冲突，则编译失败，彻底解决函数选择器冲突问题，但是会让合约更加复杂。

解决方法：
- 代理合约里的回调函数fallback()：所有人可执行。
- 逻辑合约里的升级函数upgrade()：只有管理员能执行。

<br>

### 参考文章

>注：所有代码来源于[WTF学院](www.wtf.academy)，添加调试打印信息，仅用于学习分享。

[WTF Solidity极简入门: 23. Delegatecall](https://www.wtf.academy/solidity-advanced/Delegatecall/)

[WTF Solidity极简入门: 46. 代理合约](https://www.wtf.academy/solidity-application/ProxyContract/)

[WTF Solidity极简入门: 47. 可升级合约](https://www.wtf.academy/solidity-application/Upgrade/)

[WTF Solidity极简入门: 48. 透明代理](https://www.wtf.academy/solidity-application/TransparentProxy/)

[WTF Solidity极简入门: 49. 通用可升级代理](https://www.wtf.academy/solidity-application/UUPS/)

[WTF Solidity源码仓库](https://github.com/AmazingAng/WTF-Solidity)