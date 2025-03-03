# Changelog

All notable changes to this project will be documented in this file.

## Table of Contents

* [Unreleased](#unreleased)
* [0.4.0](#040---2020-02-03)
* [0.3.0](#030---2019-07-16)
* [0.2.0](#020---2019-05-01)

## [Unreleased]

## [0.4.0] - 2020-02-03

### Added

* Support exported global variables
  ([#32](https://github.com/wasmerio/wasmer-ruby/pull/32) by
  [@Hywan])

  ```ruby
  instance = Wasmer::Instance.new bytes
  x = instance.globals.x

  assert x.mutable
  assert_equal x.value, 7

  x.value = 42

  assert_equal x.value, 42
  ```

* Support memory without an exported memory
  ([#31](https://github.com/wasmerio/wasmer-ruby/pull/31) by
  [@Hywan])

### Changed

* Set `@memory` to `nil` if none is exported
  ([#33](https://github.com/wasmerio/wasmer-ruby/pull/33) by
  [@Hywan])
* Migrate CI from CircleCI to Github Actions
  ([#28](https://github.com/wasmerio/wasmer-ruby/pull/28) by
  [@Hywan])
* Format code
  ([#26](https://github.com/wasmerio/wasmer-ruby/pull/26) by
  [@Atul9])
* Update Rutie to 0.7.0 (Rust) and 0.0.4 (Ruby)
  ([#23](https://github.com/wasmerio/wasmer-ruby/pull/23) by
  [@Hywan])
* Update Wasmer from 0.6.0 to 0.14.0
  ([#23](https://github.com/wasmerio/wasmer-ruby/pull/23),
  [#24](https://github.com/wasmerio/wasmer-ruby/pull/24),
  [#25](https://github.com/wasmerio/wasmer-ruby/pull/25),
  [#35](https://github.com/wasmerio/wasmer-ruby/pull/35),
  by [@Hywan])

### Security

* Update `rake` and other dependencies
  ([#36](https://github.com/wasmerio/wasmer-ruby/pull/36) by
  [@Hywan])

## [0.3.0] - 2019-07-16

### Added

* Add the `Memory.grow` method
  ([#19](https://github.com/wasmerio/wasmer-ruby/pull/19) by
  [@Hywan])
* Typed arrays implement [the `Enumerable`
  mixin](https://docs.ruby-lang.org/en/2.6.0/Enumerable.html)
  ([#15](https://github.com/wasmerio/wasmer-ruby/pull/15) by
  [@irxground])

  ```ruby
  memory = instance.memory.uint8_view pointer
  string = ""

  # Now we can write this more Ruby idiomatic loop:
  memory.each do |char|
    break if char == 0
    string += char.chr
  end
  ```
* Implement `instance.exports.respond_to?` to test whether an exported
  function exists
  ([#9](https://github.com/wasmerio/wasmer-ruby/pull/9) by
  [@irxground])

   ```ruby
   instance = Wasmer::Instance.new(self.bytes)

   assert instance.exports.respond_to?(:foo)
   ```
* Handle exported functions that return nothing, aka void functions
  ([#8](https://github.com/wasmerio/wasmer-ruby/pull/8) by [@Hywan])
* Add the `greet` eaxmple
  ([#12](https://github.com/wasmerio/wasmer-ruby/pull/12) by
  [@Hywan])
* Set up Bors
  ([#10](https://github.com/wasmerio/wasmer-ruby/pull/10) by
  [@Hywan])

### Changed

* Improve Ruby exception raising
  ([#5](https://github.com/wasmerio/wasmer-ruby/pull/5) by
  [@irxground])
* Update Wasmer to 0.5.5
  ([#20](https://github.com/wasmerio/wasmer-ruby/pull/20) by
  [@Hywan])
* Update Wasmer to 0.4.2
  ([#11](https://github.com/wasmerio/wasmer-ruby/pull/11) by
  [@Hywan])
* Update Wasmer to 0.4.1
  ([#7](https://github.com/wasmerio/wasmer-ruby/pull/7) by [@Hywan])
* Update Rutie
  ([#16](https://github.com/wasmerio/wasmer-ruby/pull/16) by
  [@edvakf])
* Move all symbols inside the `Wasmer` module
  ([#4](https://github.com/wasmerio/wasmer-ruby/pull/4) by
  [@irxground])

  ```ruby
  # Read from the `Wasmer` module.
  instance = Wasmer::Instance.new()
  ```

* Improve documentation
  ([#14](https://github.com/wasmerio/wasmer-ruby/pull/14) by
  [@denniscollective])
* Use `assert_nil` instead of `assert_equal_nil` in tests
  ([#13](https://github.com/wasmerio/wasmer-ruby/pull/13) by
  [@Hywan])

## [0.2.0] - 2019-05-01

[Unreleased]: https://github.com/wasmerio/wasmer-ruby/compare/0.4.0...HEAD
[0.4.0]: https://github.com/wasmerio/wasmer-ruby/compare/0.3.0...0.4.0
[0.3.0]: https://github.com/wasmerio/wasmer-ruby/compare/0.2.0...0.3.0
[0.2.0]: https://github.com/wasmerio/wasmer-ruby/compare/0.1.0...0.2.0
[@Hywan]: https://github.com/Hywan
[@irxground]: https://github.com/irxground
[@edvakf]: https://github.com/edvakf
[@Atul9]: https://github.com/Atul9
