# <img height="48" src="https://wasmer.io/static/icons/favicon-96x96.png" alt="Wasmer logo" valign="middle"> Wasmer Ruby [![Wasmer Ruby Gem version](https://img.shields.io/gem/v/wasmer.svg)](https://rubygems.org/gems/wasmer) [![Wasmer Ruby Gem Documentation](https://img.shields.io/badge/docs-read-green)](https://www.rubydoc.info/gems/wasmer/) [![Wasmer Ruby Gem downloads](https://img.shields.io/gem/dt/wasmer.svg)](https://rubygems.org/gems/wasmer) [![Wasmer Slack Channel](https://img.shields.io/static/v1?label=chat&message=on%20Slack&color=green)](https://slack.wasmer.io)

A complete and mature WebAssembly runtime for Ruby based on [Wasmer].

Features:

  * **Easy to use**: The `wasmer` API mimics the standard WebAssembly API,
  * **Fast**: `wasmer` executes the WebAssembly modules as fast as
    possible, close to **native speed**,
  * **Safe**: All calls to WebAssembly will be fast, but more
    importantly, completely safe and sandboxed.

> Note: [Rust][rust] is required to install the Ruby library (Cargo
— the build tool for Rust — is used to compile the extension).

[Wasmer]: https://github.com/wasmerio/wasmer
[rust]: https://www.rust-lang.org/

# Install

