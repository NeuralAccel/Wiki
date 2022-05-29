# Verilog Coding Style Guide

## Keypoints

- Use standard DFF module to initialize registers.

- Use `assign` to replace `if-else` and `case`.

- Label module name and direction on module port.

- Active low reset.

## Use standard DFF module to initialize registers

For registers, use instance of standardized library DFF modules instead of an `always` block. This will help when

1. globally replacing register types, 

2. inserting delay to all registers, 

3. explicit `enable` signal to help automation tools to insert power gate control for power optimization, and 

4. avoid the propagation issue of undefined states.

## Use `assign` to replace `if-else` and `case`

There's two flaws of `if-else` and `case` in Verilog:

1. cannot propagate undefined states

2. will generate hieratical select logic, instead of parallel ones, which are bad for area and timing

### Case A

For example, when the condition `a` is an undefined `X`, Verilog will default it to `a == 0`, and thus out will be assigned to `in2`, and the undefined state will be lost. This behavior will thus be hidden in simulation stage, and can only be discovered later on.

```verilog
if (a)
  out = in1;
else
  out = in2;
```

If we use `assign` syntax, when `a` is undefined, it will assign `out` to also be `X`, which correctly propagates the undefined state.

```verilog
assign out = a ? in1 : in2;
```

### Case B

In verilog, the `if-else` syntax will generate a hieratical circuit, which is not optimal in terms of area and timing.

```verilog
if (sel1)
  out = in1[3:0];
else if (sel2)
  out = in2[3:0];
else if (sel3)
  out = in3[3:0];
else
  out = 4'b0;
```

If we indeed want to generate a hieratical logic (meaning that each selection has different weight), we can do

```verilog
assign out = sel1 ? in1[3:0] :
             sel2 ? in2[3:0] :
             sel3 ? in3[3:0] :
                    4'b0;
```

Otherwise, we can use AND and OR to write

```verilog
assign out = ({4{sel1}} & in1[3:0])
           | ({4{sel2}} & in2[3:0])
           | ({4{sel3}} & in3[3:0]);
```

## Label module name and direction on module port

Label module name and direction on module port. `clk` and `rst` can be an exception.

Example:

```verilog
module ifu (
  input clk,
  input rst,
  input ifu_i_is_halt,
  input ifu_i_is_jump,
  input ifu_i_is_branch_taken,
  input [31:0] ifu_i_pc_target,

  output [31:0] ifu_o_pc,
  output [31:0] ifu_o_inst,

  output [31:0] ifu_o_iaddr,
  input [31:0] ifu_i_idata
);
```

## Active low reset

Yes, use active low reset.
