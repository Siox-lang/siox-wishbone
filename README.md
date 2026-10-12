# siox-wishbone

The Wishbone B4 (pipelined) interface for [siox](https://github.com/Siox-lang/sioxc): one
struct with every signal, and one view per endpoint role.

- Struct: `Wishbone<A, D>`, where `A` address width, `D` data width
- Views: `Master` and `Slave`

The clock and reset are not part of the struct; they stay ordinary ports of
the endpoint entities. See the header of `wishbone.siox` for what is left out.

## Use

Until siox has a package manager, put `wishbone.siox` at your project root (copy,
symlink or submodule) and import it:

```siox
use wishbone::{Wishbone, Slave};

entity Peripheral { clk: Bit in, bus: Wishbone<30, 32> Slave }
```

A parent connects a `Master` and a `Slave` through one `let wire: Wishbone<30, 32>;`.

## Components

| Entity | What it does |
|---|---|
| `wishbone::Interconnect<A, D, N, REGION>` | One Master to `N` Slaves. Slave `i` owns the `2^REGION` addresses at `i << REGION`; unmapped strobes get `err`. Pipelined transfers are allowed, but not across Slaves while acknowledges are outstanding. |
| `wishbone::Arbiter<A, D, N>` | `N` Masters sharing one Slave. Fixed priority (lowest index wins); the winner keeps the bus until it drops `cyc`. |

## Test

```bash
sioxc --std <sioxc>/std --test wishbone_test.siox -o wishbone_test && ./wishbone_test
```

`wishbone_test.siox` and `interconnect_test.siox` drive the interface, the
interconnect and the arbiter from testbenches. Needs a
`sioxc` with Siox-lang/sioxc#69: arrays of view ports.
