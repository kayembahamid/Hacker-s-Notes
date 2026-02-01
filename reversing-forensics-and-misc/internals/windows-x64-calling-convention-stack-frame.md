# Windows x64 Calling Convention: Stack Frame

## Windows x64 Calling Convention: Stack Frame

When a function in a Windows x64 binary is called, the stack frame is used in the following manner:

* First four integer arguments are passed to RCX, RDX, R8 and R9 registers accordingly (green)
* Arguments 5, 6, and further are pushed on to the stack (blue)
* Return address to the caller's next instruction is pushed is found at RSP + 0x0 (yellow)
* Below return address (RSP + 0x0) 32 bytes are always allocated for RCD, RDX, R8 and R9, even if the callee uses less than 4 arguments
* Local variables and non-volatile registers are stored above the return address (red)
* RBP is not used for referencing local variables/function arguments (except for when functions use `alloca()`) as it used to be the case for X86. RSP is responsible for that, hence RSP value does not change throughout the function body (push and pop is only used for epilogue/prologue)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MCIa9pSPy_0kCY1QurB%2F-MCmJ-mF3kZ4qAsLpCRF%2Fimage.png?alt=media\&token=83e4b8d8-4bdb-43e6-a423-5e317cd10776)

As an example, let's take a look at the function `msv1_0.LsaInitializePackage` in Ghidra.\
Below shows how the first four arguments are stored in ECX (lower part of RCX), RDX, R8 and R9:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MCIa9pSPy_0kCY1QurB%2F-MCmJGmlZfVc2PA1qzOu%2Fimage.png?alt=media\&token=e4b4597f-d95a-42d8-a9dd-fee7e6b1b829)

### References

{% embed url="https://docs.microsoft.com/en-us/cpp/build/stack-usage?view=vs-2019" %}
