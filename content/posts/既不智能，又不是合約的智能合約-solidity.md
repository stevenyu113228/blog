---
title: "既不智能，又不是合約的智能合約 - Solidity"
date: 2022-12-26T13:04:27+08:00
draft: false
url: "/2022/12/26/既不智能，又不是合約的智能合約-solidity/"
categories:
  - Block Chain
showToc: true
TocOpen: false
---

智能合約並不智能，也不是合約，它只是可以執行在 EVM (Ethereum Virtual Machine) 中的一段 ByteCode 而已。之所以叫這個名字應該只是比較潮ㄅ，而且我覺得「智能」還有一點支語的味道，但在台灣也很少聽到「智慧型合約」這種說法。

合約不會自動執行，也不會自動結算跟驗證之類的事情，如果需要執行合約內的 Function，需要呼叫指定的 Transaction。

## Account

在 Ethereum 底下，有兩種的 Account，分別是具有 Contract Code 的 Contract Account 以及沒有 Contract 的 EOA (External Owned Account)。 創建 Contract 需要使用到 EOA，且 Contract Account 比起 EOA 增加了 Storage hash 以及 Code hash 的欄位。

在 EOA 底下，總共有三種 Transaction

1. 轉 Ether
2. 部署 Contract
3. 呼叫 Contract 裡面的特定 Function

在進行轉 Ether 的 Transaction 時，總共會有 4 個欄位：From, To, Value 以及 Data，其中，Data 為備註欄位，可以填入任意值。

而透過 EOA 執行部屬 Contract 時，在做的事情跟轉 Ether 一樣，同樣需要填入上述的 4 個欄位。只是 To 需要留空，只要 To 留空就代表該筆 Transaction 是 Create New Contract。 而 Data 欄位則輸入填入 Byte Code， Value 則可以塞一些 Ether 進入合約內。

為了避免裝置差異，所以 Contract 的 OPCode 會放置於 Ethereum Virtual Machine 內執行，所有的合約都會嚴格限制版本以避免各種問題發生。通常 EVM ByteCode 的開頭都是 `0x60806040`，每一條指令都是一個 Bytes。

User 可以透過 ABI (Application Binary Interface) 與 Contract 進行溝通。

## Remix IDE

以太坊提供了瀏覽器版本的 IDE 可以快速部署以及測試合約內容，這套 IDE 被稱為 Remix，透過外觀可以發現它是魔改 VSCode 的產物。

