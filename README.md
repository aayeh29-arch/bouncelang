# Bouncelang

Bouncelang is an esoteric programming language where computation happens through **physical collisions between spheres bouncing inside a 3D box**. Instead of a call stack or an AST walk, the "interpreter" is a real-time physics simulation, built in [Zig](https://ziglang.org/) on top of [raylib.zig](https://github.com/Not-Nik/raylib-zig): values and operations are spheres with position, velocity, and radius, and a program executes as those spheres collide, merge, and reflect off the walls of a bounded cube.

## Concept

A Bouncelang program describes a set of spheres. Each sphere is one of:

- **A `ball`** — an active, moving value that seeks out collisions.
- **An operand** — a sphere tagged with an operation (`set`, `add`, `sub`, `mul`, `div`, `mod`, `gt`, `lt`, `eq`, `shr`, `shl`, `and`, `or`, `xor`, I/O commands, etc.) that a `ball` collides into to trigger computation.

When a `ball` collides with another sphere, the two merge into a single new sphere:

- Position and velocity are combined using a **volume-weighted (mass-like) average** based on each sphere's radius, so collisions behave a bit like inelastic physical mergers.
- The resulting value is computed according to the operand's command — e.g. colliding with an `add` sphere sums the two values, colliding with a `gt` sphere keeps the larger value (and reflects the losing sphere's velocity as a kind of "failure" bounce).
- The merged sphere keeps moving and colliding as a `ball`, carrying its new value forward — **unless** the operand was a *sticky* variant.

Spheres also bounce off the walls of a fixed bounding cube and off a rotating cube mesh rendered at the center of the scene.

### Sticky operations

Every arithmetic/comparison/bitwise command has a "sticky" counterpart, prefixed with `s` (`sset`, `sadd`, `ssub`, `smul`, `sdiv`, `smod`, `sgt`, `slt`, `seq`, `sshr`, `sshl`, `sand`, `sor`, `sxor`):

- A **non-sticky** operand (`add`, `mul`, `gt`, ...) is consumed by the collision — the resulting sphere carries the computed value and keeps moving as a `ball`.
- A **sticky** operand (`sadd`, `smul`, `sgt`, ...) instead freezes in place after the collision: its velocity is zeroed, its command drops the `s` prefix (`sadd` → `add`), and it keeps the colliding ball's value. In effect, it turns into a new, reusable operand sitting in the scene for future balls to collide with — a way to "place" persistent operations mid-simulation.

### I/O

Four commands handle input/output, and (unlike arithmetic operands) never carry a value of their own in the source file:

| Command | Behavior |
|---|---|
| `ini`   | Reads an integer from stdin on collision |
| `outi`  | Writes the colliding ball's integer value to stdout |
| `inch`  | Reads a single byte from stdin |
| `outch` | Writes the colliding ball's value as a byte to stdout |

## Program format (`.bn` files)

Each line of a `.bn` file defines one sphere. Fields are whitespace-separated:

```
<command> <radius> <x> <y> <z> [<value>] [<vx> <vy> <vz>]
```

- `command` — one of the command names above (`set`, `sset`, `ini`, `outi`, `inch`, `outch`, `add`, `sadd`, `sub`, `ssub`, `mul`, `smul`, `div`, `sdiv`, `mod`, `smod`, `gt`, `sgt`, `lt`, `slt`, `eq`, `seq`, `shr`, `sshr`, `shl`, `sshl`, `and`, `sand`, `or`, `sor`, `xor`, `sxor`, `ball`)
- `radius` — the sphere's radius (float)
- `x y z` — initial position (float)
- `value` — required only for `ball` and the non-sticky arithmetic/comparison/bitwise commands (`set`, `add`, `sub`, `mul`, `div`, `mod`, `gt`, `lt`, `eq`, `shr`, `shl`, `and`, `or`, `xor`, `ball`); omitted for `ini`, `outi`, `inch`, `outch`, and all sticky variants
- `vx vy vz` — initial velocity, only present (and only meaningful) for `ball` spheres; every other sphere starts stationary until it's struck

Example — a `ball` carrying the value `3`, moving diagonally, on a collision course with an `add` operand carrying `4`:

```
ball 5.0 -50.0 0.0 0.0 3 1.0 0.0 0.0
add  5.0  0.0  0.0 0.0 4
```

## Building and running

Bouncelang is built with the Zig build system, opens an 800x800 raylib window and steps the simulation in real time at 60 FPS: a rotating cube sits at the center of the scene, and each sphere in the program is drawn in red as it moves, bounces, and collides according to the rules above.

## Status

This is an experimental, personal project exploring what a physically-simulated, spatial esoteric language looks like — not a production interpreter. Expect rough edges.
