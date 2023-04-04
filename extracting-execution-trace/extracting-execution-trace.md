## Introduction

To prove arbitrary computation we need to extract the execution trace and prove that every step of the computation is correct. In our case, we want to prove the execution trace of WebAssembly run-time. We pick the [wasmi - WebAssembly (Wasm) Interpreter](https://github.com/paritytech/wasmi) a light weight and deterministic Wasm run-time to starting with.

## Method

- We created two new structures namely, `Predator`, `Trace`. `Predator` will record every single `Trace` to a vector of `Trace`.
- Inject the `Predator` instance to `Executor` of `wasmi` and start extracting before the opcode is executed (_wasmi have some engineering turning so we might need to transform to the necessary format of the `Trace`_).
- Passing a single reference of `Predator` from the construction of `Engine` to the `EngineExecutor`.

## Format of the trace

After execute a Wasm program we might have a `Vec<Trace>` following this structure.

- **iaddr:** Address of instruction in the memory
- **program_counter:** Number of instructions have been executed
- **opcode:** Opcode has been executed
- **stack:** The stack of Wasm run-time
- **stack_depth:** The depth of the above stack
- **local:** Local variables which can be accessed by using `local.get <index>`
- **local_depth:** The depth of local
- **calling_frame:** Calling frame of Wasm run-time
- **calling_frame_depth:** The depth of calling frame
- **action:** Memory access action
- **memory_address:** Memory address
- **memory:** Memory chunks

```
Trace {
    iaddr: InstructionPtr {
        ptr: 0x0000000131607800,
    },
    program_counter: 4,
    opcode: Return(
        DropKeep {
            drop: 2,
            keep: 1,
        },
    ),
    stack: [
        UntypedValue {
            bits: 9,
        },
    ],
    stack_depth: 1,
    local: [
        UntypedValue {
            bits: 4,
        },
        UntypedValue {
            bits: 5,
        },
    ],
    local_depth: 2,
    calling_frame: [],
    calling_frame_depth: 0,
    action: None,
    memory_address: 0,
    memory: [],
}
```

## To test

The source code is available here [https://github.com/orochi-network/wasmi](https://github.com/orochi-network/wasmi)

```
cargo run --bin example
```

We tried with a very simple WebAssembly program

```
(module
    (func (export "add_values") (param i64 i64) (result i64)
      local.get 0
      local.get 1
      i64.add))
```

This program took two parameters and calculate the sum of them.
