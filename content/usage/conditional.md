---
title: Conditional
weight: 5
---

{{< toc >}}

## When Statement

`when`, `elsewhen` and `otherwise` are similar to `if`, `else if` and `else`.
 
 Depends on variable's type, generating different `always` block:
 
| Variable type                          | Generated Block                          |
| -------------------------------------- | ---------------------------------------- |
| wire                                   | always @(*)                              |
| register synchronous reset             | always @(posedge clock)                  |
| register asynchronous reset at posedge | always @(posedge clock or posedge reset) |
| register asynchronous reset at negedge | always @(posedge clock or negedge reset) |

### When Example

```scala
class WhenCase extends RawModule {
  val clk = IO(Input(Clock()))
  setClock(clk)
  val sel1 = IO(Input(Bool()))
  val sel2 = IO(Input(Bool()))
  val out1 = IO(Output(UInt(3.W)))
  val out2 = IO(Output(UInt(3.W)))

  val foo = Wire(UInt(3.W))
  val bar = Reg(UInt(3.W))

  foo := 2.U
  when (sel1) {
    foo := 5.U
  }
  bar := 4.U
  when (sel2) {
    bar := 3.U
  }
  out1 := foo
  out2 := bar
}
```

Generated verilog as below:

```verilog
module WhenCase(
  input        clk,
  input        sel1,
  input        sel2,
  output [2:0] out1,
  output [2:0] out2
);
  wire [2:0] foo;
  reg  [2:0] bar;
  assign foo = 3'h2;
  assign out1 = foo;
  assign out2 = bar;
  always @(*) begin
    if (sel1) begin
      foo <= 3'h5;
    end
  end
  always @(posedge clk) begin
    if (sel2) begin
      bar <= 3'h3;
    end
    bar <= 3'h4;
  end
endmodule
```

### ElseWhen Example

```scala
class ElseWhenCase extends RawModule {
  val clk = IO(Input(Clock()))
  setClock(clk)
  val foo_sel1 = IO(Input(Bool()))
  val foo_sel2 = IO(Input(Bool()))
  val bar_sel1 = IO(Input(Bool()))
  val bar_sel2 = IO(Input(Bool()))
  val out1     = IO(Output(UInt(3.W)))
  val out2     = IO(Output(UInt(3.W)))

  val foo = Wire(UInt(3.W))
  val bar = Reg(UInt(3.W))

  foo := 0.U
  when (foo_sel1) {
    foo := 5.U
  } .elsewhen (foo_sel2) {
    foo := 4.U
  }
  bar := 0.U
  when (bar_sel1) {
    bar := 3.U
  } .elsewhen (bar_sel2) {
    bar := 2.U
  }
  out1 := foo
  out2 := bar
}
```

Generated verilog as below:

```verilog
module ElseWhenCase(
  input        clk,
  input        foo_sel1,
  input        foo_sel2,
  input        bar_sel1,
  input        bar_sel2,
  output [2:0] out1,
  output [2:0] out2
);
  wire [2:0] foo;
  reg  [2:0] bar;
  assign foo = 3'h0;
  assign out1 = foo;
  assign out2 = bar;
  always @(*) begin
    if (foo_sel1) begin
      foo <= 3'h5;
    end
    else if (foo_sel2) begin
      foo <= 3'h4;
    end
  end
  always @(posedge clk) begin
    if (bar_sel1) begin
      bar <= 3'h3;
    end
    else if (bar_sel2) begin
      bar <= 3'h2;
    end
    bar <= 3'h0;
  end
endmodule
```

### Otherwise Example

```scala
class OtherwiseCase extends RawModule {
  val clk = IO(Input(Clock()))
  val rst = IO(Input(Reset()))
  setClockAndReset(clk, rst)
  val sel1 = IO(Input(Bool()))
  val sel2 = IO(Input(Bool()))
  val out1 = IO(Output(UInt(3.W)))
  val out2 = IO(Output(UInt(3.W)))
  val out3 = IO(Output(UInt(4.W)))

  val foo = Wire(UInt(3.W))
  val bar = Reg(UInt(3.W))
  val car = RegInit(14.U(4.W))

  foo := 1.U
  when (sel1) {
    foo := 5.U
    when (sel2) {
      foo := 7.U
    }
  } otherwise {
    foo := 2.U
  }

  bar := 2.U
  when (sel2) {
    bar := 3.U
  } otherwise {
    bar := 4.U
  }

  car := 1.U
  when (sel2) {
    car := 2.U
  } .elsewhen(sel1) {
    car := 3.U
  } otherwise {
    car := 4.U
  }

  out1 := foo
  out2 := bar
  out3 := car
}
```

Generated verilog as below:

