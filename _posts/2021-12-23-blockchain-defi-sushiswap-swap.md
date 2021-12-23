---
layout: post
title:  "sushiswap 분석해보기 (스왑 편)"
description: "이번에 포스팅 내용은 blockchain DeFi 중에서 다양한 서비스가 있지만, ethereum Network를 기반으로 하는 분산형 거래소(Decentralized Exchange) 스시스왑(sushiswap)의 주요기능을 알아보겠습니다. 스시스왑은 A Token과 B Token 있을때 A Token으로 B Token으로 교환해주는 기능을 제공합니다. 이 기능은 화폐에서 환전 기능을 동일하다고 볼 수 있습니다. 그리고 Token을 스시스왑 예치 풀에 입금하게 되면 네트워크 블럭생성마다 이자를 발생하게 됩니다. 이것은 화폐에서 예금과 동일할 수 있십낟. 이렇게 주요 두 기능을 스왑, 스테이킹이라는 주제로 solidity의 contract를 분석해보며, truffle을 사용하여 컨트렉트 배포하여 알아보도록 하겠습니다."
date:   2021-12-23
writer: "김준우"
categories: BlockChain
---
## 소개

이번에 포스팅 내용은 blockchain DeFi 중에서 다양한 서비스가 있지만, ethereum Network를 기반으로 하는 분산형 거래소(Decentralized Exchange) 스시스왑(sushiswap)의 주요기능을 알아보겠습니다. 스시스왑은 A Token과 B Token 있을때 A Token으로 B Token으로 교환해주는 기능을 제공합니다. 이 기능은 화폐에서 환전 기능을 동일하다고 볼 수 있습니다. 그리고 Token을 스시스왑 예치 풀에 입금하게 되면 네트워크 블럭생성마다 이자를 발생하게 됩니다. 이것은 화폐에서 예금과 동일할 수 있십낟. 이렇게 주요 두 기능을 스왑, 스테이킹이라는 주제로 solidity의 contract를 분석해보며, truffle을 사용하여 컨트렉트 배포하여 알아보도록 하겠습니다.

## 컨트렉트 분석

아래 그림을 보면 스시스왑의 컨트렉트 프로젝트는 유니스왑v2 컨트렉트를 포함하고 있습니다. 스시스왑에서도 토큰을 스왑을 제공할 수 있는 이유입니다. 이번에 스왑은 유니스왑v2의 컨트렉트를 주로 이용하게 됩니다.



### 주요 컨트렉트 구성
![Untitled](/images/2021-12-23-blockchain-defi-sushiswap-swap/Untitled.png)

### IUniswapV2Factory 컨트렉트

IUniswapV2Factory 주요 역할을 두개의 토큰으로 새로운 페어 토큰을 생성해줍니다. 페어 토큰 또한 ERC20 규격의 토큰입니다. 

```solidity
pragma solidity >=0.5.0;

interface IUniswapV2Factory {
    event PairCreated(address indexed token0, address indexed token1, address pair, uint);

    function feeTo() external view returns (address);
    function feeToSetter() external view returns (address);
    function migrator() external view returns (address);

    function getPair(address tokenA, address tokenB) external view returns (address pair);
    function allPairs(uint) external view returns (address pair);
    function allPairsLength() external view returns (uint);

    function createPair(address tokenA, address tokenB) external returns (address pair);

    function setFeeTo(address) external;
    function setFeeToSetter(address) external;
    function setMigrator(address) external;
}
```

### IUniswapV2Pair 컨트랙트

Factory에서 두개의 토큰으로 IUniswapV2Pair 토큰을 생성해줍니다. UniswapV2Pair의 역할을 ERC20의 동일한 인터페이스를 가지고 있으며, 추가로 Swap, token0, token1, KLast 등 토큰정보 및 스왑과 같은 함수를 추가로 가지고 있습니다. 

