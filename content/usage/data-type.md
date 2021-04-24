---
title: Data Types
weight: 1
---

| Name | Usage     | Description                                   |
| -----| ----------|-----------------------------------------------|
| UInt | UInt(5.W) | Unsigned Integer with 5-bit width             |
| SInt | SInt(3.W) | Signed Integer with 3-bit width               |
| Bool | Bool()    | 1-bit unsigned integer, the same as UInt(1.W) |

## Literal 

Constant or literal values are expressed using Scala integers or strings passed to constructors for the types:

```scala
class LitValue1 extends RawModule {
  val out1 = IO(Output(UInt(5.W)))
  val out2 = IO(Output(UInt(4.W)))
  val out3 = IO(Output(UInt(3.W)))
  val out4 = IO(Output(SInt(4.W)))
  val out5 = IO(Output(SInt(4.W)))
  val out6 = IO(Output(SInt(32.W)))
  val out7 = IO(Output(UInt(4.W)))
  val out8 = IO(Output(UInt(4.W)))
  val out9 = IO(Output(UInt(4.W)))
  val outB0 = IO(Output(Bool()))
  val outB1 = IO(Output(Bool()))
  val outB2 = IO(Output(Bool()))
  val outB3 = IO(Output(Bool()))

  /* inferred width according to LHS */
  out1 := 1.U
  out3 := 5.U          // unsigned 3-bit integer
  out4 := 5.S          // signed 4-bit integer
  out5 := -8.S         // negative decimal 4-bit literal
  /* specified width */
  out2 := 8.U(4.W)
  out6 := -152.S(32.W) // 32-bit signed decimal, value -152
  /* string literals */
  out7 := "ha".U       // hexadecimal 4-bit
  out8 := "o12".U      // octal 4-bit lit from string
  out9 := "b1010".U    // binary 4-bit lit from string
  /* Boolean */
  outB0 := true.B
  outB1 := false.B
  outB2 := 1.B
  outB3 := 0.B
}
```

Generated verilog as below:

```verilog
module LitValue1(
  output [4:0]  out1,
  output [3:0]  out2,
  output [2:0]  out3,
  output [3:0]  out4,
  output [3:0]  out5,
  output [31:0] out6,
  output [3:0]  out7,
  output [3:0]  out8,
  output [3:0]  out9,
  output        outB0,
  output        outB1,
  output        outB2,
  output        outB3
);
  assign out1 = 5'h1;
  assign out2 = 4'h8;
  assign out3 = 3'h5;
  assign out4 = 4'sh5;
  assign out5 = -4'sh8;
  assign out6 = -32'sh98;
  assign out7 = 4'ha;
  assign out8 = 4'ha;
  assign out9 = 4'ha;
  assign outB0 = 1'h1;
  assign outB1 = 1'h0;
  assign outB2 = 1'h1;
  assign outB3 = 1'h0;
endmodule
```

The `.asSInt` and `.asUInt` for literal are the same as `.S` and `.U`.

Underscores can be used as separators in long string literals to aid readability, but are ignored when creating the value.

```scala
class LitValue2 extends RawModule {
  val out1 = IO(Output(UInt(32.W)))
  val out2 = IO(Output(UInt(8.W)))
  val out3 = IO(Output(UInt(6.W)))
  val out4 = IO(Output(UInt(12.W)))
  val out5 = IO(Output(SInt(7.W)))
  val out6 = IO(Output(UInt(8.W)))

  out1 := "h_dead_beef".U      // Underscores for readability

  out2 := "ha".asUInt(8.W)     // hexadecimal 8-bit lit of type UInt
  out3 := "o12".asUInt(6.W)    // octal 6-bit lit of type UInt
  out4 := "b1010".asUInt(12.W) // binary 12-bit lit of type UInt

  out5 := 5.asSInt(7.W)        // signed decimal 7-bit lit of type SInt
  out6 := 5.asUInt(8.W)        // unsigned decimal 8-bit lit of type UInt
}
```

Generated verilog as below:

```verilog
module LitValue2(
  output [31:0] out1,
  output [7:0]  out2,
  output [5:0]  out3,
  output [11:0] out4,
  output [6:0]  out5,
  output [7:0]  out6
);
  assign out1 = 32'hdeadbeef;
  assign out2 = 8'ha;
  assign out3 = 6'ha;
  assign out4 = 12'ha;
  assign out5 = 7'sh5;
  assign out6 = 8'h5;
endmodule
```

`asSInt` and `asUInt` used for casting.

```scala
class Casting extends RawModule {
  val out0 = IO(Output(SInt(3.W)))
  val out1 = IO(Output(UInt(3.W)))
  val out2 = IO(Output(UInt(3.W)))

  out0 := 7.U.asSInt.asUInt.asSInt.asSInt
  out1 := -1.S(3.W).asUInt
  out2 := -1.S.asUInt
}
```

Generated verilog as below:

```verilog
module Casting(
  output [2:0] out0,
  output [2:0] out1,
  output [2:0] out2
);
  assign out0 = -3'sh1;
  assign out1 = 3'h7;
  assign out2 = 3'h1;
endmodule
```
