---
name: stm32-c-development
description: Rules for writing C code for STM32 ARM microcontrollers with focus on reliability, readability, embedded constraints, and maintainability.
---

# STM32 C Development Skill

You are writing C code for STM32 ARM microcontrollers.  
The code must be suitable for embedded systems with limited resources, real hardware interaction, and long-term maintainability.

## General principles

- Prefer simple, explicit, deterministic code.
- Optimize first for correctness, robustness, and readability.
- Avoid clever constructs that make debugging harder.
- Keep runtime behavior predictable.
- Write code that is easy to inspect in a debugger.
- Favor static allocation over dynamic allocation.
- Minimize hidden side effects.
- Assume the code may run in safety-relevant or production-critical contexts.

## Language standard

- Use C, not C++.
- Prefer a modern C subset compatible with embedded toolchains, typically C11 if available.
- Do not rely on compiler-specific extensions unless explicitly required.
- If compiler-specific features are used, document them clearly.

## Project structure

- Separate hardware access, business logic, and application flow.
- Keep modules small and focused.
- Each `.c` file should have a matching `.h` file unless it is strictly private.
- Header files define the public interface.
- Source files contain the implementation and internal helpers.
- Avoid cyclic dependencies between modules.

## Naming conventions

- Use clear and descriptive names.
- Use `snake_case` for variables and functions.
- Use `PascalCase` only for type names if the project already uses that style consistently. Otherwise use `_t` style typedef names consistently.
- Use `UPPER_CASE` for macros, compile-time constants, and include guards.
- Prefix module-private static objects with a short module-specific context if needed for clarity.
- Avoid cryptic abbreviations unless they are standard in embedded development, such as `irq`, `dma`, `adc`, `gpio`, `uart`.

## Types

- Use fixed-width integer types from `<stdint.h>` such as `uint32_t`, `int16_t`, and `uint8_t`.
- Use `bool` from `<stdbool.h>` for boolean logic.
- Use `size_t` for sizes, lengths, and buffer capacities where appropriate.
- Avoid plain `int` and `unsigned` when exact width matters.
- Avoid floating point unless it is truly needed and justified by the target and timing constraints.
- Prefer enums for named states, modes, and result categories.
- Use structs to group logically connected data.

## Constants and macros

- Prefer `static const` over macros for typed constants.
- Use macros only where the preprocessor is actually needed, such as:
  - include guards
  - conditional compilation
  - register bit definitions
  - compile-time configuration
- Parenthesize macro parameters and results carefully.
- Never use function-like macros where an inline function is safer and clearer.
- Avoid magic numbers. Name all important constants.

## Functions

- Keep functions short and focused.
- A function should do one thing well.
- Prefer explicit input parameters over hidden global state.
- Validate function inputs where practical.
- Document assumptions for each public function.
- Avoid long parameter lists. Use structs if needed.
- Return status explicitly for operations that can fail.
- Use early returns for error paths when it improves clarity.
- Avoid recursion.

## State management

- Minimize global variables.
- If shared state is needed, confine it to a module and expose controlled access.
- Mark internal-only objects as `static`.
- Make ownership of data clear.
- Prefer explicit state machines for non-trivial behavior.

## Hardware access

- Isolate direct register access in dedicated low-level modules.
- Keep HAL or LL usage contained behind clear interfaces where possible.
- Do not scatter peripheral register manipulation across the whole codebase.
- Use `volatile` only where required, such as:
  - memory-mapped registers
  - data changed by interrupts
  - hardware status flags
- Do not use `volatile` as a substitute for synchronization or good design.
- Always document timing assumptions and hardware dependencies.

## STM32-specific guidance

- Use CMSIS types and definitions where appropriate.
- Use STM32 HAL or LL consistently within a project. Do not mix styles randomly.
- Prefer LL or direct register access for timing-critical paths.
- Prefer HAL for faster development in non-critical paths if that matches project goals.
- Initialize peripherals in a structured and traceable way.
- Keep clock setup, GPIO setup, interrupt setup, DMA setup, and peripheral init organized and easy to review.
- Treat startup code, linker scripts, interrupt vectors, and memory sections as critical infrastructure.
- Be explicit about cache, DMA, and memory alignment issues on MCUs where relevant.
- Consider watchdog behavior from the beginning of the design.

## Interrupts

- Keep interrupt service routines short.
- Do the minimum work in the ISR.
- Defer non-critical work to the main loop, task context, or a lower-priority handler.
- Avoid blocking operations in interrupts.
- Avoid complex branching in interrupts unless necessary.
- Protect shared data exchanged between ISR and non-ISR code.
- Be explicit about atomicity assumptions.
- Document which variables are shared with interrupts.