```solidity
pragma solidity >=0.5.0;

interface IUniswapV2Pair {
    event Approval(address indexed owner, address indexed spender, uint value);
    event Transfer(address indexed from, address indexed to, uint value);

    function name() external pure returns (string memory);
    function symbol() external pure returns (string memory);
    function decimals() external pure returns (uint8);
    function totalSupply() external view returns (uint);
    function balanceOf(address owner) external view returns (uint);
    function allowance(address owner, address spender) external view returns (uint);

    function approve(address spender, uint value) external returns (bool);
    function transfer(address to, uint value) external returns (bool);
    function transferFrom(address from, address to, uint value) external returns (bool);

    function DOMAIN_SEPARATOR() external view returns (bytes32);
    function PERMIT_TYPEHASH() external pure returns (bytes32);
    function nonces(address owner) external view returns (uint);

    function permit(address owner, address spender, uint value, uint deadline, uint8 v, bytes32 r, bytes32 s) external;

    event Mint(address indexed sender, uint amount0, uint amount1);
    event Burn(address indexed sender, uint amount0, uint amount1, address indexed to);
    event Swap(
        address indexed sender,
        uint amount0In,
        uint amount1In,
        uint amount0Out,
        uint amount1Out,
        address indexed to
    );
    event Sync(uint112 reserve0, uint112 reserve1);

    function MINIMUM_LIQUIDITY() external pure returns (uint);
    function factory() external view returns (address);
    function token0() external view returns (address);
    function token1() external view returns (address);
    function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);
    function price0CumulativeLast() external view returns (uint);
    function price1CumulativeLast() external view returns (uint);
    function kLast() external view returns (uint);

    function mint(address to) external returns (uint liquidity);
    function burn(address to) external returns (uint amount0, uint amount1);
    function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data) external;
    function skim(address to) external;
    function sync() external;

    function initialize(address, address) external;
}
```

### IUniswapV2Router 컨트렉트

IUniswapV2Router의 주요 역할은 사용자의 호출의 창구 역할을 하며, 페어 토큰의 유동성 비율의 편의기능을 제공합니다. 현재 IUniswapV2Router01, IUniswapV2Router02 컨트렉트가 제공되고 있으며, IUniswapV2Router02 에서는 수수료 지원 토큰에 대한 인터페이스가 추가 되었습니다. 이번에는 IUniswapV2Router01 위주의 기능만 사용하겠습니다.

UniswapV2Router 컨트렉트가 유니스왑에선 제일 많이 호출됩니다.
- 유동성을 추가/제거: addLiqudity, removeLiquidity 이며 특수하게 베이스토큰용 함수가 따로 있습니다.
- 토큰 스왑 함수: swapExactTokensForTokens, ... 유사 이름 함수들을 통해 스왑을 할 수 있습니다. 차이점이라면 토큰 두개를 어디에 정확하게 맞출지와 토큰일지 eth 인지에 따라 호출이하는 함수가 달라집니다.

