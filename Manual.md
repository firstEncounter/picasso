# picasso Manual

## Basic concepts

Comments are introduced by the semicolon character. E.g.

```
; This is a comment
.fvec myFloat ; They can also appear in the same line
```

Identifiers follow the same rules as C identifiers.

Labels consist of an identifier plus a colon. E.g.

```
myLabel:
	mov r0, r1
```

Procedures are delimited using the `.proc` and `.end` directives. E.g.

```
.proc normalize3
	dp4 r15, r8, r8
	rsq r15, r15
	mul r8, r15, r8
.end
```

A valid PICA200 shader must contain a `main` procedure.

Instructions consist of an opcode name and a comma-delimited list of arguments.

Directives are special statements that start with a period and control certain aspects of `picasso`'s code emission; such as defining procedures, uniforms, constants and more.

PICA200 registers are often used as arguments to instructions. There exist the following registers:

- `o0` through `o7`: Output registers (usable as a destination operand).
- `v0` through `v7`: Input registers (usable as a source operand).
- `r0` through `r15`: Scratch registers (usable as both destination and source operands).
- `c0` through `c95`: Floating-point vector uniforms (usable as a special type of source operand called SRC1).
- `i0` through `i3`: Integer vector uniforms (special purpose).
- `b0` through `b15`: Boolean uniforms (special purpose).

All registers contain 32-bit floating point vectors; except for integer vector uniforms (containing 8-bit integers) and boolean uniforms. Vectors have 4 components: x, y, z and w. Uniforms are special registers that are writable by the CPU; thus they are used to pass configuration parameters to the shader such as transformation matrices. Sometimes they are preloaded with constant values that may be used in the logic of the shader.