[https://remix.ethereum.org/](https://remix.ethereum.org/)

透過 Remix Ethereum IDE 可以直接連結 Metamask 進行部署，也可以使用運行在瀏覽器上的 Remix VM 鏈進行測試。

## Solidity 基礎

### 架構

通常一個 Contract 的程式碼架構如下

```
// SPDX-License-Identifier: MIT
// 定義 License，如果不寫會跳 Warning

// 定義版本
pragma solidity >0.8.0;

// 合約開頭
contract HelloWorld{

    // 宣告變數
    int x; 

    // 建構子
    constructor(int _y){
      // xxx
    }

    // 函數
    function getx() public view returns (int){
        return x;
    }
}
```

### 資料型態

Solidity 是一個強型別語言，由於浮點誤差的緣故，通常不會使用到浮點數運算，也不提供 float 以及 double，而是全部都使用 int。 常見的解法是使用 decimal 之類的方法，把值同乘以某個數字來代表。舉例來說 `decimals = 8`, 則 `7000000000` 代表的是 `70`。

如果直接使用 `int` 則預設代表 `int256`，變數如果不初始化會預設為 0。

Array 有兩種，分別是靜態以及動態，動態陣列宣告方式為不宣告陣列長度 `[]`，並且可以透過 `.push()` 在後面塞入值 (類似 Python 的 append)，並且可以透過 `.length` 取得當前陣列長度。

Mapping 等於 Python 的 Dict，宣告方式為 `mapping(typeA => typeB)`，舉例來說，`mapping(string => int) public mp` 就可以宣告一個 mp 的 mapping (字典)，其中 key 為 string，而 value 為 int。

### Function

Solidity 的 Function 比起其他語言的 Function，有一點點不一樣的點在於，需要額外定義 Visibility 以及 State Mutability。

Visibility (能見性) 代表這個 Function 可以被怎麼樣的合約呼叫，有一點點像是物件導向程式語言的 Public 以及 Private，但除了這兩個之外，還有一些其他的定義如下。

- Public : 任何合約或帳戶都可以呼叫
- Private : 只有目前的合約可以呼叫
- External : 除了目前合約與其繼承合約之外可以呼叫
- Internal : 只有目前合約可以呼叫

Mutability (可變性)代表了這個 Function 是否會對狀態進行寫入

- Pure : 不會讀取也不會改變區塊鏈內的東西 (不需要花 Gas Fee)
- View : 只會讀取合區塊鏈內的東西 (不需要花 Gas Fee)
- nonPayable : 可以對區塊鏈進行寫入，但不能接受 Ether
- Payable : 可以對區塊鏈進行寫入，且可以額外接受 Ether

Solidity 的 Function 可以接受多個回傳值，而他們的格式長的如下列。

```
function FnName [V] [SM] [returns ()] {}
```

會在函數宣告時直接宣告變數的回傳型態，除了使用傳統 return 之外，也可以直接在 returns 上面定義要回傳的變數名稱。

```
function ret2var() public pure returns (int xx,int yy){
    xx = 9487;
    yy = 9453;
}
```

### msg object

在名為 msg 的這個 Object 中，可以取得一些當前 Transaction 中的資料。

- msg.sender : 發送者的 address
- msg.value : 合約內的以太數量 (以 Wei 為單位)
- msg.data : 完整的調用資料

而在 address 中，我們也可以接使用 `.balance` 來查詢使用者的 ETH 餘額

## Summary

以下我寫了一個簡單的小例子，理論上讀懂上面的所有程式碼，就可以搞懂基礎的 Solidity 語法了！！

```
// SPDX-License-Identifier: MIT
pragma solidity >0.8.0;

// 合約開頭
contract HelloWorld{

    // int 等於 int256
    int x; // 不寫等於初始化成 0
    int y;

    // mapping 等於 python 的 dict, 裡面需要寫變數型態 to 變數型態
    mapping (string => int) public mp;

    // 建構子，創建合約時可以丟進去的東西
    constructor(int _y){
        y = _y;

        mp["meow"] = 87;
        mp["dog"] = 88;
    }

    /*
        function FnName [V] [SM] [returns ()] {}
        V : Visibility
            - Public : 任何合約或帳戶都可以呼叫
            - Private : 只有目前的合約可以呼叫
            - External : 除了目前合約與其繼承合約之外可以呼叫
            - Internal : 只有目前合約可以呼叫

        SM : State Mutability
            - Pure : 不會讀取或寫入任何狀態
            - View : 只會讀取狀態
        returns
    */

    // getx function 可以直接簡化為下面的 uint public x;
    // public 代表外面可以 call
    // view 代表不會修改到內部的東西，執行不需要花 gas fee
    function getx() public view returns (int){
        return x;
    }

    string public meow = "meowmeow"; // 直接給變數 public = getter 方法

    // 會修改到全域變數 x 以及回傳 sum，需要花 gas
    function add(int a, int b) public returns (int){
        int sum = a + b;
        x = sum;
        return sum;
    }

    // 可以不用寫 return，直接在宣告時定義回傳的是哪個變數 
    function mul(int a, int b) public returns (int sum){
        sum = 0;
        for(int i=0; i<b; i++){
            sum += a;
        }
        x = sum;
    }

    // 可以回傳多個值
    function ret2var() public pure returns (int xx,int yy){
        xx = 9487;
        yy = 9453;
    }

    // call function 範例
    function call2func() public pure{
        (int a,int b) = ret2var();
        a += 1;
        b += 2;
    }

    /*
        資料位置有三種
        memory, storage, calldata
    */

    // mapping 範例，總之就是類似 py 的 dict
    function write_dict(string memory name, int val) public{
        mp[name] = val;
    }

    function dict(string memory name) public view returns(int){
        return mp[name];

    }

    // Array
    int [] ar;
    function push_array(int val) public returns(int [] memory, uint256){
        ar.push(val);
        uint256 len = ar.length;
        return (ar, len);
    }

    // 要碰到 msg.value 需要使用 payable
    function msg_info() public payable returns(address, uint256, bytes memory, uint256){
        return (
            msg.sender,  // 發送者訊息
            msg.value,  // 合約內的以太數量 (wei)
            msg.data,   // 完整的調用資料
            gasleft() // 剩下的 gass
        );
    }

    // message sender 的 address
    address public owner = msg.sender; 
    function adr() public view returns(uint balance){
        // 用戶餘額 Ether
        balance = owner.balance;
    }

}
```
