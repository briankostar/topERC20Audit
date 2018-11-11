# topERC20Audit

Study of top ERC20 tokens

# Description

Collection of solidity attacks and analysis of top ERC20 tokens.

# Common Attacks

## 1. Reentry

    Ethereum smart contracts can hold, receive, and send ether. It can also call other contracts or send ether to other addresses via external call. Problem is this call can be hijacked by the receiving address through a fallback function an execute code it was not meant to, including calling code back it itself.
    Attacker sets up a contract, which, when receiving ether, will execute the fallback function with malicious code.
    Eg. If we had made a vault smart contract which allows user to store and take out their code, say we had the following requirement for withdrawl: require(msg.sender.call.value(_ethToWithdraw)());
    The malicious contract can use the vulnerable contract's address and call its functions.

### How to Prevent
    Easiest to fix this is by using tranfer() when sending eth to external contract. It only has 2300 max gas by default so there isn't enough gas to do a re-entry. You could also make sure the logics are completed before making a send call. Lastly, you could use mutex, which locks the contract during code execution.

## Int Overflow and Underflow
    Integer variables can only store up to a certain number. Eg uint8, which has 8 bits, can only hold from 0 to 255. If we try to assign 256 to it, it'll be considered a 0 (overflow). If we try to assign -1 to it, it'll be considered 255 (underflow). As this can result in unexpected behaviours, if not properly accounted for, can result in an attack.

### How to Prevent
    Easiest and the best practice method to solve this is to use tested, libraries such as OpenZeppelin's SafeMath.

## Unexpected Ether
    Solidity's selfdestruct does two things. 1. It deletes the contract bytecode off the blockchain and 2. It sends the Eth it holds to another address.
    Now consider if this other address is another contract. Normally, when Eth is sent to a contract, it's fallback function gets executed. However, this is not so in this case!
    So what?
    Well this could be potentially dangerous if we assume a contract has 0 Ether. 
    Code like require(this.balance > 0) could be false and can be bypassed.

### How to Prevent
    Just never assume that a contract has 0 Eth balance. Applying logic based on this.balance should be avoided.

## Misused of Visible Identifiers
    Solidity has 4 visibility types: Private, Internal, Public, and External.
    Private functions/variables can only be used within same contract.
    Internal is similar to 'protected' in other programming languages. It can only be used within current contract or childs of the contract.
    Public is by default. It can be called from anywhere and has default getter.
    External must by called from elsewhere. Either from other contracts or by transaction.

    Since by default everything is public, biggest vulnerability is leaving this public for functions/variables are should be hidden.

### How to Prevent
    Make sure to follow best practices and explictly state access identifiers even when not necessary.

## Randomness Illusion
    Since EVM transactions are deterministic & pure, there will always be same output for same input. 
    This makes randomness difficult. Often people use block variables such as time, hashes, etc to generate "randomness" but this is an illusion. A miner can easily modify this and not publish blocks that doesn't suit their needs.

### How to Prevent
    Easiest way to mitigate this would be by using 3rd party oracle to generate randomness. 

## Short Address 
    When executing code, EVM will encode the param into bytecode and add 0s to the end of the encoded parameters if the actual parameter is shorter than the expected. The ERC20 standard transfer() function, can then be encoded as bytecode of transferFunctionName + address + amount.
    If the user sent the request for transfer with shorter address, there is vulnerability in that EVM will add extra 0s and the request for 100Eth for example could be changed to higher.

### How to Prevent
    This is a dangerous pitfall but can be mitigated if we validate the inputs and because padding is only added at the end, you could order the params such that sensitive variables are placed first.

## Unchecked .call values
    Solidity can make external calls with .call() .send() or .transfer().
    The issue with using .call() or .send() is that if the call fails for whatever reason, it will return true/false value but will not revert code executed thus far.
    This means if you expect it to revert on fail and not explictly check for the returned success/false, you could leave vulnerabilities.
    Eg a lotto contract. Upon drawing, sends value to an address via .send() and allows remaining to be drawn right away. If this .send failed for some reason (out of gas, sent to contract that results fail, etc), anyone would be open to drawing the remaining fund.

