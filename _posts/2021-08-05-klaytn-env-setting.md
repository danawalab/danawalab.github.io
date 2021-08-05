---
layout: post
title: "klaytn 개발 환경 구축"
description: klaytn 개발 환경 구축하는 방법입니다.
date:   2021.08.05. 
writer: "선지호"
categories: klaytn
---

## 소개 : Klaytn 이란?
카카오의 자회사 그라운드X에서 만든 Klaytn은 엔터프라이즈급 안정성을 목표로 고도로 최적화된, BFT 알고리즘 기반 퍼블릭 블록체인입니다. 

## 개발 환경 구축

저는 개발 환경을 도커로 구축하였습니다.
사용한 웹 IDE는 code-server이고, 도커 이미지를 통해 간편하게 구축을 진행했습니다.

먼저, 도커파일을 작성합니다.
사용한 이미지는 우분투와 nodejs 10버전의 이미지입니다.

```jsx
FROM ubuntu:18.04
FROM node:10

RUN apt update -y
RUN apt upgrade -y
RUN apt install -y wget sudo vim build-essential 

WORKDIR /app

RUN mkdir /app/.npm-global 
RUN chmod 777 /app/.npm-global
RUN npm config set prefix '/app/.npm-global' 
ENV PATH /app/.npm-global/bin:$PATH

WORKDIR /root

# truffle 설치 필수
RUN npm install -g --unsafe-perm=true --allow-root truffle

# klaytn bapp 샘플
RUN git clone https://github.com/klaytn/countbapp
RUN git clone https://github.com/klaytn/klaystagram

# code-server 설치
WORKDIR /app/.code-server
RUN wget https://github.com/cdr/code-server/releases/download/3.2.0/code-server-3.2.0-linux-x86_64.tar.gz
RUN tar -xvzf code-server-3.2.0-linux-x86_64.tar.gz -C ./ --strip-components 1

EXPOSE 10000
EXPOSE 8888

CMD ["/bin/bash", "-c", "/app/.code-server/code-server --port 10000 --host 0.0.0.0 --auth none"]
```

도커파일을 위와 같이 작성한 후, 아래와 같이 입력하여 docker 컨테이너를 실행시킵니다. 

(8888번 포트는 위의 샘플들의 동작 하는 포트 입니다.)

```jsx
$ docker build -t blockchain-dev-server:local .
$ docker run -d --name blockchain -p 10000:10000 -p 8888:8888 blockchain-dev-server:local
```

## 설명

이제 환경 구축이 되었으니, 샘플 프로젝트를 실행해 보겠습니다.

제일 먼저, klaytn 계정을 하나 만들어주세요. 그리고 facet에 가서 토큰을 발급받으세요.

