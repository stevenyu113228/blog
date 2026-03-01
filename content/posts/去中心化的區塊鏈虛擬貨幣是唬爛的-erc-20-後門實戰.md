---
title: "去中心化的區塊鏈虛擬貨幣是唬爛的 - ERC-20 後門實戰"
date: 2023-01-05T14:08:15+08:00
draft: false
url: "/2023/01/05/去中心化的區塊鏈虛擬貨幣是唬爛的-erc-20-後門實戰/"
categories:
  - 網頁安全
  - Block Chain
showToc: true
TocOpen: false
---

前一篇文中，我們使用了 Solidity 實作 ERC 20 代幣，當時我們提到了區塊鏈的虛擬貨幣，其實就只是在 EVM 中的一個 map (類似 Python 的 Dictionary)，裡面儲存了位址以及金額。 而 ERC 20 標準，只需要符合指定的 Interface，講白話文就是，僅需要 Implement 指定的幾個 Function，就能發行一個新的虛擬貨幣。

然而，這會有兩個明顯的問題，首先，我們只需要依照標準定義 function 的名稱、輸入、輸出就可以符合 ERC 20 的標準，這邊並沒有限制任何的 function 內容需要怎麼實作，這意味著我們可以自己幫 Function 加料，例如在 Function 中埋藏一些後門。 另外，我們也可以在同一個 Contract 中，額外實作標準以外的其他 Function 來達成我們希望的功能，做這兩件事情，都依然是一個合法、符合 ERC 20 標準的代幣。

為了避免重造輪子，事實上我們可以直接 import 由 OpenZeppelin 提供的 Library 來進行繼承或是修改，OpenZeppelin 也提供了很多擴充功能可以使用。

```
// SPDX-License-Identifier: MIT 
pragma solidity ^0.8.17;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
```

舉例來說，我們想要發行一個後門幣 (Back Door Coin, BDC) 的話，僅需在合約的 Constructor 定義我們初始需要鑄造多少顆幣，預設的 decimals 是 18，也就是真正的貨幣金額會是 Balance 值 *10^-18，以避免浮點誤差。

```
constructor() ERC20("Back Door Coin", "BDC"){
    _mint(msg.sender, 10 * 10**decimals());
}
```

其實到這邊為止，我們就已經做好一個最基本的幣了，而本篇文章我們將透過上述的兩種手法，使後門幣可以重新辦到中心化的效果。

```
function transferFrom(address from, address to, uint256 amount) public override returns (bool){
    if(msg.sender == owner()){
        _transfer(from, to, amount);
    }else{
        super.transferFrom(from, to, amount);
    }
    return true;
}
```

首先，我們可以直接 override 原本 OpenZeppelin 提供的 transferFrom，原始的 transferFrom 程式碼功能是，若 A 允許 B 操縱其 (A) 的指定金額，可以先使用 approve 來定義清楚金額， approve 後，B 就可以任意的操縱 A 身上指定金額的錢錢，例如 B 可以使用 transferFrom 把錢從 A 身上轉移到 C 身上等，這邊的前提是 A 必須要預先 approve 允許 B 進行操作。

然而，我們可以透過上面的程式碼，自行定義一個 Admin 特權，我們可以使用 owner() 取得建立該合約的 admin address，並檢查，若發送此請求的 address 是這個 owner 的 address 時，就直接執行內部的 _transfer 函數，這個函數不會檢查任何 approve，會直接乖乖的依照上述內容把值進行轉帳。

透過這個函數的修改，我們只要是管理員權限，就可以任意的轉走別人身上的錢，而不需要任何的 Approve，也因為不用 Approve。大家應該都知道，無論是冷熱錢包，虛擬貨幣的錢並不是存在錢包裡面的，所謂的錢包只是儲存了 Private Key 而已，因此就算是完全斷電的冷錢包裡面的錢(該私鑰對應到的帳號的錢)也可以直接被轉走！

除此之外，我們也可以直接新增 mint 以及 burn 的函數，任意的燒毀或是鑄造新的貨幣，僅需要是 Owner 的權限即可，帶有底線的 _mint 以及 _burn 都已經在 OpenZeppelin 中幫我們實踐好了！

```
function mint(address adr, uint256 balance) public onlyOwner returns (bool){
    _mint(adr, balance);
    return true;
}

function burn(address adr, uint256 balance) public onlyOwner returns (bool){
    _burn(adr, balance);
    return true;
}
```

透過任意的發錢，我們可以快速的讓一個幣通貨膨脹，而透過 Burn，我們則可以直接把幣給銷毀，無論你原本身家有多少錢，只要 owner 開心，隨時都可以幫你歸 0，畢竟這跟上面的轉帳一樣，都只是操縱合約內部的一個 map 型態變數而已。

而 Owner 開心時，也可以寫一個 transferOwnership 的函數來更換這個 admin 權限的 address，事實上，我們也可以直接在合約內設定多個可變的 admin 帳號，來達到多人管裡的效果。

```
function transferOwnership(address newOwner) public override{
    super.transferOwnership(newOwner);
}
```

除了上述的這些玩法之外，我們也可以透過增加新功能的方法，使整個合約凍結；或是讓特定帳戶凍結；使帳戶裡面的錢只可進、不可出 ……，這些手段從合約角度來看都只是幾行 code 的事情，實作起來非常非常的簡單。

上述的完整合約 Code 如下

