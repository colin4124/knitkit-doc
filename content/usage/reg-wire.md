---
title: Combinational and Sequential Circuits
weight: 5
---

{{< toc >}}

## Wire

Wires as hardware nodes to which one can assign values or connect other nodes.

```scala
class WireCase extends RawModule {
  val out1 = IO(Output(UInt(8.W)))
  val out2 = IO(Output(UInt(8.W)))

  val my_node = Wire(UInt(8.W))
  my_node := 10.U

  val init_node = WireInit(10.U(8.W))

  out1 := my_node
  out2 := init_node
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

## Register

### Reg Example

Register with Clock.

```scala
class RegCase extends RawModule {
  val clk = IO(Input(Clock()))
  val in  = IO(Input(UInt(3.W)))
  val out = IO(Output(UInt(3.W)))

  val delayReg = withClock(clk) { Reg(UInt(3.W)) }

  delayReg := in
  out := delayReg
}
```

Generated verilog as below:

```verilog
module RegCase(
  input        clk,
  input  [2:0] in,
  output [2:0] out
);
  reg  [2:0] delayReg;
  assign out = delayReg;
  always @(posedge clk) begin
    delayReg <= in;
  end
endmodule
```

### Register Width Inference Example

Register unspecified widths will be inferred by the value assigned to it.

```scala
class RegInferredCase extends RawModule {
  val clk = IO(Input(Clock()))
  val rst = IO(Input(Reset()))
  val in  = IO(Input(UInt(3.W)))
  val out = IO(Output(UInt(3.W)))

  val delayReg = withClock(clk) { Reg(UInt()) }

  delayReg := in
  out := delayReg
}
```

Generated verilog as below:

```verilog
module RegInferredCase(
  input        clk,
  input        rst,
  input  [2:0] in,
  output [2:0] out
);
  reg  [2:0] delayReg;
  assign out = delayReg;
  always @(posedge clk) begin
    delayReg => in;
  end
endmodule
```


Register with Clock and Synchronous Reset/AsyncNegReset/AsyncPosReset.

### Register Initialize Example

Register initialized depends on its reset's type: synchronous, asynchronous posedge/negedge.

```scala
class RegInitCase extends RawModule {
  val clk_1   = IO(Input(Clock()))
  val rst_1   = IO(Input(Reset()))
  val clk_2   = IO(Input(Clock()))
  val rst_2   = IO(Input(AsyncNegReset()))
  val clk_3   = IO(Input(Clock()))
  val rst_3   = IO(Input(AsyncPosReset()))
  val foo     = IO(Input(UInt(3.W)))
  val bar     = IO(Input(UInt(3.W)))
  val cat     = IO(Input(UInt(3.W)))
  val foo_out = IO(Output(UInt(3.W)))
  val bar_out = IO(Output(UInt(3.W)))
  val cat_out = IO(Output(UInt(3.W)))

  val reg_foo = withClockAndReset(clk_1, rst_1) { RegInit(0.U(3.W)) }
  val reg_bar = withClockAndReset(clk_2, rst_2) { RegInit(0.U(3.W)) }
  val reg_cat = withClockAndReset(clk_3, rst_3) { RegInit(0.U(3.W)) }

  reg_foo := foo
  reg_bar := bar
  reg_cat := cat
  foo_out := reg_foo
  bar_out := reg_bar
  cat_out := reg_cat
}
```

Generated verilog as below:

```verilog
module RegInitCase(
  input        clk_1,
  input        rst_1,
  input        clk_2,
  input        rst_2,
  input        clk_3,
  input        rst_3,
  input  [2:0] foo,
  input  [2:0] bar,
  input  [2:0] cat,
  output [2:0] foo_out,
  output [2:0] bar_out,
  output [2:0] cat_out
);
  reg  [2:0] reg_foo;
  reg  [2:0] reg_bar;
  reg  [2:0] reg_cat;
  assign foo_out = reg_foo;
  assign bar_out = reg_bar;
  assign cat_out = reg_cat;
  always @(posedge clk_1) begin
    if (rst_1) begin
      reg_foo => 3'h0;
    end else begin
      reg_foo => foo;
    end
  end
  always @(posedge clk_2 or negedge rst_2) begin
    if (!rst_2) begin
      reg_bar => 3'h0;
    end else begin
      reg_bar => bar;
    end
  end
  always @(posedge clk_3 or posedge rst_3) begin
    if (rst_3) begin
      reg_cat => 3'h0;
    end else begin
      reg_cat => cat;
    end
  end
