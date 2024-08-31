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
