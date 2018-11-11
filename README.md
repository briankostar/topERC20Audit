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
    When ether is sent to a contract, it must invoke a function. If it doesn't exist, it'll invoke a fallback function.


## Time attack

## DOS

## Transaction ordering in mempool

# Best Solidity Practices