```
// SPDX-License-Identifier: MIT 
pragma solidity ^0.8.17;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

contract BDC is ERC20, Ownable{

    constructor() ERC20("Back Door Coin", "BDC"){
        _mint(msg.sender, 10 * 10**decimals());
    }

    function transferFrom(address from, address to, uint256 amount) public override returns (bool){
        if(msg.sender == owner()){
            _transfer(from, to, amount);
        }else{
            super.transferFrom(from, to, amount);
        }
        return true;
    }

    function mint(address adr, uint256 balance) public onlyOwner returns (bool){
        _mint(adr, balance);
        return true;
    }

    function burn(address adr, uint256 balance) public onlyOwner returns (bool){
        _burn(adr, balance);
        return true;
    }

    function transferOwnership(address newOwner) public override{
        super.transferOwnership(newOwner);
    }
}
```

我也把上述的合約部署到 Goerli 測試鏈上做了一個簡易的 PoC：  
Etherscan ERC 20 : [https://goerli.etherscan.io/token/0x2b282197affe3517fbfbd6c2a3465d685493fd24](https://goerli.etherscan.io/token/0x2b282197affe3517fbfbd6c2a3465d685493fd24)

![](/uploads/2023/01/截圖-2023-01-05-下午2.11.49-1024x369.png)

Etherscan Contract : [https://goerli.etherscan.io/address/0x2b282197affe3517fbfbd6c2a3465d685493fd24](https://goerli.etherscan.io/address/0x2b282197affe3517fbfbd6c2a3465d685493fd24)

![](/uploads/2023/01/截圖-2023-01-05-下午2.12.36-1024x403.png)

我們這邊定義了三個角色，以及一個合約  
帳號 A : 0xa3D8D1Ed2EAE476164Fc056077d70FA8C4551191  
帳號 B : 0x6c4631840b8D0320e5d1e4d7498670c9fb402c08  
帳號 C : 0x30377288a005eE70c1D06451622e79b3c1b054F5  
部署上去的合約：0x2B282197AFfe3517FBFBd6C2A3465d685493Fd24

依照時序來看

[Tx 0x5e421186](https://goerli.etherscan.io/tx/0x5e4211866337e583b3d44ec319a2150c4ee31b5670db8390d433f0df846578f6) 我們可以看出，我們成功地透過帳號 A 建立了一個合約，而從 ERC-20 Tokens Transferred 可以看出，我們在建立合約時同時發了 10 顆 BDC 後門幣給 Owner (A)．

[Tx 0x4e4e1633](https://goerli.etherscan.io/tx/0x4e4e16330430dbf405f03f6f2d41a550aecf39357e2d65cda1c7f13671426575) 這個 Transaction 透過 A 進行 Mint 鑄造了 30 顆新的 BDC 給 B 帳號。

[Tx 0xd03409bf](https://goerli.etherscan.io/tx/0xd03409bf6c1893f9ca7b1cfaa2f7f810d718fad790dd47aa68b69fdf5535dba8) 這邊開始就有鬼了，如果從 ERC 20 角度來看，我們把 B 帳號的 10 BDC 轉給了 C 帳號，但從合約角度來看，發起這個 Transaction 的是 A 帳號！而這是該合約的第 3 個 Transaction，照理來說要執行這個 Transfer From 的前提是 B 必須要先 Approve A 做這件事，A 才能進行轉錢的動作。

[Tx 0x6a4061ae](https://goerli.etherscan.io/tx/0x6a4061ae3e5e7a10c2be9695c10ebd40a270c641c9cb89df1ef0e23f4923c85c) and [Tx 0x77827936](https://goerli.etherscan.io/tx/0x778279364b964c26277aa1ee70a637885ac03333b52b706296759087891759fb) 展示了正規的 Approve 以及 Transfer From 執行方式，Demo 了 A 允許 B 轉他身上的 5 BDC ，並使用 B 帳號將其轉給 C，這一切都是正常行為。

[Tx 0xbcaf5c41](https://goerli.etherscan.io/tx/0xbcaf5c41fa463c9b4a69262ad1cb56ca743e3a3a6027b7728a8144c9b3b99726) 執行了 transferOwnership ，把 A 原本是 Owner 的管理員身份轉換給了 B。

[Tx 0xc6c9ce1d](https://goerli.etherscan.io/tx/0xc6c9ce1d2e5a0a54801bea2fa2804075ef2a76d1e2dbb148835fa6633e71f608) 則是燒毀錢錢，因為我們已經把 Owner 給轉換成了 B 帳號，因此使用 B 帳號呼叫合約上的這個 Function，把 C 身上的 15 塊錢給燒掉。當然，這個過程全程都不需要 C 的任何允許，C 就算使用冷錢包，全程離線的前題底下也一樣可以把他的錢燒掉。

透過以上的例子，我們就可以得知，雖然區塊鏈上的東西不能竄改也是去中心化的，理想上無論是政府、銀行或任何高層想要動你的錢都是不行的，以上這些的前提都是作為一個理想狀況會發生的事。 然而，現實中的代幣可沒有想像中這麼簡單，只需要短短的把 Code 隨意多加料個幾行，就可以快速的實作出一個具有後門的代幣，在去中心化的世界中，運用 Owner 的角色重拾中心化的概念。

希望上面的例子可以讓大家重新重視區塊鏈角度的安全，並不是什麼ㄉㄉ老師、立頓大哥之類的騙子隨便講一個區塊鏈落地應用，發個貓貓幣、減肥幣、節能減碳幣、抗中保台幣，就傻傻地買下去。對於非主流 (比特幣、以太幣)，之外的幣，都需要特別的檢視其 ABI 以及合約內容。而相對主流的一些幣，舉例來說 [USDT](https://etherscan.io/address/0xdac17f958d2ee523a2206206994597c13d831ec7#writeContract) 也有 Pause、addBlackList 的功能，而馬斯克最愛的[狗勾幣(DOGE)](https://bscscan.com/token/0xba2ae424d960c26247dd6c32edc70b295c744c43) 從合約角度來看，也有 Mint、Burn 功能，當持有者想要做壞事時，這些幣一點也不安全，而這些操作就是中心化操作！
