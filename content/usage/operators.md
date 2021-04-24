---
title: Operators
weight: 2
---

## Bitwise operators

| Operation                             | Explanation |
| ------------------------------------- | ----------- |
| `val invertedX = ~x`                  | Bitwise NOT |
| `val hiBits    = x & "h_ffff_0000".U` | Bitwise AND |
| `val OROut     = flagsIn | overflow`  | Bitwise OR  |
| `val XOROut    = flagsIn ^ toggle`    | Bitwise XOR |

```scala
class Bitwise_Ops extends RawModule {
  val x         = IO(Input(UInt(32.W)))
  val invertedX = IO(Output(UInt(32.W)))
  val hiBits    = IO(Output(UInt(32.W)))
  val OROut     = IO(Output(UInt(32.W)))
  val XOROut    = IO(Output(UInt(32.W)))

  invertedX := ~x                   // Bitwise NOT
  hiBits    :=  x & "h_ffff_0000".U // Bitwise AND
  OROut     := invertedX | hiBits   // Bitwise OR
  XOROut    := invertedX ^ hiBits   // Bitwise XOR
}
```

Generated verilog as below:

```verilog
module Bitwise_Ops(
  input  [31:0] x,
  output [31:0] invertedX,
  output [31:0] hiBits,
  output [31:0] OROut,
  output [31:0] XOROut
);
  assign invertedX = ~ x;
  assign hiBits = x & 32'hffff0000;
  assign OROut = invertedX | hiBits;
  assign XOROut = invertedX ^ hiBits;
endmodule
```
## Bitwise reductions

| Operation             | Explanation  |
| --------------------- | ------------ |
| `val allSet = x.andR` | AND reduction|
| `val anySet = x.orR`  | OR reduction |
| `val parity = x.xorR` | XOR reduction|

```scala
class Bitwise_Reductions extends RawModule {
  val x      = IO(Input(UInt(32.W)))
  val allSet = IO(Output(Bool()))
  val anySet = IO(Output(Bool()))
  val parity = IO(Output(Bool()))

  allSet := x.andR
  anySet := x.orR
  parity := x.xorR
}
```

Generated verilog as below:

```verilog
module Bitwise_Reductions(
  input  [31:0] x,
  output        allSet,
  output        anySet,
  output        parity
);
  assign allSet = x == 32'hffffffff;
  assign anySet = x != 1'h0;
  assign parity = ^x;
endmodule
```

## Equality comparison

| Operation           | Explanation  |
| ------------------- | ------------ |
| `val equ = x === y` | Equality     |
| `val neq = x =/= y` | Inequality   |

```scala
class Equality_Comparison extends RawModule {
  val x = IO(Input(UInt(5.W)))
  val y = IO(Input(UInt(4.W)))
  val equ = IO(Output(Bool()))
  val neq = IO(Output(Bool()))

  equ := x === y
  neq := x =/= y
}
```

Generated verilog as below:

```verilog
module Equality_Comparison(
  input  [4:0] x,
  input  [3:0] y,
  output       equ,
  output       neq
);
  assign equ = x == y;
  assign neq = x != y;
endmodule
```

## Shifts

| Operation                  | Explanation                                           |
| -------------------------- | ----------------------------------------------------- |
| `val twoToTheX = 1.S << x` | Logical shift left                                    |
| `val hiBits = x >> 16.U`   | Right shift (logical on UInt and arithmetic on SInt). |

TODO: arithmetic on SInt
```scala
class Shifts extends RawModule {
  val x         = IO(Input(UInt(3.W)))
  val y         = IO(Input(UInt(32.W)))
  val twoToTheX = IO(Output(SInt(33.W)))
  val hiBits    = IO(Output(UInt(16.W)))
  twoToTheX := (1.S << x)
  hiBits    := y >> 16.U
}
```

Generated verilog as below:

```verilog
module Shifts(
  input  [2:0]  x,
  input  [31:0] y,
  output [32:0] twoToTheX,
  output [15:0] hiBits
);
  assign twoToTheX = $signed(2'sh1) << x;
  assign hiBits = y >> 5'h10;
endmodule
```

## Bitfield manipulation

| Operation                                   | Explanation                                           |
| ------------------------------------------- | ----------------------------------------------------- |
| `val xLSB = x(0)`                           | Extract single bit, LSB has index 0.                  |
| `val xTopNibble = x(15, 12)`                | Extract bit field from end to start bit position.     |
| `val usDebt = Fill(3, "hA".U)`              | Replicate a bit string multiple times.                |
| `val float = Cat(sign, exponent, mantissa)` | Concatenates bit fields, with first argument on left. |

