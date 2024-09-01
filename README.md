# rust-notes
Here is the analysis of the Rust features.
The Rust code representation in assembly.

## String literals
``` rust
let str = "Hello, World!";
```
``` assembly
lea	rax, [rip + .L__unnamed_2] ; calculate the pointer to the string literal
mov	qword ptr [rsp - 16], rax  ; move it to the stack
mov	qword ptr [rsp - 8], 13    ; move the string literal length to the stack
```

## Slices
``` rust
// we need explisitly specify the type because otherwice it will be `pointer to array`
let slice: &[u32] = &[1,2,3];
```
``` assembly
lea	rax, [rip + .L__unnamed_2] ; calculate the pointer
mov	qword ptr [rsp - 16], rax  ; move it to the stack
mov	qword ptr [rsp - 8], 3     ; move its length to the stack
```
To change the pointer of slice without functions from `std`
``` rust
slice = &slice[1..];
```
``` assembly
	mov	rsi, qword ptr [rsp]      ; the slice ptr
	mov	rdx, qword ptr [rsp + 8]  ; the slice length
	mov	qword ptr [rsp + 16], rsi
	mov	qword ptr [rsp + 24], rdx
	mov	qword ptr [rsp + 32], 1
	mov	edi, 1
	lea	rcx, [rip + .L__unnamed_5]
	call	<core::ops::range::RangeFrom<usize> as core::slice::index::SliceIndex<[T]>>::index
	mov	qword ptr [rsp], rax
	mov	qword ptr [rsp + 8], rdx
```

## Clojures

### The Clojure's capturing

When clojure captures a variable rust creates something like capturing buffer on the stack.
> [!NOTE]
> If the buffer's variable count is less then 3 rust takes them as seperate registers..
``` rust
let a = 10;
let b = 12;
let c = 13;

// Capturing buffer
let a_cap = &a;
let b_cap = &a;
let c_cap = &a;
```

When you pass clojure to a function you also pass and this buffer by pointer.
Here is a simple example of the capturing:

``` rust
let a = 12;
let b = 13;
let c = 14;
some_func(|x| x*a*b);
```
``` assembly
sub    rsp,0x38

mov    DWORD PTR [rsp+0x8],0xc  ; let a = 12;
mov    DWORD PTR [rsp+0xc],0xd  ; let b = 13;
mov    DWORD PTR [rsp+0x10],0xe ; let c = 14;

mov    DWORD PTR [rsp+0x14],0xc ; let a_copy = 12;
mov    DWORD PTR [rsp+0x18],0xd ; let b_copy = 13;
mov    DWORD PTR [rsp+0x1c],0xe ; let c_copy = 14;

lea    rax,[rsp+0x14]           ;
mov    QWORD PTR [rsp+0x20],rax ;
lea    rax,[rsp+0x18]           ; Creating the capturing buffer
mov    QWORD PTR [rsp+0x28],rax ;
lea    rax,[rsp+0x1c]           ;
mov    QWORD PTR [rsp+0x30],rax ;

lea    rdi,[rsp+0x20]           ; pass the buffer
call   <main::some_func>        ; call some func

add    rsp,0x38
ret
```