### How to Prevent
    Use transfer() where possible as it will do rever() by default on fail. If using .call or .send(), make sure to check for the returned value.

## Race Conditions / Front
    Since miners that solve for a hash block chooses which transactions go into it (usually via gas price order), an attacker can watch the mempool and put in transactions that are favorable to them with higher gas price so that it gets included in the block before the original.
    Eg. Say there is a reward for solving a puzzle. To claim it, you have to call solve() with the solution. An attacker can listen for this in the pool, and hijack it by submitting a higher gas price. The miner will likely accept the attacker's solution first and the honest user will have no Eth to claim.
    Race conditions can be affected by those who sets the gas price in order to get in front, or even by miners themselves who orders transactions to their favor. The first is way more dangerous and likely as for the second one to work, the miner needs to be the winning miner for the block.

### How to Prevent
    One way to solve the first race condition is by putting upper limit on gas price. A better method would be to use commit-reveal method to submit transactions but this often takes more work.

## DOS / Inoperable
    This is any attack that renders a contract to be inoperable.
    This could be due to:
    1. A function call that uses all gas and prevents intended logic from being called.
        Eg. toAddress.call.value(amount)  --if the address uses all gas, logic after this wont ever be called
    2. Looping dynamic arrays. Trying to loop a dynamic array that is ever growing.. will be vulnerable to attack. Eventually the gas limit wont be able to cover the loop calculation.
    3. Privilged functions. If only owner has ability to call certain functions and the owner loses their private key, the smart contract will be rendered useless.
    4. Depending on a function to progress state. If that function fails, the contract will not be able to move forward in its state.

### How to Prevent
    1. Set max gas amount: toAddress.call.gas(50000).value(amount)
    2. Try to avoid dynamic array looping or at least prevent access to it by all.
    3. Have backup plans. Common solution is a multi-sig or time access. Ie accessible after 1 year
    4. Also have backup plans. Account for failure and perhaps add time setting to upgrade to new state after a while.

## Timestamp
    This is similar to Front vulnerability where dishonest miners can change the blocktime slightly for their favor. If a contract uses blocktime for randomness, miners can exploit this.

### How to prevent
    Do not use timestamp for randomness

## Constructor
    Constructors are special functions that are invoked automatically at contract initialization. If there is a name mismatch, it becomes like any other function and can have dire consequences.
    Consider a constructor function that does owner = _owner.
    If the name didnt match, anyone would be able to call this function to claim ownership

### How to prevent
    Make sure to have a matching name with the contract. After 0.4.22, also use 'constructor' keyword.

##  Storage type pointer
    Storage types are grouped together and stored in slots in the blockchain in order they appear on the contract.
    Another thing to note is that complex data type such as struct is also a storage type. When we define but do not initialize, it'll point to the first slot. This means.. we can manually read/change the data in the first slot.

### How to prevent
    Explictly use 'storage' or 'memory' data type and initialize them properly.

## Floating point values
    Solidity has no float numbers. Developers must implment this themselves using int. As such, there can be mistakes made.

### How to Prevent
    Be aware of the round downs and use high numbers for accuracy. Ordering of operations do matter as it affects precision.

## Transaction Origin
    Misuse of tx.origin to check for the owner can be vulnerable for phishing attacks.
    tx.origin returns the address tha started the transaction.
    Using require(tx.origin == owner) for example, can be a mistake as tx.origin can be masked.
    For example, a phisher can write a contract that calls your vulnerable contract and get actual owner to call their contract. 
    If the code was: 
     function withdrawAll(address _recipient) public {
        require(tx.origin == owner);
        _recipient.transfer(this.balance);
    }
    Then the phisher would be able to add their own _recipient and withdraw all funds,

### How to Prevent
    Do not use tx.origin