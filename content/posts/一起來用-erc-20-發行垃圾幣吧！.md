---
title: "一起來用 ERC-20 發行垃圾幣吧！"
date: 2022-12-26T17:33:17+08:00
draft: false
url: "/2022/12/26/一起來用-erc-20-發行垃圾幣吧！/"
categories:
  - Block Chain
showToc: true
TocOpen: false
---

只要建立了 ERC-20 標準的合約，就可以自動相容各種 ERC-20 代幣的軟硬體錢包，交易所等，事實上，只需要寫約 100 行的 Code 就能自己發行代幣，也可以很輕易的在這些合約 Code 內留下後門來做壞壞ㄉ事。

## Interface

為了能夠把常見的介面給定義清楚， Solidity 有 interface 的功能。所有的函式必須是 External，即使最後宣告時使用 public。

一個 ERC20 的介面必須有 2 個事件以及 6 個函式

```
interface IERC20 {
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);

    function totalSupply() external view returns (uint256);

    function balanceOf(address account) external view returns (uint256);
    function transfer(address to, uint256 amount) external returns (bool);

    function approve(address spender, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
}
```

### totalSupply

可以回傳代幣的總發行量，通常會使用狀態變數 `_uint256 _totalSupply` 來儲存。

```
function totalSupply() external view returns (uint256){
    return _totalSupply;
}
```

### balanceOf

給定一個帳戶 (adress)，回傳該賬戶擁有的代幣餘額 (uint256)，因此會使用一個 mapping 來儲存帳戶跟餘額 `mapping(address => uint256) _balance;`

```
function balanceOf(address account) external view returns (uint256){
    return _balance[account];
}
```

### transfer

```
function transfer(address to, uint256 amount) external returns (bool);
```

呼叫者(msg.sender)，轉移 amount (uint256) 數量的代幣給 to (address)，轉帳成功回傳 true，轉帳失敗回傳 false。

而轉帳會觸發 `Transfer` 事件，分別會丟 `from`, `to` 以及 `value` 三個值出去。只要有代幣產生就必須觸發，就算轉帳金額是 0 也需要。

```
function transfer(address to, uint256 amount) external returns (bool){
    address from = msg.sender;
    require(_balance[from] >= amount, "No money"); // 餘額不足 (Require 要寫必須達成的狀態，若不達成則會噴 Exception)
    require(to != address(0), "Transfer to address 0"); // 不准轉去 0

    _balance[from] -= amount;
    _balance[to] += amount;
    emit Transfer(from, to, amount);
    return true;
}
```

### approve

有時候，我並不會是本人想要去操作 transfer 的行為，我會希望找我的理專或其他人來代為我進行一些 Transfer 的動作；或是我授權給我家人可以動用我錢包的指定的錢，這個時候就可以使用 approve 來進行設定。 成功回傳 true，反之回傳 false。

呼叫者 (msg.sender) 授權 amount 數量的代幣給第三方帳戶 spender 進行使用。

而授權函數會觸發授權事件 Approve。

```
function approve(address spender, uint256 amount) external returns (bool){
    _allowance[msg.sender][spender] = amount; // 允許 spender 花 msg.sender 的 amount 元
                                              // 通常會直接用 = 而不是 +=
    emit Approval(msg.sender, spender, amount);
    return true;
}
```

### allowance

授權數量查詢函式，可以查詢代幣擁有者 owner 允許第三方帳戶 spender 花多少的代幣數量。

`mapping(address => mapping(address => uint256))`

第一層是一個 owner 的 mapping，每一個 owner 可以對應到一個 mapping，而內部的 mapping 則是 spender 對應到 amount。

意義上，會是一個長得像是下面的 json 結構，其中， A 允許 B 花他的 87 元，允許 C 花他的 65 元； B 允許 A 花他的 99 元等。

```json
{
    "A" : {
        "B" : 87,
        "C" : 65
    },
    "B" :{
        "A" : 99
    }
}
```

```
function allowance(address owner, address spender) external view returns (uint256){
    return _allowance[owner][spender];
}
```

### transferFrom

第三方轉帳函式，呼叫者為第三方帳戶 (msg.sender)，可以從代幣持有者 from 轉移 amount 數量的代幣給接收者 to，成功回傳 true，反之則回傳 false。

```
function transferFrom(address from, address to, uint256 amount) public returns (bool){
    require(to != address(0), "Transfer to address 0"); // 不准轉去 0

    uint256 myAllowance = _allowance[from][msg.sender];
    require(myAllowance >= amount, "Not allow!!"); // 持有者授權給 Sender
    _allowance[from][msg.sender] = myAllowance - amount;
    emit Approval(from, msg.sender, myAllowance - amount); // 因為改變了 _allowance，所以需要觸發 Approval

    require(_balance[from] >= amount, "No Money!!"); // 如果在這邊噴錯，會整筆交易 revert，而不會只有上面執行到一半
    _balance[from] -= amount;
    _balance[to] += amount;
    emit Transfer(from, to, amount);
    return true;
}
```

