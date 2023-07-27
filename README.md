# Smart-Contract-Security-Checklist
A Smart Contract Audit Checklist: A Comprehensive Guide to Ensuring Secure and Robust Smart Contracts


Please be aware that the following code is not an exhaustive list, and keep in mind that Solidity is continually evolving. Therefore, it is crucial for you to conduct thorough research and exercise due diligence when dealing with your specific smart contract.

It's important to note that not all the items listed below may be applicable to your particular smart contract.


### Smart Contract Review Checklist:

- [ ] Read the project's docs, specs, and whitepaper to understand what the smart contracts are meant to do.

- [ ] Construct a mental model of what you expect the contracts to look like before checking out the code.

- [ ] Glance over the contracts to get a sense of the project's architecture. Tools like Surya can come in handy.

- [ ] Compare the architecture to your mental model. Look into areas that are surprising.

- [ ] Create a threat model and make a list of theoretical high-level attack vectors.

- [ ] Look at areas that can do value exchange. Especially functions like `transfer`, `transferFrom`, `send`, `call`, `delegatecall`, and `selfdestruct`. Walk backward from them to ensure they are secured properly.

- [ ] Look at areas that interface with external contracts and ensure all assumptions about them are valid, like share price only increases, etc.

- [ ] Do a generic line-by-line review of the contracts.

- [ ] Do another review from the perspective of every actor in the threat model.

- [ ] Glance over the project's tests + code coverage and look deeper at areas lacking coverage.

- [ ] Run tools like Slither/Solhint and review their output.

- [ ] Look at related projects and their audits to check for any similar issues or oversights.

### Variables

- [ ] `V1` - Can it be `internal`?

```solidity
contract ExampleContract {
    uint internal V1;
}
```

- [ ] `V2` - Can it be `constant`?

```solidity
contract ExampleContract {
    uint constant V2 = 42;
}
```

- [ ] `V3` - Can it be `immutable`?

```solidity
contract ExampleContract {
    address immutable V3 = 0xAbCdEf0123456789;
}
```

- [ ] `V4` - Is its visibility set? (SWC-108)

```solidity
contract ExampleContract {
    uint private V4;
}
```

- [ ] `V5` - Is the purpose of the variable and other important information documented using natspec?

```solidity
contract ExampleContract {
    uint public V5;

    /// @dev The total count of items in the contract.
    uint public itemCount;
}
```

- [ ] `V6` - Can it be packed with an adjacent storage variable?

```solidity
contract ExampleContract {
    struct PackedStruct {
        uint16 a;
        uint16 b;
    }

    PackedStruct public V6;
}
```

- [ ] [ ] `V7` - Can it be packed in a struct with more than 1 other variable?

```solidity
contract ExampleContract {
    struct PackedStruct {
        uint16 a;
        uint16 b;
    }

    struct MultiVarStruct {
        uint32 c;
        PackedStruct d;
    }

    MultiVarStruct public V7;
}
```

- [ ] `V8` - Use full 256 bit types unless packing with other variables.

```solidity
contract ExampleContract {
    uint256 public V8;
}
```

- [ ] `V9` - If it's a public array, is a separate function provided to return the full array?

```solidity
contract ExampleContract {
    uint[] public V9;

    function getV9() public view returns (uint[] memory) {
        return V9;
    }
}
```

- [ ] `V10` - Only use `private` to intentionally prevent child contracts from accessing the variable, prefer `internal` for flexibility.

```solidity
contract ExampleContract {
    uint internal V10;
}
```

### Structs

- [ ] `S1` - Is a struct necessary? Can the variable be packed raw in storage?

```solidity
contract ExampleContract {
    uint32 public S1;
}
```

- [ ] `S2` - Are its fields packed together (if possible)?

```solidity
contract ExampleContract {
    struct PackedStruct {
        uint16 a;
        uint16 b;
    }

    PackedStruct public S2;
}
```

- [ ] `S3` - Is the purpose of the struct and all fields documented using natspec?

```solidity
contract ExampleContract {
    struct UserInfo {
        address userAddress;
        uint256 balance;
    }

    /// @dev Mapping of user addresses to their corresponding UserInfo.
    mapping(address => UserInfo) public userInfos;
}
```

