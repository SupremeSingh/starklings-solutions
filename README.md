# Starklings!

Only Dust is a great Cairo-focused think tank and study group. I highly recommend joining there discord through [here](https://www.onlydust.xyz/).  True to their reputation, they have made an awesome Cairo-based programming tutorial called [Starklings](https://github.com/onlydustxyz/starklings).  

Starklings goes through each major concept in Cairo programming step-by-step, checks your solutions in real time and even shows you the solutions! Here, I will be discussing these solutions and what we can take away from from them. 

## P1 - Syntax

Here, we will go through 5 simple exercises to get comfortable with Cairo syntax. Note that this is Cairo in the context of writing a StarkNet contract, not just vanilla Cairo. 

The **first** one familiarizes you with the flag needed to identify a Cairo program as a smart contract. The **second** one introduces you to the concept of built-ins. In this case, 

- `syscall_ptr`: Allows the contract to make low-level system calls
- `range_check_ptr`: Allows the program to compare integer values
- `pedersen_ptr`: Let's the program calculate a 252-bit Pedersen hash 

It may not be apparent why we are calling these imports when the program isn't necessarily using them. This is because the Cairo compiler saves storage variables in slots as per an algorithm. The contract requires these implicit arguments to compute the actual memory address of this variable if you ever try to retrieve it.

The **third** one talks about variable entry, and return values. And the **fifth** one discusses structs, which we spoke about at length in the Cairo playground solutions. 

The most interesting one is the **fourth**, because it introduces the concept of **namespaces**. Let's understand how Cairo stores our variables exactly - 

Local contract storage is a persistent space where you can  read, write and modify data. Storage is a map with  2²⁵¹ slots, where each slot is a felt and is initialized to 0. All temporary variables are simply assigned to one such slot, and retrieved later by the program using the slot's number. 

For storage variables, however, one which store the state of the contract - StarkNet maps their name and values to a unique address generate by a hashing algorithm called `sn_keccak`. (Make sense why we imported that hashing built-in earlier ?)

So the thing with namespaces is they allow us to containerize code from different contracts and libraries when are we inheriting or importing them from one place to another. For instance, I could have ... 

```
./libA.cairo

namespace LIBRARY_A:
	func increase_balance{
	syscall_ptr : felt*, pedersen_ptr : HashBuiltin*,
	range_check_ptr}(amount : felt):
		let (res) = balance.read()
		balance.write(res + amount)
		return ()
	end
end
```
and another library like ...
```
./libB.cairo

namespace LIBRARY_B:
	func increase_balance{
	syscall_ptr : felt*, pedersen_ptr : HashBuiltin*,
	range_check_ptr}(amount : felt):
		let (res) = balance.read()
		balance.write(res * amount)
		return ()
	end
end
```

Now, if, for some reason, I need both Libraries A and B in a contract, and they have this one function with the same name - that's going to cause a massive memory mess in Cairo. 

However, putting variables and functions in namespaces allow us to use them despite having the same identifiers by prefixing the name of the Namespace to them ... like -

```
LIBRARY_A.increase_balance(amount)
```

Read more about this concept [here](https://medium.com/coinmonks/storage-variable-clashing-in-starknet-ce5f28e60886). 
