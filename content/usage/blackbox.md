---
title: BlackBox
weight: 6
---

{{< toc >}}

**BlackBoxes** are used to instantiate externally defined modules. This construct is useful for hardware constructs that cannot be described in Scala and for connecting to FPGA or other IP not defined in

```scala
class IBUFDS extends BlackBox (
  Map(
    "DIFF_TERM" -> "TRUE",
    "IOSTANDARD" -> "DEFAULT"
  )
) {
  val O  = IO(Output(Clock()))
  val I  = IO(Input(Clock()))
  val IB = IO(Input(Clock()))
}

class IBUF extends BlackBox {
  val O  = IO(Output(Clock()))
  val I  = IO(Input(Clock()))
  val IB = IO(Input(Clock()))
}

class BlackBoxCase extends RawModule {
  val clk_125M = IO(Input(Clock()))
  val clk_25M  = IO(Input(Clock()))
  val sel      = IO(Input(Bool()))
  val clk_out  = IO(Output(Clock()))

  val ibufds = Module(new IBUFDS)
  val ibuf   = Module(new IBUF  )

  val u_ibufds = ibufds.genInst()
  val u_ibuf   = ibuf.genInst()

  u_ibufds("I" ) <> clk_125M
  u_ibufds("IB") <> clk_25M

  u_ibuf("I" ) <> clk_125M
  u_ibuf("IB") <> clk_25M

  clk_out := Mux(sel, u_ibufds("O"), u_ibuf("O"))
}
```

Generated verilog as below:

```verilog
module BlackBoxCase(
  input   clk_125M,
  input   clk_25M,
  input   sel,
  output  clk_out
);
  wire  u_ibufds_O;
  wire  u_ibuf_O;
  IBUFDS #(.DIFF_TERM("TRUE"), .IOSTANDARD("DEFAULT")) u_ibufds (
    .IB (clk_25M   ),
    .I  (clk_125M  ),
    .O  (u_ibufds_O)
  );
  IBUF u_ibuf (
    .IB (clk_25M ),
    .I  (clk_125M),
    .O  (u_ibuf_O)
  );
  assign clk_out = sel ? u_ibufds_O : u_ibuf_O;
endmodule
```
