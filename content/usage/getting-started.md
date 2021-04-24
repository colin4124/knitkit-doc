---
title: Getting Started
weight: 0
---

This page tells you how to get started with the **KnitKit**, including installation and default configuration.

{{< toc >}}

## Install requirements

Install **KnitKit** with following shell command, if you would like to install it to system, use `sudo` and remove `--user`.

```Shell
# require Python versions: 3.6+
$ pip3 --user install knitkit
```

## Project Directory Structure

**.knitkit**: **mill** build tool and dependencies for building **knitkit**.

**doc** : Documents, default is empty.

**hw**: hardware relatived files, default including **rtl** for verilog files, **knitkit** for scala project files.

**Makefile**: Project Makefile, default including knitkit generation steps.

```shell
.
├── .knitkit
│   └── knitkit
|       ├── .cache
│       ├── jars
│       │   └── knitkit.jar
│       └── mill
├── doc
├── hw
│   ├── knitkit
│   │   ├── build.sc
│   │   └── src
│   │       └── Main.scala
│   └── rtl
│── Makefile
```

## Simple Case

`hw/knitkit/src/Main.scala` demonstrates a 2-to-1 multiplexer.

```scala
class Mux2 extends RawModule {
  val sel = IO(Input(UInt(1.W)))
  val in0 = IO(Input(UInt(1.W)))
  val in1 = IO(Input(UInt(1.W)))
  val out = IO(Output(UInt(1.W)))
  out := (sel & in1) | (~sel & in0)
}
```

Type `make verilog` to generate verilog `builds/Mux2.v`:

```verilog
module Mux2(
  input   sel,
  input   in0,
  input   in1,
  output  out
);
  assign out = (sel & in1) | ((~ sel) & in0);
endmodule
```