### Summary

所以到這邊的完整程式碼如下

```
// SPDX-License-Identifier: MIT 
pragma solidity ^0.8.11; 

interface IERC20 {
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);

    function totalSupply() external view returns (uint256);

    function balanceOf(address account) external view returns (uint256);
    function transfer(address to, uint256 amount) external returns (bool);

    function approve(address spender, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
}

contract MeowCoin is IERC20{
    uint256 _totalSupply; // 總共發行多少代幣
    mapping(address => uint256) _balance; // Address 對應到代幣餘額
    mapping(address => mapping(address => uint256)) _allowance;

    constructor(uint256 totalSupply_){
        _totalSupply = totalSupply_;
        _balance[msg.sender] = _totalSupply; // 最初，把所有錢轉給自己
    }

    function totalSupply() external view returns (uint256){
        return _totalSupply;
    }

    function balanceOf(address account) external view returns (uint256){
        return _balance[account];
    }

    function transfer(address to, uint256 amount) external returns (bool){
        address from = msg.sender;
        require(_balance[from] >= amount, "No money"); // 餘額不足 (Require 要寫必須達成的狀態，若不達成則會噴 Exception)
        require(to != address(0), "Transfer to address 0"); // 不准轉去 0

        _balance[from] -= amount;
        _balance[to] += amount;
        emit Transfer(from, to, amount);
        return true;
    }

    function approve(address spender, uint256 amount) external returns (bool){
        _allowance[msg.sender][spender] = amount; // 允許 spender 花 msg.sender 的 amount 元
                                                  // 通常會直接用 = 而不是 +=
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    function allowance(address owner, address spender) external view returns (uint256){
        return _allowance[owner][spender];
    }

    function transferFrom(address from, address to, uint256 amount) public returns (bool){
        uint256 myAllowance = _allowance[from][msg.sender];
        require(myAllowance >= amount, "Not allow!!"); // 持有者授權給 Sender
        _allowance[from][msg.sender] = myAllowance - amount;
        emit Approval(from, msg.sender, myAllowance - amount); // 因為改變了 _allowance，所以需要觸發 Approval

        require(_balance[from] >= amount, "No Money!!"); // 如果在這邊噴錯，會整筆交易 revert，而不會只有上面執行到一半
        _balance[from] -= amount;
        _balance[to] += amount;
        emit Transfer(from, to, amount);
        return true;
    }
}
```

## ERC20 擴充

其實做到上述為止，就已經完成依照 ERC20 定義的代幣發行了，但事實上我們還可以幫它增加一些其他的功能，舉例來說：鑄造(mint) 以及銷毀 (burn)，還有可以幫它填上元資料 (metadata)。

### Metadata

約定成俗的介面長這樣

```
interface IERC20Metadata is IERC20 {
    function name() external view returns (string memory);
    function symbol() external view returns (string memory);
    function decimals() external view returns (uint8);
}
```

name 代表代幣名稱，通常會使用 constructor 時給定

symbol 代表代幣的縮寫，通常會使用三四個字母之類的(如 ETH)，一樣使用 constructor 來給定

decimals 代幣的小數點位置，假設 decimals = 3 則 balance = 1234 會顯示成 1.234，基本上代幣都會把 decimals 設定成 18。

```
uint256 _totalSupply; // 總共發行多少代幣
string _name;
string _symbol;

constructor(uint256 totalSupply_ , string memory name_, string memory symbol_){
    _totalSupply = totalSupply_;
    _balance[msg.sender] = _totalSupply; // 最初，把所有錢轉給自己

    _name = name_; // 設定代幣名字
    _symbol = symbol_; //設定代幣代號
}

function name() public view returns (string memory){
    return _name;
}

function symbol() public view returns (string memory){
    return _symbol;
}

function decimals() public pure returns (uint8){
    return 18;
}

function totalSupply() external view returns (uint256){
    return _totalSupply;
}
```

### 鑄造與銷毀

鑄造(mint)：無中生有的產出代幣，只有合約擁有者或是特殊權限的人能呼叫。無中生有的無可以是 address(0)，需要觸發 Transfer 事件。

銷毀(Burn)：把代幣回歸虛無，把需要銷毀的錢轉去 address(0)。誰可以銷毀代幣？合約擁有者或是特定人，看程式怎麼寫都可以，一樣需要觸發 Transfer 事件。

以下範例限制鑄造跟銷毀僅能使用建立合約的 owner 能使用。