```scala
class Bitfield_Manipulation extends RawModule {
  val sel        = IO(Input(UInt(4.W)))
  val x          = IO(Input(UInt(16.W)))
  val dynamicSel = IO(Output(Bool()))
  val xLSB       = IO(Output(Bool()))
  val xTopNibble = IO(Output(UInt(4.W)))
  val usDebt     = IO(Output(UInt(12.W)))
  val float      = IO(Output(UInt(17.W)))

  dynamicSel := x(sel, "foo") // foo used for naming intermediate variable.
  xLSB       := x(0)
  xTopNibble := x(15, 12)
  usDebt     := Fill(3, "hA".U)
  float      := Cat(xLSB, xTopNibble, usDebt)
}
```

Generated verilog as below:

```verilog
module Bitfield_Manipulation(
  input  [3:0]  sel,
  input  [15:0] x,
  output        dynamicSel,
  output        xLSB,
  output [3:0]  xTopNibble,
  output [11:0] usDebt,
  output [16:0] float
);
  wire [15:0] foo;
  assign foo = x >> sel;
  assign dynamicSel = foo[0];
  assign xLSB = x[0];
  assign xTopNibble = x[15:12];
  assign usDebt = {4'ha, 4'ha, 4'ha};
  assign float = {xLSB, xTopNibble, usDebt};
endmodule
```

## Logical Operations

| Operation                                   | Explanation                       |
| ------------------------------------------- | --------------------------------- |
| `val sleep = !busy`                         | Logical NOT                       |
| `val hit = tagMatch && valid`               | Logical AND                       |
| `val stall = src1busy || src2busy`          | Logical OR                        |
| `val out = Mux(sel, inTrue, inFalse)`       | Two-input mux where sel is a Bool |

```scala
class Logical_Operations extends RawModule {
  val sel   = IO(Input(Bool()))
  val a     = IO(Input(Bool()))
  val b     = IO(Input(Bool()))
  val sleep = IO(Output(Bool()))
  val hit   = IO(Output(Bool()))
  val stall = IO(Output(Bool()))
  val out   = IO(Output(Bool()))

  sleep := !a
  hit   := a && b
  stall := a || b
  out   := Mux(sel, a, b)
}
```

Generated verilog as below:

```verilog
module Logical_Operations(
  input   sel,
  input   a,
  input   b,
  output  sleep,
  output  hit,
  output  stall,
  output  out
);
  assign sleep = a == 1'h0;
  assign hit = a & b;
  assign stall = a | b;
  assign out = sel ? a : b;
endmodule
```

## Arithmetic operations

| Operation                                   | Explanation                          |
| ------------------------------------------- | ------------------------------------ |
| `val sum = a + b` *or* `val sum = a +% b`   | Addition (without width expansion)   |
| `val diff = a - b` *or* `val diff = a -% b` | Subtraction (without width expansion)|
| `val prod = a * b`                          | Multiplication                       |
| `val div = a / b`                           | Division                             |
| `val mod = a % b`                           | Modulus                              |

```scala
class Arithmetic_Operations extends RawModule {
  val a = IO(Input(UInt(8.W)))
  val b = IO(Input(UInt(8.W)))
  val sum  = IO(Output(UInt(8.W)))
  val diff0 = IO(Output(UInt(9.W)))
  val diff1 = IO(Output(UInt(9.W)))
  val diff2 = IO(Output(UInt(9.W)))
  val prod  = IO(Output(UInt(16.W)))
  val div   = IO(Output(UInt(8.W)))
  val mod   = IO(Output(UInt(8.W)))

  sum   := a + b
  diff0 := a - b
  prod  := a * b
  div   := a / b
  mod   := a % b
}
```

Generated verilog as below:

```verilog
module Arithmetic_Operations(
  input  [7:0]  a,
  input  [7:0]  b,
  output [7:0]  sum,
  output [8:0]  diff0,
  output [8:0]  diff1,
  output [8:0]  diff2,
  output [15:0] prod,
  output [7:0]  div,
  output [7:0]  mod
);
  assign sum = a + b;
  assign diff0 = a - b;
  assign prod = a * b;
  assign div = a / b;
  assign mod = a % b;
endmodule
```

## Arithmetic comparisons

| Operation          | Explanation           |
| ------------------ | ----------------------|
| `val gt = a > b`   | Greater than          |
| `val gte = a >= b` | Greater than or equal |
| `val lt = a < b`   | Less than             |
| `val lte = a <= b` | Less than or equal    |

```scala
class Arithmetic_Comparisons extends RawModule {
  val a = IO(Input(UInt(8.W)))
  val b = IO(Input(UInt(8.W)))
  val gt  = IO(Output(Bool()))
  val gte = IO(Output(Bool()))
  val lt  = IO(Output(Bool()))
  val lte = IO(Output(Bool()))

  gt  := a > b
  gte := a >= b
  lt  := a < b
  lte := a <= b
}
```

Generated verilog as below:

```verilog
module Arithmetic_Comparisons(
  input  [7:0] a,
  input  [7:0] b,
  output       gt,
  output       gte,
  output       lt,
  output       lte
);
  assign gt = a > b;
  assign gte = a >= b;
  assign lt = a < b;
  assign lte = a <= b;
endmodule
```
