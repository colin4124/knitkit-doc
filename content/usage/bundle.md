---
title: Bundle
weight: 4
---

**Bundle** is `Map`-like data structure for knitkit ground type, the name of value in generated verilog file is the key name by default, but you can use suggestName change it. 

For example:

```scala
  val io = IO(Bundle(
    "sign"     -> Output(Bool()),
    "exponent" -> Bundle(
      "1" -> Output(UInt(8.W)),
      "2" -> Output(UInt(8.W)),
    ),
    "significand" -> Output(UInt(23.W)),
  ))
```

Naming result should be:

```
  io_sign        = Output(Bool())
  io_exponent_1  = Output(UInt( 8.W))
  io_exponent_2  = Output(UInt( 8.W))
  io_significand = Output(UInt(23.W))
```

If use `suggestName`, for example:

```scala
  val io = IO(Bundle(
    "sign"     -> Output(Bool()).suggestName("foo"),
    "exponent" -> Bundle(
      "1" -> Output(UInt( 8.W)),
      "2" -> Output(UInt( 8.W)),
    ).suggestName("bar"),
    "significand" -> Output(UInt(23.W)),
  ))
```

`suggestName` only rename one key-level, not the whole. Naming result should be:

```
  io_foo         = Output(Bool())
  io_bar_1       = Output(UInt( 8.W))
  io_bar_2       = Output(UInt( 8.W))
  io_significand = Output(UInt(23.W))
```

A complete example as below:

```scala
class BundleCase extends RawModule {
  val io = IO(Bundle(
    "sign"     -> Output(Bool()).suggestName("foo"),
    "exponent" -> Bundle(
      "1" -> Output(UInt( 8.W)),
      "2" -> Output(UInt( 8.W)),
    ).suggestName("bar"),
    "significand" -> Output(UInt(23.W)),
  ))

  val my_float = Bundle(
   "sign"        -> Bool(),
   "exponent"    -> UInt(8.W),
   "significand" -> UInt(23.W),
  )
  // Floating point constant.
  val floatConst = Wire(my_float)
  floatConst("sign"       ) := true.B
  floatConst("exponent"   ) := 10.U
  floatConst("significand") := 128.U

  io("sign"       )      := floatConst("sign")
  io("exponent"   )("1") := floatConst("exponent")
  io("exponent"   )("2") := floatConst("exponent")
  io("significand")      := floatConst("significand")
}
```

Generated verilog as below:

```verilog
module BundleCase(
  output        io_foo,
  output [7:0]  io_bar_1,
  output [7:0]  io_bar_2,
  output [22:0] io_significand
);
  wire  floatConst_sign;
  wire [7:0] floatConst_exponent;
  wire [22:0] floatConst_significand;
  assign floatConst_sign = 1'h1;
  assign floatConst_exponent = 8'ha;
  assign floatConst_significand = 23'h80;
  assign io_foo = floatConst_sign;
  assign io_bar_1 = floatConst_exponent;
  assign io_bar_2 = floatConst_exponent;
  assign io_significand = floatConst_significand;
endmodule
```

Use `asUInt` convert **Bundle** to `UInt`, the first order in bundle as the Most Significant Bit.

```scala
package example
package bundle

import knitkit._

class AsUIntCase extends RawModule {
  val out = IO(Output(UInt(32.W)))

  val bundle = Wire(Bundle(
    "foo" -> UInt(16.W),
    "bar" -> UInt(16.W),
  ))

  bundle("foo") := 0x1234.U
  bundle("bar") := 0x5678.U

  out := bundle.asUInt
}
```

Generated verilog as below:

```verilog
module AsUIntCase(
  output [31:0] out
);
  wire [15:0] bundle_foo;
  wire [15:0] bundle_bar;
  assign bundle_foo = 16'h1234;
  assign bundle_bar = 16'h5678;
  assign out = {bundle_foo, bundle_bar};
endmodule
```

Use `<>` to connect `Bundle`

```scala
class PacketCase extends RawModule {
  val packet = Seq(
    "header" -> UInt(16.W),
    "addr"   -> UInt(16.W),
    "data"   -> UInt(32.W),
    )
  val io = IO(Bundle(
    "inPacket"  -> Input (Bundle(packet)),
    "outPacket" -> Output(Bundle(packet)),
  ))

  val reg = Wire(Bundle(packet))
  reg <> io("inPacket")
  reg <> io("outPacket")

  io("inPacket") <> io("outPacket")
}
```

Generated verilog as below:

```verilog
module PacketCase(input  [15:0] io_inPacket_header,
  input  [15:0] io_inPacket_addr,
  input  [31:0] io_inPacket_data,
  output [15:0] io_outPacket_header,
  output [15:0] io_outPacket_addr,
  output [31:0] io_outPacket_data
);
  wire [15:0] reg_header;
  wire [15:0] reg_addr;
  wire [31:0] reg_data;
  assign reg_header = io_inPacket_header;
  assign reg_addr = io_inPacket_addr;
  assign reg_data = io_inPacket_data;
  assign io_outPacket_header = reg_header;
  assign io_outPacket_addr = reg_addr;
  assign io_outPacket_data = reg_data;
  assign io_outPacket_header = io_inPacket_header;
  assign io_outPacket_addr = io_inPacket_addr;
  assign io_outPacket_data = io_inPacket_data;
endmodule
```

TODO: Flipping Bundles

```scala
class ABBundle extends Bundle {
  val a = Input(Bool())
  val b = Output(Bool())
}
class MyFlippedModule extends RawModule {
  // Normal instantiation of the bundle
  // 'a' is an Input and 'b' is an Output
  val normalBundle = IO(new ABBundle)
  normalBundle.b := normalBundle.a

  // Flipped recursively flips the direction of all Bundle fields
  // Now 'a' is an Output and 'b' is an Input
  val flippedBundle = IO(Flipped(new ABBundle))
  flippedBundle.a := flippedBundle.b
}
```

Generated verilog as below:

```verilog
module MyFlippedModule(
  input   normalBundle_a,
  output  normalBundle_b,
  output  flippedBundle_a,
  input   flippedBundle_b
);
  assign normalBundle_b = normalBundle_a; // @[bundles-and-vecs.md 61:18]
  assign flippedBundle_a = flippedBundle_b; // @[bundles-and-vecs.md 66:19]
endmodule
```
