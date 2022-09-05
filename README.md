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

 ## P2 - Strings

Here, we will be working with short string literals in Cairo. Strings ?? I thought we didn't have anything but felts in Cairo. You are right - we don't. But we can always have more abstract data types that ultimately compile into felts. 

Short string literals can be written into a contract as strings, hex code or literally as felts. To convert between all these data types, look for the `strings.py` file in the `utils` folder above. Essentially, this allows you to visualize one data type as another. 

Just remember, you see the string, not Cairo or StarkNet. They only see numbers, so you can even perform arithmetic operations on strings themselves as you can tell from the solutions. 

 ## P3 - Implicit Arguments

So far, we have discussed implicit parameters such as Pedersen hash pointers, which are required by certain Cairo functions. Interestingly, functions can also accept custom implicit parameters in Cairo !

Since felts are the default data type for everything in Cairo, if your custom implicit parameter is a felt you do not need to give it a type. Doing something like this is enough - 

```
func parent_function{a, b}() -> (result : felt):
```
However, a data type is certainly in order for anything else - 
```
func test_ok{syscall_ptr : felt*}():
```

It is useful to remember that if you wish to edit the value of an implicit parameter in a function, you have to instantiate it by the same name in the function itself. For instance, to change the value of `a` above - 

```
func parent_function{a, b}() -> (result : felt):
	let a = 2 
	return()
end
```

Finally, these parameters **must** be passed down to all the functions which  use the base function in which they are defined. 

 ## P4 - Storage

We have already chatted a bit about Storage in Cairo. First things first, Cairo doesn't (necessarily) have a concept of global, storage variables . A global variable is just a function which takes no arguments and returns exactly 1 value. 

Similarly, a mapping is a value which takes 1 input and, using some process, returns a corresponding output. Because of this, mappings in Cairo can be customized way more than traditional Solidity ones. 

As we covered, a storage variable can be declared using a specific flag - 

```
@storage_var 
func bool() -> (bool:felt):
end
```
and then StarkNet will basically generate a mapping under the hood between the variable name, it's value and a hash-generated address. 

Finally, there are 2 ways to access a storage variables in Cairo - 

- `.read()`: Allows you to get the data out of that function 
- `.write()`: Allows you to modify state

Do remember that a `write` operation must be done from an `external` function in the contract, since you do not want the contract to be able to change it's own state without external interaction. 

 - let - Refers to any reference in memory, can be given a [type](https://www.cairo-lang.org/docs/how_cairo_works/consts.html#typed-references)
 - local -  Stored in the frame pointer, may have a type and not revoked throughout program (see below). 
 - tempvar - Stored in the allocation pointer, can be [revoked](https://www.cairo-lang.org/docs/how_cairo_works/consts.html#revoked-references) by certain function calls. 

 ## P5 - Revoked References

We know that there are 3 kinds of pointers to memory in an Cairo program - 

- `fp`: points to the start of the current executable function 
- `ap`: points to the first slot of free memory in the program 
- `pc`: points to the current executable instruction in the trace

Any variable which is referenced with respect to the `ap` is hence dependent on the compiler being able to remember it's relative position with respect to the pointer. 

However, since the `ap` can change if jump to a different function or part of the program, we may just lose the value stored in a particular slot forever. Read more [here](https://www.cairo-lang.org/docs/how_cairo_works/consts.html). 

To solve this, we have to protect variables which are prone to being called across multiple functions from revocation. One such strategy is to reference the variable with respect to the frame pointer using the `local` keyword.  

For instance, 
```
alloc_locals
let (local x) = foo(10)
```
we are basically calling `foo(10)`, then setting that to x and making x a local variable so it [cannot be revoked](https://stackoverflow.com/questions/71738301/what-does-it-mean-to-declare-a-local-variable-inside-a-let?rq=1) across multiple function calls. 