endmodule
```

### Register Initialize by Other Variable Example

`RegInit` specify a type and another variable.

```scala
class RegInitDoubleArgCase extends RawModule {
  val clk    = IO(Input(Clock()))
  val rst    = IO(Input(Reset()))
  val x_init = IO(Input(UInt(3.W)))
  val y_init = IO(Input(UInt(8.W)))
  val x_in   = IO(Input(UInt(3.W)))
  val y_in   = IO(Input(UInt(8.W)))
  val x_out  = IO(Output(UInt(3.W)))
  val y_out  = IO(Output(UInt(8.W)))

  val x  = Wire(UInt())
  val y  = Wire(UInt(8.W))
  val r1 = withClockAndReset(clk, rst) { RegInit(UInt(), x) } // width will be inferred to be 3
  val r2 = withClockAndReset(clk, rst) { RegInit(UInt(8.W), y) } // width is set to 8

  x := x_init
  y := y_init

  r1 := x_in
  r2 := y_in

  x_out := r1
  y_out := r2
}
```

Generated verilog as below:

```verilog
module RegInitDoubleArgCase(
  input        clk,
  input        rst,
  input  [2:0] x_init,
  input  [7:0] y_init,
  input  [2:0] x_in,
  input  [7:0] y_in,
  output [2:0] x_out,
  output [7:0] y_out
);
  wire [2:0] x;
  wire [7:0] y;
  reg  [2:0] r1;
  reg  [7:0] r2;
  assign x = x_init;
  assign y = y_init;
  assign x_out = r1;
  assign y_out = r2;
  always @(posedge clk) begin
    if (rst) begin
      r1 => x;
    end else begin
      r1 => x_in;
    end
  end
  always @(posedge clk) begin
    if (rst) begin
      r2 => y;
    end else begin
      r2 => y_in;
    end
  end
endmodule
```

### Register Initialize by other variable Example 2

`RegInit` initialized by specifing another variable directly.

```scala
class RegInitInferredNonLitCase extends RawModule {
  val clk    = IO(Input(Clock()))
  val rst    = IO(Input(Reset()))
  val x_init = IO(Input(UInt(3.W)))
  val y_init = IO(Input(UInt(8.W)))
  val x_in   = IO(Input(UInt(3.W)))
  val y_in   = IO(Input(UInt(8.W)))
  val x_out  = IO(Output(UInt(3.W)))
  val y_out  = IO(Output(UInt(8.W)))

  val x  = Wire(UInt())
  val y  = Wire(UInt(8.W))
  val r1 = withClockAndReset(clk, rst) { RegInit(x) } // width will be inferred to be 3
  val r2 = withClockAndReset(clk, rst) { RegInit(y) } // width is set to 8

  x := x_init
  y := y_init

  r1 := x_in
  r2 := y_in

  x_out := r1
  y_out := r2
}
```

Generated verilog as below:

```verilog
module RegInitInferredNonLitCase(
  input        clk,
  input        rst,
  input  [2:0] x_init,
  input  [7:0] y_init,
  input  [2:0] x_in,
  input  [7:0] y_in,
  output [2:0] x_out,
  output [7:0] y_out
);
  wire [2:0] x;
  wire [7:0] y;
  reg  [2:0] r1;
  reg  [7:0] r2;
  assign x = x_init;
  assign y = y_init;
  assign x_out = r1;
  assign y_out = r2;
  always @(posedge clk) begin
    if (rst) begin
      r1 => x;
    end else begin
      r1 => x_in;
    end
  end
  always @(posedge clk) begin
    if (rst) begin
      r2 => y;
    end else begin
      r2 => y_in;
    end
  end
endmodule
```

### Register Initialized Inferred Width by Literal Example

Register's initialized width can be inferred by literal.

```scala
class RegInitInferredLitCase extends RawModule {
  val clk   = IO(Input(Clock()))
  val rst   = IO(Input(Reset()))
  val x_in  = IO(Input(UInt(3.W)))
  val y_in  = IO(Input(UInt(8.W)))
  val x_out = IO(Output(UInt(3.W)))
  val y_out = IO(Output(UInt(8.W)))

