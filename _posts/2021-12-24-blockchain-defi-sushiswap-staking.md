---
layout: post
title:  "sushiswap 분석해보기 (스테이킹 편)"
description: "이번 포스팅은 스시스왑의 스테이킹 기능에 알아보도록 하겠습니다. 스테이킹이란 화폐를 은행에 예금하게 되면 일정기간마다 이자가 발생하여 수익을 얻을 수 있습니다.  스시스왑에서도 동일하게 마스터쉐프 컨트렉트를 통해 토큰을 스테이킹하면 이자 토큰이 증가하여 출금때 원금과 함께 이자를 받을 수 있습니다. 앞서 스테이킹을 진행하기위해서는 “sushiswap 분석해보기 (스왑편)”을 내용을 읽어보시면 도움이 됩니다."
date:   2021-12-24
writer: "김준우"
categories: BlockChain
---
## 소개

이번 포스팅은 스시스왑의 스테이킹 기능에 알아보도록 하겠습니다. 스테이킹이란 화폐를 은행에 예금하게 되면 일정기간마다 이자가 발생하여 수익을 얻을 수 있습니다.  스시스왑에서도 동일하게 마스터쉐프 컨트렉트를 통해 토큰을 스테이킹하면 이자 토큰이 증가하여 출금때 원금과 함께 이자를 받을 수 있습니다. 앞서 스테이킹을 진행하기위해서는 “[sushiswap 분석해보기 (스왑편)](https://danawalab.github.io/blockchain/2021/12/23/blockchain-defi-sushiswap-swap.html)”을 내용을 읽어보시면 이해에 도움이 됩니다. 


## 스테이킹 컨트렉트 분석

**MasterChef 컨트렉트**

스시스왑에서 핵심 컨트렉트입니다. 마스터쉐프의 역할은 스테이킹 풀의 목록을 가지고 있으며, 예치한 사용자 목록을 가지고 있습니다. 그리고 이차계산을 하는 함수를 포함하고 있습니다.  마스터 쉐프는 코드상에 추가로 주석으로 설명하겠습니다.

```solidity
pragma solidity 0.6.12;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/SafeERC20.sol";
import "@openzeppelin/contracts/utils/EnumerableSet.sol";
import "@openzeppelin/contracts/math/SafeMath.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "./SushiToken.sol";

// MasterChef is the master of Sushi. He can make Sushi and he is a fair guy.
//
// Note that it's ownable and the owner wields tremendous power. The ownership
// will be transferred to a governance smart contract once SUSHI is sufficiently
// distributed and the community can show to govern itself.
//
// Have fun reading it. Hopefully it's bug-free. God bless.
contract MasterChef is Ownable {
    using SafeMath for uint256;
    using SafeERC20 for IERC20;
		// 추가주석: UserInfo, PoolInfo 의 구조 정의입니다. 아래 라인에 변수선언이 있습니다.
    // Info of each user.
    struct UserInfo {
        uint256 amount;     // How many LP tokens the user has provided.
        uint256 rewardDebt; // Reward debt. See explanation below.
        //
        // We do some fancy math here. Basically, any point in time, the amount of SUSHIs
        // entitled to a user but is pending to be distributed is:
        //
        //   pending reward = (user.amount * pool.accSushiPerShare) - user.rewardDebt
        //
        // Whenever a user deposits or withdraws LP tokens to a pool. Here's what happens:
        //   1. The pool's `accSushiPerShare` (and `lastRewardBlock`) gets updated.
        //   2. User receives the pending reward sent to his/her address.
        //   3. User's `amount` gets updated.
        //   4. User's `rewardDebt` gets updated.
    }

    // Info of each pool.
    struct PoolInfo {
        IERC20 lpToken;           // Address of LP token contract.
        uint256 allocPoint;       // How many allocation points assigned to this pool. SUSHIs to distribute per block.
        uint256 lastRewardBlock;  // Last block number that SUSHIs distribution occurs.
        uint256 accSushiPerShare; // Accumulated SUSHIs per share, times 1e12. See below.
    }
		// 추가주석: 마스터쉐프는 이자를 주기 위해 ERC20 토큰(SushiToken)의 Owner역할을 가집니다.
    // The SUSHI TOKEN!
    SushiToken public sushi;
		// 추가주석: 개발자의 혜택으로 풀에서 이자의 10%를 받게 됩니다.
    // Dev address.
    address public devaddr;
		// 추가주석: 스시스왑의 보너스 만료 블록번호입니다. 보너스기간에는 리워드는 10배로 받게 됩니다.
    // Block number when bonus SUSHI period ends.
    uint256 public bonusEndBlock;
		// 추가주석: 블럭생성마다 이자토큰 발행 갯수 입니다.
    // SUSHI tokens created per block.
    uint256 public sushiPerBlock;
		// 추가주석: 보너스기간동안 리워드가 몇배로 받는 변수입니다.
    // Bonus muliplier for early sushi makers.
    uint256 public constant BONUS_MULTIPLIER = 10;
    // The migrator contract. It has a lot of power. Can only be set through governance (owner).
    IMigratorChef public migrator;

		// 추가주석: 풀 목록
    // Info of each pool.
    PoolInfo[] public poolInfo;
		// 추가주석: 예치한 사용자 정보
    // Info of each user that stakes LP tokens.
    mapping (uint256 => mapping (address => UserInfo)) public userInfo;
		// 추가주석: 리워드 분배비율
    // Total allocation poitns. Must be the sum of all allocation points in all pools.
    uint256 public totalAllocPoint = 0;
		// 추가주석: 예치 시작할 블럭 번호
    // The block number when SUSHI mining starts.
    uint256 public startBlock;

    event Deposit(address indexed user, uint256 indexed pid, uint256 amount);
    event Withdraw(address indexed user, uint256 indexed pid, uint256 amount);
    event EmergencyWithdraw(address indexed user, uint256 indexed pid, uint256 amount);

    constructor(
        SushiToken _sushi,
        address _devaddr,
        uint256 _sushiPerBlock,
        uint256 _startBlock,
        uint256 _bonusEndBlock
    ) public {
        sushi = _sushi;
        devaddr = _devaddr;
        sushiPerBlock = _sushiPerBlock;
        bonusEndBlock = _bonusEndBlock;
        startBlock = _startBlock;
    }
		// 추가주석: 마스터쉐프가 보유한 풀의 갯수
    function poolLength() external view returns (uint256) {
        return poolInfo.length;
    }
		// 추가주석: 풀을 추가할때 사용합니다.
    // Add a new lp to the pool. Can only be called by the owner.
    // XXX DO NOT add the same LP token more than once. Rewards will be messed up if you do.
    function add(uint256 _allocPoint, IERC20 _lpToken, bool _withUpdate) public onlyOwner {
        if (_withUpdate) {
            massUpdatePools();
        }
        uint256 lastRewardBlock = block.number > startBlock ? block.number : startBlock;
        totalAllocPoint = totalAllocPoint.add(_allocPoint);
        poolInfo.push(PoolInfo({
            lpToken: _lpToken,
            allocPoint: _allocPoint,
            lastRewardBlock: lastRewardBlock,
            accSushiPerShare: 0
        }));
    }
		// 추가주석: 풀의 리워드 비율을 조정합니다.
    // Update the given pool's SUSHI allocation point. Can only be called by the owner.
    function set(uint256 _pid, uint256 _allocPoint, bool _withUpdate) public onlyOwner {
        if (_withUpdate) {
            massUpdatePools();
        }
        totalAllocPoint = totalAllocPoint.sub(poolInfo[_pid].allocPoint).add(_allocPoint);
        poolInfo[_pid].allocPoint = _allocPoint;
    }

    // Set the migrator contract. Can only be called by the owner.
    function setMigrator(IMigratorChef _migrator) public onlyOwner {
        migrator = _migrator;
    }

    // Migrate lp token to another lp contract. Can be called by anyone. We trust that migrator contract is good.
    function migrate(uint256 _pid) public {
        require(address(migrator) != address(0), "migrate: no migrator");
        PoolInfo storage pool = poolInfo[_pid];
        IERC20 lpToken = pool.lpToken;
        uint256 bal = lpToken.balanceOf(address(this));
        lpToken.safeApprove(address(migrator), bal);
        IERC20 newLpToken = migrator.migrate(lpToken);
        require(bal == newLpToken.balanceOf(address(this)), "migrate: bad");
        pool.lpToken = newLpToken;
    }

    // Return reward multiplier over the given _from to _to block.
    function getMultiplier(uint256 _from, uint256 _to) public view returns (uint256) {
        if (_to <= bonusEndBlock) {
            return _to.sub(_from).mul(BONUS_MULTIPLIER);
        } else if (_from >= bonusEndBlock) {
            return _to.sub(_from);
        } else {
            return bonusEndBlock.sub(_from).mul(BONUS_MULTIPLIER).add(
                _to.sub(bonusEndBlock)
            );
        }
    }
		// 추가주석: 예치한 풀에서 발생된 이자를 조회합니다.
    // View function to see pending SUSHIs on frontend.
    function pendingSushi(uint256 _pid, address _user) external view returns (uint256) {
        PoolInfo storage pool = poolInfo[_pid];
        UserInfo storage user = userInfo[_pid][_user];
        uint256 accSushiPerShare = pool.accSushiPerShare;
        uint256 lpSupply = pool.lpToken.balanceOf(address(this));
        if (block.number > pool.lastRewardBlock && lpSupply != 0) {
            uint256 multiplier = getMultiplier(pool.lastRewardBlock, block.number);
            uint256 sushiReward = multiplier.mul(sushiPerBlock).mul(pool.allocPoint).div(totalAllocPoint);
            accSushiPerShare = accSushiPerShare.add(sushiReward.mul(1e12).div(lpSupply));
        }
        return user.amount.mul(accSushiPerShare).div(1e12).sub(user.rewardDebt);
    }

    // Update reward variables for all pools. Be careful of gas spending!
    function massUpdatePools() public {
        uint256 length = poolInfo.length;
        for (uint256 pid = 0; pid < length; ++pid) {
            updatePool(pid);
        }
    }

    // Update reward variables of the given pool to be up-to-date.
    function updatePool(uint256 _pid) public {
        PoolInfo storage pool = poolInfo[_pid];
        if (block.number <= pool.lastRewardBlock) {
            return;
        }
        uint256 lpSupply = pool.lpToken.balanceOf(address(this));
        if (lpSupply == 0) {
            pool.lastRewardBlock = block.number;
            return;
        }
        uint256 multiplier = getMultiplier(pool.lastRewardBlock, block.number);
        uint256 sushiReward = multiplier.mul(sushiPerBlock).mul(pool.allocPoint).div(totalAllocPoint);
        sushi.mint(devaddr, sushiReward.div(10));
        sushi.mint(address(this), sushiReward);
        pool.accSushiPerShare = pool.accSushiPerShare.add(sushiReward.mul(1e12).div(lpSupply));
        pool.lastRewardBlock = block.number;
    }
		// 추가주석: 풀에 예치합니다.
    // Deposit LP tokens to MasterChef for SUSHI allocation.
    function deposit(uint256 _pid, uint256 _amount) public {
        PoolInfo storage pool = poolInfo[_pid];
        UserInfo storage user = userInfo[_pid][msg.sender];
        updatePool(_pid);
        if (user.amount > 0) {
            uint256 pending = user.amount.mul(pool.accSushiPerShare).div(1e12).sub(user.rewardDebt);
            if(pending > 0) {
                safeSushiTransfer(msg.sender, pending);
            }
        }
        if(_amount > 0) {
            pool.lpToken.safeTransferFrom(address(msg.sender), address(this), _amount);
            user.amount = user.amount.add(_amount);
        }
        user.rewardDebt = user.amount.mul(pool.accSushiPerShare).div(1e12);
        emit Deposit(msg.sender, _pid, _amount);
    }
		// 추가주석: 풀에서 출금합니다.
    // Withdraw LP tokens from MasterChef.
    function withdraw(uint256 _pid, uint256 _amount) public {
        PoolInfo storage pool = poolInfo[_pid];
        UserInfo storage user = userInfo[_pid][msg.sender];
        require(user.amount >= _amount, "withdraw: not good");
        updatePool(_pid);
        uint256 pending = user.amount.mul(pool.accSushiPerShare).div(1e12).sub(user.rewardDebt);
        if(pending > 0) {
            safeSushiTransfer(msg.sender, pending);
        }
        if(_amount > 0) {
            user.amount = user.amount.sub(_amount);
            pool.lpToken.safeTransfer(address(msg.sender), _amount);
        }
        user.rewardDebt = user.amount.mul(pool.accSushiPerShare).div(1e12);
        emit Withdraw(msg.sender, _pid, _amount);
    }
		// 추가주석: 긴급상황에서 풀의 예치금을 빼게됩니다.
    // Withdraw without caring about rewards. EMERGENCY ONLY.
    function emergencyWithdraw(uint256 _pid) public {
        PoolInfo storage pool = poolInfo[_pid];
        UserInfo storage user = userInfo[_pid][msg.sender];
        pool.lpToken.safeTransfer(address(msg.sender), user.amount);
        emit EmergencyWithdraw(msg.sender, _pid, user.amount);
        user.amount = 0;
        user.rewardDebt = 0;
    }

    // Safe sushi transfer function, just in case if rounding error causes pool to not have enough SUSHIs.
    function safeSushiTransfer(address _to, uint256 _amount) internal {
        uint256 sushiBal = sushi.balanceOf(address(this));
        if (_amount > sushiBal) {
            sushi.transfer(_to, sushiBal);
        } else {
            sushi.transfer(_to, _amount);
        }
    }
		// 추가주석: 개발저 주소 변경
    // Update dev address by the previous dev.
    function dev(address _devaddr) public {
        require(msg.sender == devaddr, "dev: wut?");
        devaddr = _devaddr;
    }
}
```

**SushiToken 컨트렉트**

스시 토큰 컨트렉트는 마스터쉐프가 오너가 되어 리워드를 발행하게 됩니다. ERC20 규격을 상속받았고, 특이사항으로 Governance 관련 함수가 추가되어 있습니다. 이번에는 거버넌스관련해서는 다루지 않고 예치를 통해 리워드 발생에 대해서만 알아볼 예정입니다.

```solidity
pragma solidity 0.6.12;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

// SushiToken with Governance.
contract SushiToken is ERC20("Danawa", "DNA"), Ownable {
    /// @notice Creates `_amount` token to `_to`. Must only be called by the owner (MasterChef).
    function mint(address _to, uint256 _amount) public onlyOwner {
        _mint(_to, _amount);
        _moveDelegates(address(0), _delegates[_to], _amount);
    }

    // Copied and modified from YAM code:
    // https://github.com/yam-finance/yam-protocol/blob/master/contracts/token/YAMGovernanceStorage.sol
    // https://github.com/yam-finance/yam-protocol/blob/master/contracts/token/YAMGovernance.sol
    // Which is copied and modified from COMPOUND:
    // https://github.com/compound-finance/compound-protocol/blob/master/contracts/Governance/Comp.sol

    /// @notice A record of each accounts delegate
    mapping (address => address) internal _delegates;

    /// @notice A checkpoint for marking number of votes from a given block
    struct Checkpoint {
        uint32 fromBlock;
        uint256 votes;
    }

    /// @notice A record of votes checkpoints for each account, by index
    mapping (address => mapping (uint32 => Checkpoint)) public checkpoints;

    /// @notice The number of checkpoints for each account
    mapping (address => uint32) public numCheckpoints;

    /// @notice The EIP-712 typehash for the contract's domain
    bytes32 public constant DOMAIN_TYPEHASH = keccak256("EIP712Domain(string name,uint256 chainId,address verifyingContract)");

    /// @notice The EIP-712 typehash for the delegation struct used by the contract
    bytes32 public constant DELEGATION_TYPEHASH = keccak256("Delegation(address delegatee,uint256 nonce,uint256 expiry)");

    /// @notice A record of states for signing / validating signatures
    mapping (address => uint) public nonces;

      /// @notice An event thats emitted when an account changes its delegate
    event DelegateChanged(address indexed delegator, address indexed fromDelegate, address indexed toDelegate);

    /// @notice An event thats emitted when a delegate account's vote balance changes
    event DelegateVotesChanged(address indexed delegate, uint previousBalance, uint newBalance);

    /**
     * @notice Delegate votes from `msg.sender` to `delegatee`
     * @param delegator The address to get delegatee for
     */
    function delegates(address delegator)
        external
        view
        returns (address)
    {
        return _delegates[delegator];
    }

   /**
    * @notice Delegate votes from `msg.sender` to `delegatee`
    * @param delegatee The address to delegate votes to
    */
    function delegate(address delegatee) external {
        return _delegate(msg.sender, delegatee);
    }

    /**
     * @notice Delegates votes from signatory to `delegatee`
     * @param delegatee The address to delegate votes to
     * @param nonce The contract state required to match the signature
     * @param expiry The time at which to expire the signature
     * @param v The recovery byte of the signature
     * @param r Half of the ECDSA signature pair
     * @param s Half of the ECDSA signature pair
     */
    function delegateBySig(
        address delegatee,
        uint nonce,
        uint expiry,
        uint8 v,
        bytes32 r,
        bytes32 s
    )
        external
    {
        bytes32 domainSeparator = keccak256(
            abi.encode(
                DOMAIN_TYPEHASH,
                keccak256(bytes(name())),
                getChainId(),
                address(this)
            )
        );

        bytes32 structHash = keccak256(
            abi.encode(
                DELEGATION_TYPEHASH,
                delegatee,
                nonce,
                expiry
            )
        );

        bytes32 digest = keccak256(
            abi.encodePacked(
                "\x19\x01",
                domainSeparator,
                structHash
            )
        );

        address signatory = ecrecover(digest, v, r, s);
        require(signatory != address(0), "SUSHI::delegateBySig: invalid signature");
        require(nonce == nonces[signatory]++, "SUSHI::delegateBySig: invalid nonce");
        require(now <= expiry, "SUSHI::delegateBySig: signature expired");
        return _delegate(signatory, delegatee);
    }

    /**
     * @notice Gets the current votes balance for `account`
     * @param account The address to get votes balance
     * @return The number of current votes for `account`
     */
    function getCurrentVotes(address account)
        external
        view
        returns (uint256)
    {
        uint32 nCheckpoints = numCheckpoints[account];
        return nCheckpoints > 0 ? checkpoints[account][nCheckpoints - 1].votes : 0;
    }

    /**
     * @notice Determine the prior number of votes for an account as of a block number
     * @dev Block number must be a finalized block or else this function will revert to prevent misinformation.
     * @param account The address of the account to check
     * @param blockNumber The block number to get the vote balance at
     * @return The number of votes the account had as of the given block
     */
    function getPriorVotes(address account, uint blockNumber)
        external
        view
        returns (uint256)
    {
        require(blockNumber < block.number, "SUSHI::getPriorVotes: not yet determined");

        uint32 nCheckpoints = numCheckpoints[account];
        if (nCheckpoints == 0) {
            return 0;
        }

        // First check most recent balance
        if (checkpoints[account][nCheckpoints - 1].fromBlock <= blockNumber) {
            return checkpoints[account][nCheckpoints - 1].votes;
        }

        // Next check implicit zero balance
        if (checkpoints[account][0].fromBlock > blockNumber) {
            return 0;
        }

        uint32 lower = 0;
        uint32 upper = nCheckpoints - 1;
        while (upper > lower) {
            uint32 center = upper - (upper - lower) / 2; // ceil, avoiding overflow
            Checkpoint memory cp = checkpoints[account][center];
            if (cp.fromBlock == blockNumber) {
                return cp.votes;
            } else if (cp.fromBlock < blockNumber) {
                lower = center;
            } else {
                upper = center - 1;
            }
        }
        return checkpoints[account][lower].votes;
    }

    function _delegate(address delegator, address delegatee)
        internal
    {
        address currentDelegate = _delegates[delegator];
        uint256 delegatorBalance = balanceOf(delegator); // balance of underlying SUSHIs (not scaled);
        _delegates[delegator] = delegatee;

        emit DelegateChanged(delegator, currentDelegate, delegatee);

        _moveDelegates(currentDelegate, delegatee, delegatorBalance);
    }

    function _moveDelegates(address srcRep, address dstRep, uint256 amount) internal {
        if (srcRep != dstRep && amount > 0) {
            if (srcRep != address(0)) {
                // decrease old representative
                uint32 srcRepNum = numCheckpoints[srcRep];
                uint256 srcRepOld = srcRepNum > 0 ? checkpoints[srcRep][srcRepNum - 1].votes : 0;
                uint256 srcRepNew = srcRepOld.sub(amount);
                _writeCheckpoint(srcRep, srcRepNum, srcRepOld, srcRepNew);
            }

            if (dstRep != address(0)) {
                // increase new representative
                uint32 dstRepNum = numCheckpoints[dstRep];
                uint256 dstRepOld = dstRepNum > 0 ? checkpoints[dstRep][dstRepNum - 1].votes : 0;
                uint256 dstRepNew = dstRepOld.add(amount);
                _writeCheckpoint(dstRep, dstRepNum, dstRepOld, dstRepNew);
            }
        }
    }

    function _writeCheckpoint(
        address delegatee,
        uint32 nCheckpoints,
        uint256 oldVotes,
        uint256 newVotes
    )
        internal
    {
        uint32 blockNumber = safe32(block.number, "SUSHI::_writeCheckpoint: block number exceeds 32 bits");

        if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {
            checkpoints[delegatee][nCheckpoints - 1].votes = newVotes;
        } else {
            checkpoints[delegatee][nCheckpoints] = Checkpoint(blockNumber, newVotes);
            numCheckpoints[delegatee] = nCheckpoints + 1;
        }

        emit DelegateVotesChanged(delegatee, oldVotes, newVotes);
    }

    function safe32(uint n, string memory errorMessage) internal pure returns (uint32) {
        require(n < 2**32, errorMessage);
        return uint32(n);
    }

    function getChainId() internal pure returns (uint) {
        uint256 chainId;
        assembly { chainId := chainid() }
        return chainId;
    }
}
```

### 동작방식

**입금**

1. 사용자는 LP 토큰을 마스터쉐프로 이체 승인과 입금을 하게 됩니다. 
2. poolInfo, userInfo 변수에 토큰수량을 변경합니다.
3. 입금된 토큰은 입금금액만큼 마스터쉐프는 해당 토큰API를 호출하여 입금자 지갑에서 토큰을 마스터쉐프 주소로 전송을 호출합니다. 최종 토큰은 마스터쉐프로 전송한 상태가 됩니다.

![Untitled](/images/2021-12-24-blockchain-defi-sushiswap-staking/Untitled.png)

**리워드 조회**

1. 사용자가 풀 아이디와 사용자 주소로 리워드를 조회합니다.
2. 마스터쉐프는 아래 공식에 따라 리워드가 리워드를 계산하여 응답합니다.
    1. 공식 해석: 리워드 = (예치금 * 주당 누적 스시) - 차감할 금액

출**금**

1. 계산된 리워드 금액만큼 sushiToken의 mint를 호출하여 사용자한테 발행해줍니다.
2. 출금할 금액은 마스터쉐프 주소에 있는 토큰 잔고에서 전송합니다.

### Truffle 테스트

**MasterChef 배포 및  풀 추가**

마스터쉐프를 배포합니다. 여기서 중요한 점은 리워드 발행을 위해 dnaToken(sushiToken) 소유권을 masterChef한테 넘겨줘야합니다.  그리고 스왑편에서 유동성 풀에 토큰 쌍을 추가를 했었습니다. 그러면 그에 증표로 LP토큰을 발급을 받았었습니다. 이번에는 LP토큰을 스테이킹을 통해 수익발생시키는 목적으로 사용할 예정입니다. LP토큰을 마스터쉐프 풀에 등록합니다.

![Untitled](/images/2021-12-24-blockchain-defi-sushiswap-staking/Untitled%201.png)

마스터쉐프 생성 파라미터

```solidity
constructor(
    SushiToken _sushi,       // 리워드 토큰 주소
    address _devaddr,        // 개발자 주소
    uint256 _sushiPerBlock,  // 블럭 당 리워드 토큰 수
    uint256 _startBlock,     // 풀 시작 블럭 번호
    uint256 _bonusEndBlock   // 보너스 끝나는 블럭 번호
)
```

**리워드 과정**

LP토큰을 30개를 풀에 입금한 후, 10초 대기하여 리워드가 생기면 출금하는 코드입니다.

![Untitled](/images/2021-12-24-blockchain-defi-sushiswap-staking/Untitled%202.png)

## 정리

이렇게 DeFi에서 스테이킹을 알아보았습니다. 이번편에서 중요한점은 리워드 토큰의 오너가 마스터쉐프로 변경은 필수였습니다. 마스터쉐프에 풀을 만들고 사용자가 혼자라면 블럭 당 발생하는 모든 리워드를 혼자 받게되고, 사용자가 점점 늘어나게 되면 사용자별로 예치한 금액의 비율에 따라 리워드가 발생하는걸 확인 할 수 있었습니다. 

참고링크

- [https://docs.uniswap.org/protocol/V2/guides/smart-contract-integration/providing-liquidity](https://docs.uniswap.org/protocol/V2/guides/smart-contract-integration/providing-liquidity)
- [https://docs.uniswap.org/protocol/V1/guides/pool-liquidity](https://docs.uniswap.org/protocol/V1/guides/pool-liquidity)
- [https://docs.sushi.com/products/amm-exchange/liquidity-pools](https://docs.sushi.com/products/amm-exchange/liquidity-pools)
- [https://www.certora.com/pubs/MasterChefV2April2021.pdf](https://www.certora.com/pubs/MasterChefV2April2021.pdf)