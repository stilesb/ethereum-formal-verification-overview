# Formal Verification of Ethereum Contracts (Yoichi's attempts)

This is the start page about my efforts around smart contract verification.

## Goal

The goal is to establish a method how to verify a smart contract so that
no surprises happen after their deployment.

## Links

* _every update comes here_: a [gitter channel](https://gitter.im/ethereum/formal-methods)
* [Solidity](https://gitter.im/ethereum/solidity/)
* a [Coq repo](https://github.com/pirapira/evmverif) and a [screen cast](https://youtu.be/Mzh4fyoaBJ0?list=PL9oaY6Y4QxRZybj86eGItGVApxLXVIXHz)
* an [Isabelle/HOL repo](https://github.com/pirapira/eth-isabelle)

## The rest of the page

* [Verification of EVM bytecodes](https://github.com/pirapira/ethereum-formal-verification-overview/blob/master/README.md#path-a-verification-of-evm-bytecodes-currently-followed)
* [Verification of Solidity Programs](https://github.com/pirapira/ethereum-formal-verification-overview/blob/master/README.md#path-b-verification-of-solidity-programs)
* [A Safe Programming Language](https://github.com/pirapira/ethereum-formal-verification-overview/blob/master/README.md#sideway-a-safe-programming-language)
* [Dr-Y's contract analyzer](https://github.com/pirapira/ethereum-formal-verification-overview/blob/master/README.md#sideway-dr-ys-contract-analyzer)
* [Transaction visualization](https://github.com/pirapira/ethereum-formal-verification-overview/blob/master/README.md#sideway-transaction-visualization)



## Path a: Verification of EVM bytecodes (currently followed)

One way is to look at the EVM bytecodes.  They are executed on a simple virtual machine.  The rules of the virtual machine is well understood by different Ethereum clients which usually match (otherwise they fix the difference with uttermost priority).
The current attempt in Coq is in [evmverif](https://github.com/pirapira/evmverif/) repository, and there is a [screen cast](https://youtu.be/Mzh4fyoaBJ0?list=PL9oaY6Y4QxRZybj86eGItGVApxLXVIXHz).

### Obstacles

* Coq proofs are currently [lengthy](https://github.com/pirapira/evmverif/blob/master/coq/example/managed_account_with_accumulators.v#L405).
    - I believe Isabelle/HOL provides much easier user experience (because it has a well-polished machine word library).  I'm porting the Coq attempt [into Isabelle/HOL](https://github.com/pirapira/eth-isabelle) (the general definitions have been ported; two examples have been ported; maybe one more example to port).
* At the bytecode level, it's harder to see what the code is doing and what to expect.
    - One solution is to make the Solidity compiler [annotate the bytecode](https://github.com/ethereum/solidity/issues/1178) with the expected properties at specific code location.
* At the bytecode level, Solidity array access looks like storage access with Kaccek hashes, and somehow we need to assume no collisions.  If I assume `∀ a, b. keccak(a) = keccak(b) -> a = b`, using the pigeon hole argument, I can prove `0 = 1` and everything.
    - I can locally say `sorry` or `admit` to indicate that I give up proofs whenever I have to prove `keccak(a) != keccak(b)` because `a != b`.  Then the proofs would be incomplete and I cannot call my results "theorems", but I'm happy that way.  Anyway the compiler assumes no hash collisions without logical justification (but empirical).
	- The module system of Coq provides [a clever workaround](https://github.com/bitemyapp/ledgertheory/blob/master/CryptoHashes.v).

### Steps on Path a

1. ~~try Isabelle/HOL to see if proofs are really 5 times easier there (less than a week)~~
    - By the way, about the trustworthiness, I would put Isabelle/HOL at least as good as Coq because Isabelle/HOL is based on a simpler deduction system (a deduction system is not a piece of software; it is a bunch of inference rules usually written in LaTeX)
2. ~~choose Coq or Isabelle/HOL (overnight)~~ I chose Isabelle/HOL over Coq.
3. verify `Deed` and some other simple bytecode programs against simple properties (6-10 days)
4. develop a method how to verify assertions between opcodes (this is [evmverif#5](https://github.com/pirapira/evmverif/issues/5)) (a week; there is a [milestone](https://github.com/pirapira/eth-isabelle/milestone/1) for this item)
5. cover all opcodes (3 days): this is already in progress. A [milestone](https://github.com/pirapira/eth-isabelle/milestone/2).
6. modify Solidity to output such assertions between opcodes (3 weeks)
7. verify the name registrar for some desired safety properties (2=5 weeks; now hard to guess)
8. try to automate the process of verification / finding vulnerabilities (3 months)


## Path b: Verification of Solidity programs

Another way would be to verify Solidity sources somehow,
not looking at the EVM bytecode.

### Obstacles

* The only way to know the meaning of a Solidity program is to compile it into bytecodes.
* The Solidity compiler is changing fast and the verification tools would need to follow the changes
* The verification tools tend to believe a different behavior from that of actually deployed bytecode

### Steps on this path

The steps that have to be taken:

1. Know Solidity program's meaning without checking the bytecode.  For this, one way is to write a Solidity interpreter in Coq or in an ML language.  Another way is to compile a Solidity program into WhyML or F*.
2. Check the above against the reality.  We need a big set of tests to compare the above translation against the bytecode.
3. Translate the ML implementation into a proof assistant (or we rely on Why3, in that case we need to trust the SMT solvers)
4. Try to assert properties in a shallow-embedded way and write proofs
5. Automate the process, maybe by deep-embedding some kind of Hoare logic



## Sideway: a safe programming language

This alone does not verify smart contracts, but I have a chance to make it static analyzer friendly.

The [bamboo compiler](https://github.com/pirapira/bamboo) is now producing snippets of bytecode for the empty contract (still lots to be done).

The language has
* no loops or recursions
    - this makes life much easier for static analyzers
    - gas consumption is stabler
    - users would need to call it again and again
* persistent program counter
    - this discourages multiple workflows in a single contract
    - reentrancy does not jump to far away in the source code, but reentrancy arrives around the last sentence executed
* protection against reentrancy
* explcit storage setting at every exit

It is inspired by an existing language but I will talk about it when it's actually working.


## Sideway: Dr-Y's contract analyzer

The [contract analyzer](https://github.com/pirapira/dry-analyzer) needs an overhaul in the functionality and the UX.
I want to see the meaning of each basic block, and how the execution can jump around the basic blocks.


## Sideway: transaction visualization

A quick hack [to visualize the dataflow in a transaction](https://github.com/pirapira/vmtrace_enricher).
This is useful after a surprise.
