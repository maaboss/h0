Part (c) – Binary Modification to Invert Password Acceptance
Objective

The goal of this task was to modify the compiled passtr program without access to the original source code so that the program accepts all passwords except the correct one. This required reversing the program’s logic at the binary level using static analysis and patching.

Initial Analysis

The program was analyzed using Ghidra. The main function performs the following steps:

Prompts the user for a password.

Reads the password using scanf("%s", buffer).

Verifies the password using a loop:

Each character is XORed with a constant value (0x1f).

The result is compared against a hardcoded byte array.

A variable named local_c is used as a validation flag:

local_c = 1 → password is valid

local_c = 0 → password is invalid

After the loop, two checks are performed:

Whether local_c != 0

Whether strlen(password) == 16

If both checks succeed, the program prints the success message and the flag.
Otherwise, it prints “Sorry, no bonus.”

Understanding the Key Control Flow
Password correctness flag

At the end of the validation loop, the following instructions decide whether the password is valid:

CMP dword ptr [RBP + local_c], 0x0
JZ  LAB_0010121b   ; failure


CMP sets processor flags based on the comparison.

JZ (Jump if Zero) is taken when local_c == 0.

This means:

Correct password → local_c = 1 → jump NOT taken → success

Wrong password → local_c = 0 → jump taken → failure

First Binary Patch: Inverting Password Validity

To make incorrect passwords accepted and correct ones rejected, the conditional jump was inverted.

Original instruction
JZ LAB_0010121b

Patched instruction
JNZ LAB_0010121b

Explanation

JZ (Jump if Zero): jumps when the comparison result is zero

JNZ (Jump if Not Zero): jumps when the comparison result is non-zero

By changing JZ to JNZ:

Correct password (local_c = 1) → jump taken → failure

Wrong password (local_c = 0) → jump not taken → continues execution

This reverses the program’s logic regarding password correctness.

Identifying the Length Check Issue

After applying the first patch, short or incorrect passwords were still rejected.

Further analysis showed an additional check:

CALL strlen
CMP  RAX, 0x10
JNZ  LAB_0010121b


This enforces a strict requirement that the password must be exactly 16 characters long.

As a result:

Even incorrect passwords were rejected if their length was not 16.

Second Binary Patch: Inverting the Length Check

To ensure all incorrect passwords are accepted, regardless of length, the length check was also inverted.

Original instruction
JNZ LAB_0010121b

Patched instruction
JZ LAB_0010121b

Explanation

Original behavior:

Length ≠ 16 → jump to failure

Patched behavior:

Length == 16 → jump to failure

Length ≠ 16 → success

This ensures that incorrect passwords of any length are accepted, while the correct 16-character password is rejected.

Final Behavior After Patching
Password Type	Result
Correct password (16 chars)	Rejected
Incorrect password (any length)	Accepted

This matches the task requirement exactly.

Summary of Modifications

Two conditional jumps were modified in the binary:

Address	Original	Patched
0x001011f6	JZ	JNZ
0x00101208	JNZ	JZ

Both modifications were performed using Ghidra’s Patch Instruction feature and exported as an Original File to preserve the executable format.

Conclusion

By statically analyzing the binary and understanding how conditional jumps (JZ and JNZ) depend on CPU flags set by CMP, it was possible to invert the password validation logic without modifying the source code. This demonstrates how small changes at the assembly level can significantly alter program behavior and highlights the importance of control-flow integrity in software security.

If you want, I can also:

shorten this to a 1-page version

add screenshots captions

or rewrite it to match a formal lab-report template