```solidity
pragma solidity >=0.6.2;

interface IUniswapV2Router01 {
    function factory() external pure returns (address);
    function WETH() external pure returns (address);

    function addLiquidity(
        address tokenA,
        address tokenB,
        uint amountADesired,
        uint amountBDesired,
        uint amountAMin,
        uint amountBMin,
        address to,
        uint deadline
    ) external returns (uint amountA, uint amountB, uint liquidity);
    function addLiquidityETH(
        address token,
        uint amountTokenDesired,
        uint amountTokenMin,
        uint amountETHMin,
        address to,
        uint deadline
    ) external payable returns (uint amountToken, uint amountETH, uint liquidity);
    function removeLiquidity(
        address tokenA,
        address tokenB,
        uint liquidity,
        uint amountAMin,
        uint amountBMin,
        address to,
        uint deadline
    ) external returns (uint amountA, uint amountB);
    function removeLiquidityETH(
        address token,
        uint liquidity,
        uint amountTokenMin,
        uint amountETHMin,
        address to,
        uint deadline
    ) external returns (uint amountToken, uint amountETH);
    function removeLiquidityWithPermit(
        address tokenA,
        address tokenB,
        uint liquidity,
        uint amountAMin,
        uint amountBMin,
        address to,
        uint deadline,
        bool approveMax, uint8 v, bytes32 r, bytes32 s
    ) external returns (uint amountA, uint amountB);
    function removeLiquidityETHWithPermit(
        address token,
        uint liquidity,
        uint amountTokenMin,
        uint amountETHMin,
        address to,
        uint deadline,
        bool approveMax, uint8 v, bytes32 r, bytes32 s
    ) external returns (uint amountToken, uint amountETH);
    function swapExactTokensForTokens(
        uint amountIn,
        uint amountOutMin,
        address[] calldata path,
        address to,
        uint deadline
    ) external returns (uint[] memory amounts);
    function swapTokensForExactTokens(
        uint amountOut,
        uint amountInMax,
        address[] calldata path,
        address to,
        uint deadline
    ) external returns (uint[] memory amounts);
    function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline)
        external
        payable
        returns (uint[] memory amounts);
    function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline)
        external
        returns (uint[] memory amounts);
    function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline)
        external
        returns (uint[] memory amounts);
    function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline)
        external
        payable
        returns (uint[] memory amounts);

    function quote(uint amountA, uint reserveA, uint reserveB) external pure returns (uint amountB);
    function getAmountOut(uint amountIn, uint reserveIn, uint reserveOut) external pure returns (uint amountOut);
    function getAmountIn(uint amountOut, uint reserveIn, uint reserveOut) external pure returns (uint amountIn);
    function getAmountsOut(uint amountIn, address[] calldata path) external view returns (uint[] memory amounts);
    function getAmountsIn(uint amountOut, address[] calldata path) external view returns (uint[] memory amounts);
}
```

### IUniswapV2Router의 Liquidity (유동성)이란?

아래 이미지는 유니스왑 공식문서에서 Providing Liquidity 설명입니다. 요약 설명한다면 Token0, Token1 각각 일정 비율로 저장된 Pool이 있을때 token1로 token0을 스왑하려고할때 pool에 예치된 token0, token1 비율에 따라 교환되는 수량이 달라지며, pool은 token1은 추가되고, token0은 줄어 들게 됩니다. 그러면 token0 의 비율이 낮아졌기 때문에 가치는 높아 지게됩니다.  

![Untitled](/images/2021-12-23-blockchain-defi-sushiswap-swap/Untitled%201.png)

### Uniswap 작동원리

유니스왑은 풀이라는 공간이 있고, 그 풀에는 준비된 토큰쌍을 각각 가지고 있습니다. 사용자들은 풀에  토큰쌍을 예치하게되면 증표로 LP 토큰을 받을 수 있습니다. 받은 LP 토큰을 다양하게 활용할 수 있으며, 풀에 예치된 토큰은 스왑을 할때 사용되게 됩니다. 풀의 가치는 k = x * y 공식처럼 가치 = token0 * token1 토큰이 변동해도 가치는 동일하게 유지됩니다.

![Untitled](/images/2021-12-23-blockchain-defi-sushiswap-swap/Untitled%202.png)

### Truffle 구성

truffle 기본 프로젝트에서 contracts 하위에 uniswapv2 컨트렉트를 추가합니다. 그리고 sushiswap의 컨트렉트 contracts 하위에 추가해줍니다. 

![Untitled](/images/2021-12-23-blockchain-defi-sushiswap-swap/Untitled%203.png)

**네트워크를 bsc network 사용하기 (option)**

truffle-config.js에서 networks를 bscTestnet을 추가하게 되면 해당 네트워크에 배포를 할 수 있습니다.

![Untitled](/images/2021-12-23-blockchain-defi-sushiswap-swap/Untitled%204.png)

### UniswapV2 컨트렉트 배포 하기.

**UniswapV2Factory 배포하기**

배포할때 유니스왑에서는 우선적으로 UniswapFactory를 배포해야합니다. UniswapV2Factory를 배포하게 되면 내부적으로 INIT HASH값을 생성하게 됩니다. 최초 생성된 hash값은 페어를 만들때 사용되며, 토큰으로 페어의 주소를 만들때 seed 값으로 식별되는 아주 중요한 값입니다. 