## Concurrency and timing

- Assume race conditions are possible when interrupts, DMA, RTOS tasks, or multiple contexts exist.
- Protect shared resources properly.
- Prefer lock-free simple designs when possible.
- Do not busy-wait unless the delay is short, intentional, and documented.
- Use timeouts for hardware polling loops.
- Every wait for hardware should have a failure path where practical.
- Make timing units explicit in variable names and APIs, for example `_ms`, `_us`, `_ticks`.

## Memory rules

- Avoid dynamic memory allocation such as `malloc`, `calloc`, `realloc`, and `free`.
- Use static or stack allocation unless there is a strong reason not to.
- Be careful with stack usage, especially in interrupt context and deeply nested call paths.
- Initialize variables before use.
- Avoid large local buffers unless stack size is known and sufficient.
- Check array bounds carefully.
- Use explicit buffer sizes in APIs.

## Error handling

- All hardware interactions that can fail should have a defined error strategy.
- Use explicit return codes or status enums.
- Distinguish between recoverable and unrecoverable errors.
- Fail safely.
- For impossible states, use assertions in debug builds if available.
- Do not silently ignore error conditions.
- Log, count, flag, or otherwise expose important faults where feasible.

## Defensive programming

- Check pointers before dereferencing when null is possible.
- Validate ranges of external inputs.
- Guard against integer overflow where relevant.
- Treat all external signals, communication inputs, and hardware states as potentially faulty.
- Assume that initialization may fail partially and handle that cleanly.

## Readability

- Use consistent indentation and brace style.
- Prefer braces even for single-line `if`, `for`, and `while` bodies.
- Keep nesting shallow.
- Use whitespace to make logic readable.
- Comment why, not what, unless the what is genuinely non-obvious.
- Remove dead code and commented-out code.
- Keep comments synchronized with the implementation.

## Comments and documentation

- Every public module should have a short description of its purpose.
- Public APIs should document:
  - purpose
  - parameters
  - return values
  - side effects
  - timing or concurrency assumptions
- Document hardware dependencies and board assumptions.
- Document units for physical values and timing values.

## Testing

- Design modules so logic can be tested off-target where possible.
- Separate pure logic from hardware-dependent code.
- Unit test calculations, parsing, state machines, and protocol handling when feasible.
- Validate critical hardware behavior on target.
- Include basic bring-up tests for clocks, GPIO, communication, and watchdog handling.

## Debugging support

- Make important state observable.
- Use structured debug output where available and appropriate.
- Avoid debug code that changes timing behavior too much in release builds.
- Provide clear error codes or status flags.
- Keep debug hooks easy to remove or compile out.

## Performance

- Optimize only after correctness and measurement.
- For hot paths:
  - reduce unnecessary copies
  - avoid repeated expensive operations
  - use appropriate data widths
  - consider interrupt load and memory access cost
- Be aware of flash size, RAM size, and CPU cycles.
- Do not trade maintainability for tiny theoretical gains without evidence.

## Safety and reliability

- Startup behavior must be deterministic.
- Initialization order must be explicit.
- The system should behave safely under:
  - invalid input
  - peripheral failure
  - timeout
  - communication loss
  - reset or brownout scenarios
- Consider watchdog integration, fault handlers, and recovery paths early.

## Forbidden or discouraged practices

- No unchecked dynamic allocation.
- No hidden dependencies between unrelated modules.
- No long blocking delays in application logic unless clearly justified.
- No register writes without clear intent.
- No mixing abstraction levels chaotically.
- No unused code left in production paths.
- No silent truncation or implicit type assumptions.
- No global mutable state unless justified and controlled.

## Preferred development style

When generating code:

- Start with a small, clear interface.
- Make assumptions explicit.
- Keep hardware-specific and generic logic separated.
- Use explicit initialization functions.
- Return meaningful status values.
- Write code that can survive future maintenance by someone else.
- Prefer boring code over fragile brilliance.

## Example expectations

Good code should look like this in spirit:

- clear module boundaries
- explicit state transitions
- deterministic main loop behavior
- short ISR handlers
- typed constants
- named configuration values
- documented hardware assumptions
- no unnecessary abstraction layers
- no allocation-heavy desktop-style patterns

## Output requirements for generated code

When generating STM32 C code:

- Provide both header and source file if appropriate.
- Include necessary includes.
- Use fixed-width integer types.
- Avoid dynamic allocation.
- Show initialization flow clearly.
- Use consistent error handling.
- Document assumptions briefly in comments.
- Keep the code ready to adapt to HAL, LL, or register-level projects.

If the user does not specify otherwise, assume:
- a HAL based environment
- no dynamic allocation
- resource-constrained target
- need for production-quality maintainable embedded C