First [install Rust][https://www.rust-lang.org/tools/install].

Then install the `wasmer` Ruby gem:

```sh
$ gem install wasmer
```

[View the `wasmer` gem on RubyGems][wasmer-gem].

[wasmer-gem]: https://rubygems.org/gems/wasmer

# Example

There is a toy program in `examples/simple.rs`, written in Rust (or
any other language that compiles to WebAssembly):

```rust
#[no_mangle]
pub extern fn sum(x: i32, y: i32) -> i32 {
    x + y
}
```

After compilation to WebAssembly, the
[`examples/simple.wasm`](https://github.com/wasmerio/wasmer-ruby/blob/master/examples/simple.wasm)
binary file is generated. ([Download
it](https://github.com/wasmerio/wasmer-ruby/raw/master/examples/simple.wasm)).

Then, we can execute it in Ruby (!) with the `examples/simple.rb` file:

```rb
require "wasmer"

bytes = IO.read "simple.wasm", mode: "rb"
instance = Wasmer::Instance.new bytes
puts instance.exports.sum 1, 2
```

And then, finally, enjoy by running:

```sh
$ ruby simple.rb
3
```

# Documentation

Browse Wasmer Ruby Gem docs at https://www.rubydoc.info/gems/wasmer/.

## The `Instance` class

Instantiates a WebAssembly module represented by bytes, and calls
exported functions on it:

```ruby
require "wasmer"

# Get the Wasm module as bytes.
wasm_bytes = IO.read "my_program.wasm", mode: "rb"

# Instantiates the Wasm module.
instance = Wasmer::Instance.new wasm_bytes

# Call a function on it.
result = instance.exports.sum 1, 2

puts result # 3
```

### Exported functions

All exported functions are accessible on the `exports`
getter. Arguments of these functions are automatically casted to
WebAssembly values.

### Exported memory

The `memory` getter exposes the `Memory` class representing the memory
of that particular instance, e.g.:

```ruby
view = instance.memory.uint8_view
```

`Instance.memory` throws an exception if no memory is exported.

See below for more information.

### Exported globals

The `globals` getter exposes the `ExportedGlobal` class represented an
exported global variable, e.g.:

```ruby
# Get the `x` global.
x = instance.globals.x

# Check whether the global is mutable.
assert x.mutable
assert_equal x.value, 7

# Update its value.
x.value = 42

# Tada!
assert_equal x.value, 42
```

## The `Memory` class

A WebAssembly instance has its own memory, represented by the `Memory`
class. It is accessible by the `Wasmer::Instance.memory` getter.

The `Memory.grow` methods allows to grow the memory by a number of
pages (of 65kb each).

```ruby
instance.memory.grow 1
```

The `Memory` class offers methods to create views of the memory
internal buffer, e.g. `uint8_view`, `int8_view`, `uint16_view`
etc. All these methods accept one optional argument: `offset`, to
subset the memory buffer at a particular offset. These methods return
respectively a `*Array` object, i.e. `uint8_view` returns a
`Uint8Array` object etc.

```ruby
offset = 7
view = instance.memory.uint8_view offset

puts view[0]
```

### The `*Array` classes

These classes represent views over a memory buffer of an instance.

| Class | View buffer as a sequence of… | Bytes per element |
|-|-|-|
| `Int8Array` | `int8` | 1 |
| `Uint8Array` | `uint8` | 1 |
| `Int16Array` | `int16` | 2 |
| `Uint16Array` | `uint16` | 2 |
| `Int32Array` | `int32` | 4 |
| `Uint32Array` | `uint32` | 4 |

All these classes share the same implementation. Taking the example of
`Uint8Array`, the class looks like this:

```ruby
class Uint8Array
    def bytes_per_element
    def length
    def [](index)
    def []=(index, value)
end
```

Let's see it in action:

```ruby
require "wasmer"

# Get the Wasm module as bytes.
wasm_bytes = IO.read "my_program.wasm", mode: "rb"

# Instantiates the Wasm module.
instance = Wasmer::Instance.new wasm_bytes

# Call a function that returns a pointer to a string for instance.
pointer = instance.exports.return_string

# Get the memory view, with the offset set to `pointer` (default is 0).
memory = instance.memory.uint8_view pointer

# Read the string pointed by the pointer.

string = ""

memory.each do |char|
  break if char == 0
  string += char.chr
end

puts string # Hello, World!
```

Notice that `*Array` treat bytes in little-endian, as required by the
WebAssembly specification, [Chapter Structure, Section Instructions,
Sub-Section Memory
Instructions](https://webassembly.github.io/spec/core/syntax/instructions.html#memory-instructions):

> All values are read and written in [little
> endian](https://en.wikipedia.org/wiki/Endianness#Little-endian) byte
> order.

Each view shares the same memory buffer internally. Let's have some fun:

```ruby
int8 = instance.memory.int8_view
int16 = instance.memory.int16_view
int32 = instance.memory.int32_view

               b₁
            ┌┬┬┬┬┬┬┐
int8[0] = 0b00000001
               b₂
            ┌┬┬┬┬┬┬┐
int8[1] = 0b00000100
               b₃
            ┌┬┬┬┬┬┬┐
int8[2] = 0b00010000
               b₄
            ┌┬┬┬┬┬┬┐
int8[3] = 0b01000000

// No surprise with the following assertions.
                  b₁
               ┌┬┬┬┬┬┬┐
assert_equal 0b00000001, int8[0]
                  b₂
               ┌┬┬┬┬┬┬┐
assert_equal 0b00000100, int8[1]
                  b₃
               ┌┬┬┬┬┬┬┐
assert_equal 0b00010000, int8[2]
                  b₄
               ┌┬┬┬┬┬┬┐
assert_equal 0b01000000, int8[3]

// The `int16` view reads 2 bytes.
                  b₂       b₁
               ┌┬┬┬┬┬┬┐ ┌┬┬┬┬┬┬┐
assert_equal 0b00000100_00000001, int16[0]
                  b₄       b₃
               ┌┬┬┬┬┬┬┐ ┌┬┬┬┬┬┬┐
assert_equal 0b01000000_00010000, int16[1]

// The `int32` view reads 4 bytes.
                  b₄       b₃       b₂       b₁
               ┌┬┬┬┬┬┬┐ ┌┬┬┬┬┬┬┐ ┌┬┬┬┬┬┬┐ ┌┬┬┬┬┬┬┐
assert_equal 0b01000000_00010000_00000100_00000001, int32[0]
```

## The `Module` class

The `Module` class contains one static method `validate`, that checks
whether the given bytes represent valid WebAssembly bytes:

```ruby
require "wasmer"

wasm_bytes = IO.read "my_program.wasm", mode: "rb"

if not Wasmer::Module.validate wasm_bytes
    puts "The program seems corrupted."
end
```

This function returns a boolean.

# Install and Testing

To compile the entire project, run the following commands:

```sh
$ just build
$ just test
$ ruby examples/simple.rb
```

(Yes, you need [`just`][just]).

[just]: https://github.com/casey/just/

# What is WebAssembly?

Quoting [the WebAssembly site][wasm]:

> WebAssembly (abbreviated Wasm) is a binary instruction format for a
> stack-based virtual machine. Wasm is designed as a portable target
> for compilation of high-level languages like C/C++/Rust, enabling
> deployment on the web for client and server applications.

About speed:

> WebAssembly aims to execute at native speed by taking advantage of
> [common hardware
> capabilities](https://webassembly.org/docs/portability/#assumptions-for-efficient-execution)
> available on a wide range of platforms.

About safety:

> WebAssembly describes a memory-safe, sandboxed [execution
> environment](https://webassembly.org/docs/semantics/#linear-memory) […].

[wasm]: https://webassembly.org/

# License

The entire project is under the MIT License. Please read [the
`LICENSE` file][license].

[license]: https://github.com/wasmerio/wasmer/blob/master/LICENSE