```


## Functions

- [ ] `F1` - Can it be `external`?

```solidity
contract ExampleContract {
    function processExternal() external {
        // Function logic
    }
}
```

- [ ] `F2` - Should it be `internal`?

```solidity
contract ExampleContract {
    function processInternal() internal {
        // Function logic
    }
}
```

- [ ] `F3` - Should it be `payable`?

```solidity
contract ExampleContract {
    function receiveFunds() external payable {
        // Function logic to receive funds
    }
}
```

- [ ] `F4` - Can it be combined with another similar function?

```solidity
contract ExampleContract {
    uint256 public counter;

    function increment() external {
        counter++;
    }

    function decrement() external {
        counter--;
    }
}
```

- [ ] `F5` - Validate all parameters are within safe bounds, even if the function can only be called by trusted users.

```solidity
contract ExampleContract {
    function safeTransfer(uint256 amount, address recipient) external {
        require(amount > 0, "Amount must be greater than zero");
        require(recipient != address(0), "Invalid recipient address");
        // Transfer logic
    }
}
```

- [ ] `F6` - Is the checks before effects pattern followed? (SWC-107)

```solidity
contract ExampleContract {
    function withdraw(uint256 amount) external {
        require(amount <= balances[msg.sender], "Insufficient balance");
        balances[msg.sender] -= amount;
        msg.sender.transfer(amount);
    }
}
```

- [ ] `F7` - Check for front-running possibilities, such as the approve function. (SWC-114)

```solidity
contract ExampleContract {
    function approve(address spender, uint256 amount) external {
        // Check if spender already has an allowance and adjust accordingly
        // Potential front-running vulnerability may exist here
        // Function logic to approve spender for a certain amount
    }
}
```

- [ ] `F8` - Is insufficient gas griefing possible? (SWC-126)

```solidity
contract ExampleContract {
    function transfer(address payable recipient) external {
        recipient.transfer(msg.sender.balance); // Potential griefing vulnerability, prefer transferWithGasLimit
    }
}
```

- [ ] `F9` - Are the correct modifiers applied, such as `onlyOwner`/`requiresAuth`?

```solidity
contract ExampleContract {
    address public owner;

    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can call this function");
        _;
    }

    function changeOwner(address newOwner) external onlyOwner {
        owner = newOwner;
    }
}
```

- [ ] `F10` - Are return values always assigned?

```solidity
contract ExampleContract {
    function calculateValue(uint256 a, uint256 b) external returns (uint256) {
        uint256 result = a + b;
        // Do something with the result
        return result;
    }
}
```

- [ ] `F11` - Write down and test invariants about state before a function can run correctly.

```solidity
contract ExampleContract {
    uint256 public totalSupply;
    mapping(address => uint256) public balances;

    function mint(address account, uint256 amount) external {
        require(account != address(0), "Invalid account address");
        require(amount > 0, "Amount must be greater than zero");
        totalSupply += amount;
        balances[account] += amount;
    }
}
```

- [ ] `F12` - Write down and test invariants about the return or any changes to state after a function has run.

```solidity
contract ExampleContract {
    uint256 public totalSupply;
    mapping(address => uint256) public balances;

    function burn(address account, uint256 amount) external {
        require(account != address(0), "Invalid account address");
        require(amount > 0, "Amount must be greater than zero");
        require(balances[account] >= amount, "Insufficient balance");
        totalSupply -= amount;
        balances[account] -= amount;
    }
}
```

- [ ] `F13` - Take care when naming functions, because people will assume behavior based on the name.

```solidity
contract ExampleContract {
    function kill() external {
        // Function name may imply termination, but it actually performs some cleanup
        // Consider using a more descriptive name
    }
}
```

- [ ] `F14` - If a function is intentionally unsafe (to save gas, etc), use an unwieldy name to draw attention to its risk.

```solidity
contract ExampleContract {
    function unsafeTransfer() external {
        // Function performs a transfer without checks to save gas, use with caution
    }
}
```

- [ ] `F15` - Are all arguments, return values, side effects, and other information documented using natspec?

```solidity
contract ExampleContract {
    /// @dev Transfers tokens from sender to recipient.
    /// @param recipient The address to receive the tokens.
    /// @param amount The amount of tokens to transfer.
    function transfer(address recipient, uint256 amount) external {
        // Function logic to transfer tokens
    }
}
```

- [ ] `F16` - If the function allows operating on another user in the system, do not assume `msg.sender` is the user being operated on.

```solidity
contract ExampleContract {
    mapping(address => uint256) public balances;

    function transferFrom(address sender, address recipient, uint256 amount) external {
        require(balances[sender] >= amount, "Insufficient balance");
        // Ensure the function caller has enough allowance to transfer from sender
        require(allowance[sender][msg.sender] >= amount, "Not enough allowance");
        // Transfer logic
        balances[sender] -= amount;
        balances[recipient] += amount;
    }
}
```

- [ ] `F17` - If the function requires the contract be in an uninitialized state, check an explicit `initialized` variable. Do not use `owner == address(0)` or other similar checks as substitutes.

```solidity
contract ExampleContract {
    bool private initialized;
    address private owner;

    modifier onlyUninitialized() {
        require(!initialized, "Contract already initialized");
        _;
    }

    function initialize() external onlyUninitialized {
        owner = msg.sender;
        initialized = true;
    }
}
```

- [ ] `F18` - Only use `private` to intentionally prevent child contracts from calling the function, prefer `internal` for flexibility.

```solidity
contract ExampleContract {
    function process() internal {
        // Function logic
    }
}
```

- [ ] `F19` - Use `virtual` if there are legitimate (and safe) instances where a child contract may wish to override the function's behavior.

```solidity
contract BaseContract {
    function process() external virtual {
        // Base contract logic
    }
}