In most situations, vectors may be [swizzled](http://en.wikipedia.org/wiki/Swizzling_%28computer_graphics%29), that is; their components may be rearranged. Register arguments support specifying a swizzling mask: `r0.wwxy`. The swizzling mask usually has 4 components (but not more), if it has less the last component is repeated to fill the mask. The default mask applied to registers is `xyzw`; that is, identity (no effect).

Output parameters have an output mask instead of a swizzling mask. This allows the shader to write to some components of a register without affecting the others. In `picasso`, the output mask is parsed exactly the same way as the swizzling mask, enabling write access for the components that are used in it. By default it is also `xyzw`; that is, writing to all components.

Registers may also be assigned additional names in order to make the code more legible. These additional names are called aliases. Aliases may also contain a swizzling mask; if a swizzling mask is applied to an alias afterwards the masks are combined. For example, provided that `someAlias` is an alias for `c0.wyxz`, `someAlias.xxww` would be equivalent to `c0.wwzz`. Aliases may be created by several directives which reserve certain kinds of registers.

For convenience, registers may be addressed using an offset from a known register. This is called indexing. For example, `c8[4]` is equivalent to `c12`; and `r4[-2]` is equivalent to `r2`. Indexing is useful for addressing arrays of registers (such as matrices).

Some source operands of instructions (called SRC1) support relative addressing. This means that it is possible to use one of the three built-in indexing registers (`a0`, `a1` and `a2` aka `lcnt`) to address a register, e.g. `someArray[lcnt]`. Adding an offset is also supported, e.g. `someArray[lcnt+2]`. This is useful in FOR loops.

Normal floating-point vector registers may also be negated by prepending a minus sign before it, e.g. `-r2` or `-someArray[lcnt+2]`.

## Supported Directives

### .proc
```
.proc procName
```
Introduces a procedure called `procName`. The procedure is terminated with `.end`.

### .else
```
.else
```
Introduces the ELSE section of an IF statement.

### .end
```
.end
```
Terminates a procedure, an IF statement or a FOR statement.

### .alias
```
.alias aliasName register
```
Creates a new alias for `register` called `aliasName`. The specified register may also have a swizzling mask.

### .fvec
```
.fvec unifName1, unifName2[size], unifName3, ...
```
Allocates new floating-point vector uniforms (or arrays of uniforms) and creates aliases for them that point to the allocated registers. Example:

```
.fvec scaler
.fvec projMatrix[4], modelViewMatrix[4]
```

### .ivec
```
.ivec unifName1, unifName2[size], unifName3, ...
```
Allocates new integer vector uniforms (or arrays of uniforms) and creates aliases for them that point to the allocated registers.

### .bool
```
.bool unifName1, unifName2[size], unifName3, ...
```
Allocates new boolean uniforms (or arrays of uniforms) and creates aliases for them that point to the allocated registers. Example:

```
.bool useLight[4]
.bool useRawVertexColor
```

### .constf
```
.constf constName(x, y, z, w)
```
Reserves a new floating-point vector uniform to be preloaded with the specified constant; creates an alias for it that points to the allocated register. Example:

```
.constf floatConsts(0.0, 1.0, -1.0, 3.14159)
```

### .consti
```
.consti constName(x, y, z, w)
```
Reserves a new integer vector uniform to be preloaded with the specified constant; creates an alias for it that points to the allocated register. Example:

```
.constf loopParams(16, 0, 1, 0)
```

### .out
```
.out outName propName
```
Allocates a new output register, wires it to a certain output property and creates an alias for it that points to the allocated register. The following property names are supported:

- `position` (or `pos`): Represents the position of the outputted vertex.
- `normalquat` (or `nquat`): Under investigation.
- `color` (or `clr`): Represents the color of the outputted vertex. Its format is (R, G, B, xx) where R,G,B are values ranging from 0.0 to 1.0. The W component isn't used.
- `texcoord0` (or `tcoord0`): Represents the texture coordinate that is fed to the Texture Unit 0. The Z and W components are not used.
- `texcoord0w` (or `tcoord0w`): Under investigation.
- `texcoord1` (or `tcoord1`): As `texcoord0`, but for the Texture Unit 1.
- `texcoord2` (or `tcoord2`): As `texcoord0`, but for the Texture Unit 2.
- `7`: Under investigation.
- `view`: Under investigation.

Example:

```
.out outPos position
.out outClr color
.out outTex texcoord0
```

## Supported Instructions

See [Shader Instruction Set](http://3dbrew.org/wiki/Shader_Instruction_Set) for more details.

Syntax                            | Description
--------------------------------- | -----------------------------------
`nop`                             | No operation.
`end`                             | Signals the end of the program.
`emit`                            | (Geoshader-only) Emits a vertex configured by a prior `setemit`.
`setemit vtxId, emitFlags`        | (Geoshader-only) Configures a vertex for emission. The `emitFlags` parameter can be omitted.
`add rDest, rSrc1, rSrc2`         |
`dp3 rDest, rSrc1, rSrc2`         |
`dp4 rDest, rSrc1, rSrc2`         |
`dph rDest, rSrc1, rSrc2`         |
`mul rDest, rSrc1, rSrc2`         |
`sge rDest, rSrc1, rSrc2`         |
`slt rDest, rSrc1, rSrc2`         |
`max rDest, rSrc1, rSrc2`         |
`min rDest, rSrc1, rSrc2`         |
`dphi rDest, rSrc2, rSrc1`        |
`sgei rDest, rSrc2, rSrc1`        |
`slti rDest, rSrc2, rSrc1`        |
`ex2 rDest, rSrc1`                |
`lg2 rDest, rSrc1`                |
`ex2 rDest, rSrc1`                |
`flr rDest, rSrc1`                |
`rcp rDest, rSrc1`                |
`rsq rDest, rSrc1`                |
`mov rDest, rSrc1`                |
`mova rSrc1`                      |
`cmp rSrc1, opx, opy, rSrc2`      |
`call procName`                   |
`for iReg`                        |
`breakc condExp`                  |
`callc condExp, procName`         |
`ifc condExp`                     |
`jmpc condExp, labelName`         |
`callu bReg, procName`            |
`ifu bReg`                        |
`jmpu bReg, labelName`            |
`madi rDest, rSrc1, rSrc2, rSrc1` |
`mad rDest, rSrc1, rSrc1, rSrc2`  |

### Description of operands

- `rDest`: Represents a destination operand (register).
- `rSrc1`: Represents a so-called SRC1 source operand (register), which allows accessing floating-point vector uniforms and relative addressing.
- `rSrc2`: Represents a so-called SRC2 source operand (register), which is limited to input and scratch registers.
- `iReg`: Represents an integer vector uniform source operand.
- `bReg`: Represents a boolean uniform source operand.
- `procName`: Represents the name of a procedure.
- `labelName`: Represents the name of a label.
- `opx` and `opy`: They represent a conditional operator that is applied to the source registers and whose result is stored in the appropriate flag (`cmp.x` and `cmp.y` respectively). Supported values include:
	- `eq`: Equal
	- `ne`: Not equal
	- `lt`: Less than
	- `le`: Less or equal than
	- `gt`: Greater than
	- `ge`: Greater or equal than
	- `6` and `7`: currently unknown, supposedly the result they yield is always true.
- `condExp`: Represents a conditional expression, which uses the conditional flags `cmp.x` and `cmp.y` set by the CMP instruction. These flags may be negated using the `!` symbol, e.g. `!cmp.x`. The conditional expression can take any of the following forms:
	- `flag1`: It tests a single flag.
	- `flag1 && flag2`: It performs AND between the two flags. Optionally, a single `&` may be specified.
	- `flag1 || flag2`: It performs OR between the two flags. Optionally, a single `|` may be specified.
- `vtxId`: An integer ranging from 0 to 3 specifying the vertex ID used in geoshader vertex emission.
- `emitFlags`: A space delimited combination of the following words:
	- `primitive` (or `prim`): Specifies that after emitting the vertex, a primitive should also be emitted.
	- `inv` (or `invert`): Specifies that the order of the vertices in the emitted primitive is inverted.