```
modifier onlyOwner(){
    require(_owner == msg.sender, "Only Owner can call this function");
    _;
}

function mint(address account, uint256 amount) public onlyOwner{
    require(account != address(0),"mint to address 0 error"); 
    _totalSupply += amount;
    _balance[account] += amount;
    emit Transfer(address(0), account, amount);
}

function burn(address account, uint256 amount) public onlyOwner{
    require(account != address(0),"burn to address 0 error");
    _totalSupply -= amount;
    require(_balance[account] >= amount, "No money");
    _balance[account] -= amount;
    emit Transfer(account, address(0), amount);
}
```

到目前為止，完整的合約程式碼為

```
// SPDX-License-Identifier: MIT 
pragma solidity ^0.8.11; 

interface IERC20 {
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);

    function totalSupply() external view returns (uint256);

    function balanceOf(address account) external view returns (uint256);
    function transfer(address to, uint256 amount) external returns (bool);

    function approve(address spender, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
}

contract MeowCoin is IERC20{
    uint256 _totalSupply; // 總共發行多少代幣
    mapping(address => uint256) _balance; // Address 對應到代幣餘額
    mapping(address => mapping(address => uint256)) _allowance;

    string _name;
    string _symbol;
    address _owner;

    constructor(uint256 totalSupply_ , string memory name_, string memory symbol_){
        _totalSupply = totalSupply_;
        _balance[msg.sender] = _totalSupply; // 最初，把所有錢轉給自己

        _name = name_; // 設定代幣名字
        _symbol = symbol_; //設定代幣代號
        _owner = msg.sender;
    }

    function name() public view returns (string memory){
        return _name;
    }

    function symbol() public view returns (string memory){
        return _symbol;
    }

    function decimals() public pure returns (uint8){
        return 18;
    }

    function totalSupply() external view returns (uint256){
        return _totalSupply;
    }

    function balanceOf(address account) external view returns (uint256){
        return _balance[account];
    }

    function transfer(address to, uint256 amount) external returns (bool){
        address from = msg.sender;
        require(_balance[from] >= amount, "No money"); // 餘額不足 (Require 要寫必須達成的狀態，若不達成則會噴 Exception)
        require(to != address(0), "Transfer to address 0"); // 不准轉去 0

        _balance[from] -= amount;
        _balance[to] += amount;
        emit Transfer(from, to, amount);
        return true;
    }

    function approve(address spender, uint256 amount) external returns (bool){
        _allowance[msg.sender][spender] = amount; // 允許 spender 花 msg.sender 的 amount 元
                                                  // 通常會直接用 = 而不是 +=
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    function allowance(address owner, address spender) external view returns (uint256){
        return _allowance[owner][spender];
    }

    function transferFrom(address from, address to, uint256 amount) public returns (bool){
        uint256 myAllowance = _allowance[from][msg.sender];
        require(myAllowance >= amount, "Not allow!!"); // 持有者授權給 Sender
        _allowance[from][msg.sender] = myAllowance - amount;
        emit Approval(from, msg.sender, myAllowance - amount); // 因為改變了 _allowance，所以需要觸發 Approval

        require(_balance[from] >= amount, "No Money!!"); // 如果在這邊噴錯，會整筆交易 revert，而不會只有上面執行到一半
        _balance[from] -= amount;
        _balance[to] += amount;
        emit Transfer(from, to, amount);
        return true;
    }

    modifier onlyOwner(){
        require(_owner == msg.sender, "Only Owner can call this function");
        _;
    }

    function mint(address account, uint256 amount) public onlyOwner{
        require(account != address(0),"mint to address 0 error"); 
        _totalSupply += amount;
        _balance[account] += amount;
        emit Transfer(address(0), account, amount);
    }

    function burn(address account, uint256 amount) public onlyOwner{
        require(account != address(0),"burn to address 0 error");
        _totalSupply -= amount;
        require(_balance[account] >= amount, "No money");
        _balance[account] -= amount;
        emit Transfer(account, address(0), amount);
    }
}
```

## ERC-20 資安議題

擁有鑄造跟銷毀其實就是一件很危險的事情，如果購買的代幣擁有者有權而且可以隨意操控的話，可以理解成這個合約裡面埋了一個後門， owner 或特定人可以把任意的錢給轉走，也可以發錢給任意的人，這個代幣發行者如果是壞人的話，也可以輕易製造通貨膨脹，或是把你身上的錢給拔掉。

只要符合 IERC20 的 Interface 就可以發行代幣，然而，除了上述的鑄造銷毀之外，因為所有的轉帳等交易行為都是由發行者自己撰寫的程式碼，因此如果代幣發行者心懷不軌，也可以在如 transferFrom 中留有後門，舉例來說，特定使用者有權無視 allowance 強制轉帳等。 也可以直接新增其他管理員專用 Function 來做各種壞壞的事。 因此在玩任何的虛擬貨幣，特別是小眾的 ICO 等內容時，必須認真地讀完其合約，魔鬼藏在 ByteCode 中！！