![Untitled](/images/2021-12-23-blockchain-defi-sushiswap-swap/Untitled%205.png)

**pairFor 함수의 hash 변경하기**

/contracts/uniswapv2/library/UniswapV2Library.sol 파일의 24라인 해시코드값을 변경합니다. 

![Untitled](/images/2021-12-23-blockchain-defi-sushiswap-swap/Untitled%206.png)

**UniswapV2Router 외 다나와 토큰 배포하기**

UniswapV2Router에는 해당 네트워크의 베이스 토큰인 WETH가 필요합니다.  UniswapV2Router 컨트렉트를 배포합니다. 테스트시 활용 목적으로 다나와 토큰(ERC20) DNA 토큰을 생성하였습니다. 

![Untitled](/images/2021-12-23-blockchain-defi-sushiswap-swap/Untitled%207.png)

**토큰 사용 승인하기**

유동성을 추가 하기위해서 UniswapV2Router 컨트렉트 주소가 토큰 쌍을 사용 승인을 해야합니다. 유동성을 WBNB, DNA 토큰의 유동성 풀을 만들어보겠습니다 . 그리고 uniswapV2Router의 addLiquidity를 통해 유동성을 추가합니다.

![Untitled](/images/2021-12-23-blockchain-defi-sushiswap-swap/Untitled%208.png)

**유동성 추가 파라미터**

파라미터의 rateA, rateB의 갯수만큼 uniswapV2Router로 전송됩니다.

```solidity
// 이미 배포된 팩토리 주소.
const factoryAddress = "0xe2Ade9dB6287520347c109B29172f7d56DF03e00";  
// 블럭 당 리워드량
const rewardForBlock = web3.utils.toWei('100');                       
// 시작 블럭
const startBlockNumber = 10750000 ;                                   
// 보너스 마감 블록 (보너시 블록 기간 동안 10배 추가 보상)
const bonusEndBlock = 10850000;                                       
// DNA 토큰 총 발행량 (owner 전달)
const dnaTokenSupply = web3.utils.toWei('50000');                     
// 유동성 비율 WBNB
const rateA = web3.utils.toWei('1');                                  
// 유동성 비율 DNA
const rateB = web3.utils.toWei('1000');                               
```

**WBNB → DNA 스왑 해보기**

지금까지 유니스왑 컨트렉트를 배포 하고, 유동성 풀을 구성해보았습니다. 생성한 풀을 이용하여 스왑을 진행해보겠습니다. 

교환할땐 시작 위치의 토큰은 approve를 통해 라우터한테 사용승인을 해주어야합니다. 그리고 라우터의 getAmounts 함수를 통해 비율을 조회합니다. 조회된 비율로 라우터로 다시 스왑함수 swapExactETHForTokens를 호출합니다. 함수 이름에서도 알 수 있듯이 Exact는 위치에 따라 정확한 수치가 지정되고, WBNB 경우 ETH로 바로 호출할 수 있습니다.

![Untitled](/images/2021-12-23-blockchain-defi-sushiswap-swap/Untitled%209.png)

**DNA → WBNB 스왑 해보기**

이번엔 반대로 스왑을 해보겠습니다. 순서는 동일하며, 호출하는 함수와 파라미터의 차이만 있습니다. 가격 비율을 조회할때 함수 이름이 getAmountsIn, getAmountsOut으로 차이가 있습니다. 나중에 UI에서 from 수량을 정확하게 받기위해서는 getAmountIn을 호출하고, to 수량을 정확하게 getAmountOut을 호출하면될거 같습니다.

![Untitled](/images/2021-12-23-blockchain-defi-sushiswap-swap/Untitled%2010.png)

## 정리

아무런 연관 없는 토큰 두개를 유사한 가치로 교환한다는 방식에서 이해하기 어려운 부분이였으나, 테스트코드로 정리해보며 스왑을 알아보았습니다. 이렇게 스왑편은 마무리를 하고, addLiquidity 수행 후 받은 LP토큰을 가지고 스시스왑의 마스터쉐프 컨트렉트 스테이킹을 알아보도록 하겠습니다.