contract ChildContract is BaseContract {
    function process() external override {
        // Custom logic for child contract
    }
}
```

## Modifiers

- [ ] `M1` - Are no storage updates made (except in a reentrancy lock)?

```solidity
contract ExampleContract {
    bool private locked;

    modifier reentrancyLock() {
        require(!locked, "Reentrancy not allowed");
        locked = true;
        _;
        locked = false;
    }

    function withdraw(uint256 amount) external reentrancyLock {
        // Function logic to withdraw funds
    }
}
```

- [ ] `M2` - Are external calls avoided?

```solidity
contract ExampleContract {
    address public externalContract;

    modifier avoidExternalCalls() {
        require(externalContract == address(0), "External calls not allowed");
        _;
    }

    function setExternalContract(address contractAddress) external avoidExternalCalls {
        externalContract = contractAddress;
    }
}
```

- [ ] `M3` - Is the purpose of the modifier and other important information documented using natspec?

```solidity
contract ExampleContract {
    address public owner;

    /// @dev Only the owner of the contract is allowed to call functions modified by `onlyOwner`.
    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can call this function");
        _;
    }

    /// @dev Sets the contract owner to the specified address.
    /// @param newOwner The new owner address.
    function changeOwner(address newOwner) external onlyOwner {
        owner = newOwner;
    }
}
```

## Code

- [ ] `C1` - Using SafeMath or 0.8 checked math? (SWC-101)
- [ ] `C2` - Are any storage slots read multiple times?
- [ ] `C3` - Are any unbounded loops/arrays used that can cause DoS? (SWC-128)
- [ ] `C4` - Use `block.timestamp` only for long intervals. (SWC-116)
- [ ] `C5` - Don't use block.number for elapsed time. (SWC-116)
- [ ] `C7` - Avoid delegatecall wherever possible, especially to external (even if trusted) contracts. (SWC-112)
- [ ] `C8` - Do not update the length of an array while iterating over it.
- [ ] `C9` - Don't use `blockhash()`, etc for randomness. (SWC-120)
- [ ] `C10` - Are signatures protected against replay with a nonce and `block.chainid` (SWC-121)
- [ ] `C11` - Ensure all signatures use EIP-712. (SWC-117 SWC-122)
- [ ] `C12` - Output of `abi.encodePacked()` shouldn't be hashed if using >2 dynamic types. Prefer using `abi.encode()` in general. (SWC-133)
- [ ] `C13` - Careful with assembly, don't use any arbitrary data. (SWC-127)
- [ ] `C14` - Don't assume a specific ETH balance. (SWC-132)
- [ ] `C15` - Avoid insufficient gas griefing. (SWC-126)
- [ ] `C16` - Private data isn't private. (SWC-136)
- [ ] `C17` - Updating a struct/array in memory won't modify it in storage.
- [ ] `C18` - Never shadow state variables. (SWC-119)
- [ ] `C19` - Do not mutate function parameters.
- [ ] `C20` - Is calculating a value on the fly cheaper than storing it?
- [ ] `C21` - Are all state variables read from the correct contract (master vs. clone)?
- [ ] `C22` - Are comparison operators used correctly (`>`, `<`, `>=`, `<=`), especially to prevent off-by-one errors?
- [ ] `C23` - Are logical operators used correctly (`==`, `!=`, `&&`, `||`, `!`), especially to prevent off-by-one errors?
- [ ] `C24` - Always multiply before dividing, unless the multiplication could overflow.
- [ ] `C25` - Are magic numbers replaced by a constant with an intuitive name?
- [ ] `C26` - If the recipient of ETH had a fallback function that reverted, could it cause DoS? (SWC-113)
- [ ] `C27` - Use SafeERC20 or check return values safely.
- [ ] `C28` - Don't use `msg.value` in a loop.
- [ ] `C29` - Don't use `msg.value` if recursive delegatecalls are possible (like if the contract inherits `Multicall`/`Batchable`).
- [ ] `C30` - Don't assume `msg.sender` is always a relevant user.
- [ ] `C31` - Don't use `assert()` unless for fuzzing or formal verification. (SWC-110)
- [ ] `C32` - Don't use `tx.origin` for authorization. (SWC-115)
- [ ] `C33` - Don't use `address.transfer()` or `address.send()`. Use `.call.value(...)("")` instead. (SWC-134)
- [ ] `C34` - When using low-level calls, ensure the contract exists before calling.
- [ ] `C35` - When calling a function with many parameters, use the named argument syntax.
- [ ] `C36` - Do not use assembly for create2. Prefer the modern salted contract creation syntax.
- [ ] `C37` - Do not use assembly to access chainid or contract code/size/hash. Prefer the modern Solidity syntax.
- [ ] `C38` - Use the `delete` keyword when setting a variable to a zero value (`0`, `false`, `""`, etc).
- [ ] `C39` - Comment the "why" as much as possible.
- [ ] `C40` - Comment the "what" if using obscure syntax or writing unconventional code.
- [ ] `C41` - Comment explanations + example inputs/outputs next to complex and fixed point math.
- [ ] `C42` - Comment explanations wherever optimizations are done, along with an estimate of much gas they save.
- [ ] `C43` - Comment explanations wherever certain optimizations are purposely avoided, along with an estimate of much gas they would/wouldn't save if implemented.
- [ ] `C44` - Use `unchecked` blocks where overflow/underflow is impossible, or where an overflow/underflow is unrealistic on human timescales (counters, etc). Comment explanations wherever `unchecked` is used, along with an estimate of how much gas it saves (if relevant).
- [ ] `C45` - Do not depend on Solidity's arithmetic operator precedence rules. In addition to the use of parentheses to override default operator precedence, parentheses should also be used to emphasise it.
- [ ] `C46` - Expressions passed to logical/comparison operators (`&&`/`||`/`>=`/`==`/etc) should not have side-effects.
- [ ] `C47` - Wherever arithmetic operations are performed that could result in precision loss, ensure it benefits the right actors in the system, and document it with comments.
- [ ] `C48` - Document the reason why a reentrancy lock is necessary whenever it's used with an inline or `@dev` natspec comment.
- [ ] `C49` - When fuzzing functions that only operate on specific numerical ranges use modulo to tighten the fuzzer's inputs (such as `x = x % 10000 + 1` to restrict from 1 to 10,000).
- [ ] `C50` - Use ternary expressions to simplify branching logic wherever possible.
- [ ] `C51` - When operating on more than one address, ask yourself what happens if they're the same.

## DeFi

- [ ] `D1` - Check your assumptions about what other contracts do and return.
- [ ] `D2` - Don't mix internal accounting with actual balances.
- [ ] `D3` - Don't use spot price from an AMM as an oracle.
- [ ] `D4` - Do not trade on AMMs without receiving a price target off-chain or via an oracle.
- [ ] `D5` - Use sanity checks to prevent oracle/price manipulation.
- [ ] `D6` - Watch out for rebasing tokens. If they are unsupported, ensure that property is documented.
- [ ] `D7` - Watch out for ERC-777 tokens. Even a token you trust could preform reentrancy if it's an ERC-777.
- [ ] `D8` - Watch out for fee-on-transfer tokens. If they are unsupported, ensure that property is documented.
- [ ] `D9` - Watch out for tokens that use too many or too few decimals. Ensure the max and min supported values are documented.
- [ ] `D10` - Be careful of relying on the raw token balance of a contract to determine earnings. Contracts which provide a way to recover assets sent directly to them can mess up share price functions that rely on the raw Ether or token balances of an address.
- [ ] `D11` - If your contract is a target for token approvals, do not make arbitrary calls from user input.



### reference
With help following we are able to make above list
- [BoringCrypto](https://github.com/sushiswap/bentobox/blob/master/documentation/checks.txt)
- [Mudit Gupta](https://www.youtube.com/watch?v=LLiJK_VeAvQ)
- [ConsenSys Diligence](https://consensys.github.io/smart-contract-best-practices/attacks/)
- [Runtime Verification](https://github.com/runtimeverification/verified-smart-contracts/wiki/List-of-Security-Vulnerabilities)
- [Smart Contract Auditing Heuristics](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics)
- [Solidity idiosyncrasies](https://github.com/miguelmota/solidity-idiosyncrasies)
- [Solidity security considerations](http://solidity.readthedocs.io/en/develop/security-considerations.html)
- [Methodological security review of a smart contract](https://ethereum.stackexchange.com/questions/8551/methodological-security-review-of-a-smart-contract)
- [Decentralized Application Security Project](https://dasp.co/)
- [Semgrep Smart-contracts](https://github.com/Decurity/semgrep-smart-contracts)
- [Ethereum Security Guide](https://github.com/ethereum/wiki/wiki/Safety)
- [Smart Contract Security Verification Standard](https://securing.github.io/SCSVS/)
- [Solidity Code Metrics By Consensys Diligence](https://github.com/ConsenSys/solidity-metrics)
- [The Repository this list was largely sourced from](https://github.com/Rari-Capital/solcurity)
- [Blockchain Security Audit List](https://github.com/0xNazgul/Blockchain-Security-Audit-List)
- [Smart contract best pracitices](https://github.com/ConsenSys/smart-contract-best-practices)

