---
title: Module Hierarchy
weight: 7
---

{{< toc >}}

Module's declaration and submodule instantiation are two steps.

If defines a module `Mux2`, use `val mux2_1 = Module(new Mux2(1))` as declaration, `val m0 = mux2_1()` to instantiate submodule.

Use map-like's method to reference submodule's port. `val sel` defines in module `Mux2`, use `m0("sel")` to reference port `sel`.

```scala
class Mux2(width: Int) extends RawModule {
  override def desiredName = s"Mux2_${width}"

  val sel = IO(Input(UInt(1.W)))
  val in0 = IO(Input(UInt(width.W)))
  val in1 = IO(Input(UInt(width.W)))
  val out = IO(Output(UInt(width.W)))
  out := Mux(sel, in1, in0)
}

class Mux4 extends RawModule {
  val in0 = IO(Input (UInt(1.W)))
  val in1 = IO(Input (UInt(1.W)))
  val in2 = IO(Input (UInt(1.W)))
  val in3 = IO(Input (UInt(1.W)))
  val sel = IO(Input (UInt(2.W)))
  val out = IO(Output(UInt(2.W)))
  val mux2_1 = Module(new Mux2(1))
  val mux2_2 = Module(new Mux2(2))
  val m0 = mux2_1()
  m0("sel") := sel(0)
  m0("in0") := in0
  m0("in1") := in1
  val m1 = mux2_1()
  m1("sel") := sel(0)
  m1("in0") := in2
  m1("in1") := in3
  val m2 = mux2_2()
  m2("sel") := sel(1)

  val m2_in0 = WireInit(Cat(m0("out"), m1("out")))
  m2("in0") := m2_in0
  m2("in1") := Cat(m1("out"), m0("out"))

  out := m2("out")
}
```

Generated verilog as below:

```verilog
module WireCase(
  output [7:0] out1,
  output [7:0] out2
);
  wire [7:0] my_node;
  wire [7:0] init_node;
  assign my_node = 8'ha;
  assign init_node = 8'ha;
  assign out1 = my_node;
  assign out2 = init_node;
endmodule
```

```scala
module Mux2_1(
  input   sel,
  input   in0,
  input   in1,
  output  out
);
  assign out = sel ? in1 : in0;
endmodule
module Mux2_2(
  input        sel,
  input  [1:0] in0,
  input  [1:0] in1,
  output [1:0] out
);
  assign out = sel ? in1 : in0;
endmodule
module Mux4(
  input        in0,
  input        in1,
  input        in2,
  input        in3,
  input  [1:0] sel,
  output [1:0] out
);
  wire  m0_out;
  wire  m1_out;
  wire [1:0] m2_in0;
  Mux2_1 m0 (
    .in0 (in0   ),
    .in1 (in1   ),
    .out (m0_out),
    .sel (sel[0])
  );
  Mux2_1 m1 (
    .in0 (in2   ),
    .in1 (in3   ),
    .out (m1_out),
    .sel (sel[0])
  );
  Mux2_2 m2 (
    .in0 (m2_in0          ),
    .in1 ({m1_out, m0_out}),
    .out (out             ),
    .sel (sel[1]          )
  );
  assign m2_in0 = {m0_out, m1_out};
endmodule
```