```verilog
module OtherwiseCase(
  input        clk,
  input        rst,
  input        sel1,
  input        sel2,
  output [2:0] out1,
  output [2:0] out2,
  output [3:0] out3
);
  wire [2:0] foo;
  reg  [2:0] bar;
  reg  [3:0] car;
  assign foo = 3'h1;
  assign out1 = foo;
  assign out2 = bar;
  assign out3 = car;
  always @(*) begin
    if (sel1) begin
      foo <= 3'h5;
      if (sel2) begin
        foo <= 3'h7;
      end
    end
    else begin
      foo <= 3'h2;
    end
  end
  always @(posedge clk) begin
    if (sel2) begin
      bar <= 3'h3;
    end
    else begin
      bar <= 3'h4;
    end
    bar <= 3'h2;
  end
  always @(posedge clk) begin
    if (rst) begin
      car => 4'he;
    end
    else if (sel2) begin
      car <= 4'h2;
    end
    else if (sel1) begin
      car <= 4'h3;
    end
    else begin
      car <= 4'h4;
    end
    car <= 4'h1;
  end
endmodule
```

## Switch Statement

`switch` statements is similar to `case` statements in verilog.

```scala
swtich (case_expr) {
  is(condition1) {
    true_statement1
  }
  is(condition2) {
    true_statement2
  }
  ...
  default {
    default_statement
  }
}
```

is similar to:

```verilog
case(case_expr)
    condition1     :             true_statement1 ;
    condition2     :             true_statement2 ;
    ...
    default        :             default_statement ;
endcase
```

### Enum as Conditional

`val state_on :: state_off :: Nil = Enum(2)` create two constants with name, from left to right represent 0, 1 and so on. It easily is used as condition.

```scala
class SwitchCase extends RawModule {
  val clk = IO(Input(Clock()))
  val rst = IO(Input(Reset()))

  setClockAndReset(clk, rst)
  val in  = IO(Input(UInt(1.W)))
  val out = IO(Output(UInt(2.W)))
  val out_num = IO(Output(UInt(3.W)))

  val reg = Reg(UInt(3.W))

  val state_on :: state_off :: Nil = Enum(2)
  val state = RegInit(state_on)

  state   := in
  out_num := reg

  swtich (state) {
    is (state_on) {
      out := 1.U
      reg := 2.U
    }
    is (state_off) {
      out := 3.U
      reg := 4.U
    }
    default {
      out := 0.U
      reg := 0.U
    }
  }
}
```

Generated verilog as below:

```verilog
module SwitchCase(
  input        clk,
  input        rst,
  input        in,
  output [1:0] out,
  output [2:0] out_num
);
  reg  [2:0] reg;
  reg   state;
  assign out_num = reg;
  always @(posedge clk) begin
    if (rst) begin
      state => 1'h0;
    end else begin
      state => in;
    end
  end
  always @(*) begin
    case (state)
      1'h0: //state_on
        out => 2'h1;
      1'h1: //state_off
        out => 2'h3;
      default:
        out => 2'h0;
    endcase //state
  end
  always @(posedge clk) begin
    case (state)
      1'h0: //state_on
        reg => 3'h2;
      1'h1: //state_off
        reg => 3'h4;
      default:
        reg => 3'h0;
    endcase //state
  end
endmodule
```

### Literal as Conditional

Use literal as condition value:

```scala
class SwitchLitCase extends RawModule {
  val clk = IO(Input(Clock()))
  val rst = IO(Input(Reset()))

  setClockAndReset(clk, rst)
  val in  = IO(Input(UInt(1.W)))
  val out = IO(Output(UInt(2.W)))
  val out_num = IO(Output(UInt(3.W)))

  val reg = Reg(UInt(3.W))

  val state = RegInit(0.U)

  state   := in
  out_num := reg

  swtich (state) {
    is (0.U) {
      out := 1.U
      reg := 2.U
    }
    is (1.U) {
      out := 3.U
      reg := 4.U
    }
    default {
      out := 0.U
      reg := 0.U
    }
  }
}
```

Generated verilog as below:

```verilog
module SwitchLitCase(
  input        clk,
  input        rst,
  input        in,
  output [1:0] out,
  output [2:0] out_num
);
  reg  [2:0] reg;
  reg   state;
  assign out_num = reg;
  always @(posedge clk) begin
    if (rst) begin
      state => 1'h0;
    end else begin
      state => in;
    end
  end
  always @(*) begin
    case (state)
      1'h0:
        out => 2'h1;
      1'h1:
        out => 2'h3;
      default:
        out => 2'h0;
    endcase //state
  end
  always @(posedge clk) begin
    case (state)
      1'h0:
        reg => 3'h2;
      1'h1:
        reg => 3'h4;
      default:
        reg => 3'h0;
    endcase //state
  end
endmodule
```