(관련 url: [계정 생성 URL](https://baobab.wallet.klaytn.com/, "계정 생성") )

이 후, countbapp으로 이동합니다. 그리고 터미널을 열어 아래 내용을 입력해줍니다.

아래 내용은 truffle을 이용하여 klaytn 테스트넷인 baobab 네트워크에 contracts 폴더 안에 솔리디티 언어로 작성한 컨트랙트를 배포하는 내용입니다.

```jsx
$ cd /root/countbapp
$ truffle deploy --reset --network baobab
```

그리고 위의 커맨드가 정상적으로 끝났다면, 아래 내용을 입력하여 프로그램을 실행해 줍시다.

```jsx
$ npm i 
$ npm run local
```

그리고 웹 브라우저를 열어 localhost:8888을 입력하여 정상적으로 작동하는지 확인해봅니다.

![/images/2021-08-05-klaytn-env-setting/1.png](/images/2021-08-05-klaytn-env-setting/1.png)

## 응용

저는 위의 countbapp을 수정해서 사칙 연산 또한 가능할 수 있도록 수정해 보았습니다.

먼저, contracts 폴더에 있는 Count.sol 파일을 아래 내용으로 바꾸어주었습니다.

모두 klaytn Docs에 있는 내용이니 Docs를 한번 자세히 보시면 충분히 따라하실수 있습니다.

```jsx
pragma solidity ^0.5.6;

library SafeMath {
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        uint256 c = a + b;
        require(c >= a, "SafeMath: addition overflow");

        return c;
    }

    function sub(uint256 a, uint256 b) internal pure returns (uint256) {
        require(b <= a, "SafeMath: subtraction overflow");
        uint256 c = a - b;

        return c;
    }

    function mul(uint256 a, uint256 b) internal pure returns (uint256) {
        if (a == 0) {
            return 0;
        }

        uint256 c = a * b;
        require(c / a == b, "SafeMath: multiplication overflow");

        return c;
    }

    function div(uint256 a, uint256 b) internal pure returns (uint256) {
        require(b > 0, "SafeMath: division by zero");
        uint256 c = a / b;
        return c;
    }

    function mod(uint256 a, uint256 b) internal pure returns (uint256) {
        require(b != 0, "SafeMath: modulo by zero");
        return a % b;
    }
}

contract Count {
    
    using SafeMath for uint256;
    
    uint256 public count = 0;
    
    address public lastParticipant;

    function getBlockNumber() public view returns (uint256) {
      return block.number;
    }
    
    function plus() public {
        count = count.add(1);
        lastParticipant = msg.sender;
    }
    
    function minus() public {
        count = count.sub(1);
        lastParticipant = msg.sender;
    }
    
    function multiply() public {
        count = count.mul(2);
        lastParticipant = msg.sender;
    }
    
    function divide() public {
        count = count.div(2);
        lastParticipant = msg.sender;
    }

    function setCount(uint256 _count) public {
      count = _count;
    }
}
```
![/images/2021-08-05-klaytn-env-setting/2.png](/images/2021-08-05-klaytn-env-setting/2.png)
이후, 저 내용을 바탕으로 src/components/Count.js 를 수정해줍니다.
먼저 두 함수, setMul과 setDiv를 추가해줍니다.

```jsx
setMul = () => {
    const walletInstance = caver.klay.accounts.wallet && caver.klay.accounts.wallet[0]

    // Need to integrate wallet for calling contract method.
    if (!walletInstance) return

    this.setState({ settingDirection: 'multiple' })

    this.countContract.methods.multiply().send({
      from: walletInstance.address,
      gas: '200000',
    })
      .once('transactionHash', (txHash) => {
        console.log(`
          Sending a transaction... (Call contract's function 'multiply')
          txHash: ${txHash}
          `
        )
      })
      .once('receipt', (receipt) => {
        console.log(`
          Received receipt! It means your transaction(calling multiply function)
          is in klaytn block(#${receipt.blockNumber})
        `, receipt)
        this.setState({
          settingDirection: null,
          txHash: receipt.transactionHash,
        })
      })
      .once('error', (error) => {
        alert(error.message)
        this.setState({ settingDirection: null })
      })
  }
  setDiv = () => {
    const walletInstance = caver.klay.accounts.wallet && caver.klay.accounts.wallet[0]

    // Need to integrate wallet for calling contract method.
    if (!walletInstance) return

    this.setState({ settingDirection: 'divide' })

    this.countContract.methods.divide().send({
      from: walletInstance.address,
      gas: '200000',
    })
      .once('transactionHash', (txHash) => {
        console.log(`
          Sending a transaction... (Call contract's function 'divide')
          txHash: ${txHash}
          `
        )
      })
      .once('receipt', (receipt) => {
        console.log(`
          Received receipt! It means your transaction(calling divide function)
          is in klaytn block(#${receipt.blockNumber})
        `, receipt)
        this.setState({
          settingDirection: null,
          txHash: receipt.transactionHash,
        })
      })
      .once('error', (error) => {
        alert(error.message)
        this.setState({ settingDirection: null })
      })
  }
```

이후, render() 부분에 있는 setMinus 버튼 밑에 아래 내용을 넣습니다.

```jsx
        <button
          onClick={this.setMul}
          className={cx('Count__button', {
            'Count__button--setting': settingDirection === 'multiple',
          })}
          disabled={count == 0}
        >
          *
        </button>
        <button
          onClick={this.setDiv}
          className={cx('Count__button', {
            'Count__button--setting': settingDirection === 'divide',
          })}
          disabled={count == 0}
        >
          /
        </button>
```


![/images/2021-08-05-klaytn-env-setting/3.png](/images/2021-08-05-klaytn-env-setting/3.png)

그리고 다시 localhost:8888 접속하면 아래와 같은 모습이 보입니다!

![/images/2021-08-05-klaytn-env-setting/4.png](/images/2021-08-05-klaytn-env-setting/4.png)
![/images/2021-08-05-klaytn-env-setting/5.png](/images/2021-08-05-klaytn-env-setting/5.png)

## 마무리

오늘은 간단히 klaytn bapp 개발환경 구축 및 개발 방법에 대해 알아보았습니다.

거의 대부분의 내용은 klaytn docs에 있지만, 추가로 개발을 어떻게 해야 하는지, 환경은 어떻게 구축해야 하는지 잘 모르시는 분들께 도움이 되었으면 합니다.

감사합니다.

## 참고 자료
- https://ko.docs.klaytn.com/klaytn