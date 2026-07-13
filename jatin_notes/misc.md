- Floating point -> learned in CS 370. Very flexible (infinite range). But harder to do computations with
- Fixed point -> fixed range. Fixed bits representing whole number part and fixed number of bits for fractional part. uses same logic as integer arithmetic (just shifted up or down). 

Confusion: both use fixed number of bits but represent numbers differently.

# HDL Bits

HDL = Hardware Description Language (e.g., Verilog)

- Module is basically a function that takes inputs and converts into outputs.
- Assignment is continuous (analog so like infinite invocations of the function)
- Vector is like an array
- But vector has indexing direction (little or big endian): `type [upper:lower] vector_name;`
- Constants can be represented in binary, decimal and hex: {4'ha, 4'd10} => 8'b10101010
- Replication operator: {num{vector}}

## Things to review

- Packed vs unpacked vector