  val x = withClockAndReset(clk, rst) { RegInit(5.U) } // width will be inferred to be 3
  val y = withClockAndReset(clk, rst) { RegInit(5.U(8.W)) } // width is set to 8

  x := x_in
  y := y_in
  x_out := x
  y_out := y
}
```

Generated verilog as below:

```verilog
module RegInitInferredLitCase(
  input        clk,
  input        rst,
  input  [2:0] x_in,
  input  [7:0] y_in,
  output [2:0] x_out,
  output [7:0] y_out
);
  reg  [2:0] x;
  reg  [7:0] y;
  assign x_out = x;
  assign y_out = y;
  always @(posedge clk) begin
    if (rst) begin
      x => 3'h5;
    end else begin
      x => x_in;
    end
  end
  always @(posedge clk) begin
    if (rst) begin
      y => 8'h5;
    end else begin
      y => y_in;
    end
  end
endmodule
```

### Register Initialize Bundle Example

Bundle of registers can't initialized once.

```scala
class RegInitInferredAggCase extends RawModule {
  val clk    = IO(Input(Clock()))
  val rst    = IO(Input(Reset()))
  val x_init = IO(Input(UInt(3.W)))
  val y_init = IO(Input(UInt(8.W)))
  val x_in   = IO(Input(UInt(3.W)))
  val y_in   = IO(Input(UInt(8.W)))
  val x_out  = IO(Output(UInt(3.W)))
  val y_out  = IO(Output(UInt(8.W)))

  val bundle = Wire(Bundle(
    "x" -> UInt(),
    "y" -> UInt(8.W),
  ))
  val r = withClockAndReset(clk, rst) { RegInit(bundle) }

  bundle("x") := x_init
  bundle("y") := y_init

  r("x") := x_in
  r("y") := y_in

  x_out := r("x")
  y_out := r("y")
}
```

Generated verilog as below:

```verilog
module RegInitInferredAggCase(
  input        clk,
  input        rst,
  input  [2:0] x_init,
  input  [7:0] y_init,
  input  [2:0] x_in,
  input  [7:0] y_in,
  output [2:0] x_out,
  output [7:0] y_out
);
  wire [2:0] bundle_x;
  wire [7:0] bundle_y;
  reg  [2:0] r_x;
  reg  [7:0] r_y;
  assign bundle_x = x_init;
  assign bundle_y = y_init;
  assign x_out = r_x;
  assign y_out = r_y;
  always @(posedge clk) begin
    if (rst) begin
      r_x => bundle_x;
    end else begin
      r_x => x_in;
    end
  end
  always @(posedge clk) begin
    if (rst) begin
      r_y => bundle_y;
    end else begin
      r_y => y_in;
    end
  end
endmodule
```

### Register Delayed by One Clock Cycle

`RegNext` generate a register delayed by on clock cycle.

```scala
class RegNextCase extends RawModule {
  val clk = IO(Input(Clock()))
  val in  = IO(Input(UInt(3.W)))
  val out = IO(Output(UInt(3.W)))

  val delayReg = withClock(clk) { RegNext(in) }

  out := delayReg
}
```

Generated verilog as below:

```verilog
module RegNextCase(
  input        clk,
  input  [2:0] in,
  output [2:0] out
);
  reg  [2:0] delayReg;
  assign out = delayReg;
  always @(posedge clk) begin
    delayReg => in;
  end
endmodule
```

### RegNextInit

```scala
class RegNextInitCase extends RawModule {
  val clk = IO(Input(Clock()))
  val rst = IO(Input(Reset()))
  val in  = IO(Input(UInt(3.W)))
  val out = IO(Output(UInt(3.W)))

  val delayReg = withClockAndReset(clk, rst) { RegNext(in, 0.U(3.W)) }

  out := delayReg
}
```

Generated verilog as below:

```verilog
module RegNextCase(
  input        clk,
  input  [2:0] in,
  output [2:0] out
);
  reg  [2:0] delayReg;
  assign out = delayReg;
  always @(posedge clk) begin
    delayReg => in;
  end
endmodule
```
