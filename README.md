# topERC20Audit

Study of top ERC20 tokens

# Description

Collection of solidity attacks and analysis of top ERC20 tokens.

# Common Attacks

## 1. Reentry

    Ethereum smart contracts can hold, receive, and send ether. It can also call other contracts or send ether to other addresses via external call. Problem is this call can be hijacked by the receiving address through a fallback function an execute code it was not meant to, including calling code back it itself.
    Attacker sets up a contract, which, when receiving ether, will execute the fallback function with malicious code.
    Eg. If we had made a vault smart contract which allows user to store and take out their code, say we had the following requirement for withdrawl: require(msg.sender.call.value(_ethToWithdraw)());

### How to Prevent

## Int Overflow and Underflow

## Time attack

## DOS

## Transaction ordering in mempool

# Best Solidity Practices
