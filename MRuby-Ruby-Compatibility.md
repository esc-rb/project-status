# MRuby / Ruby Compatibility Notes

## Core

In MRuby, `__FILE__` is a keyword that is converted into to a Ruby string by the parser. Thus, when MRuby is actually running bytecode, `__FILE__` cannot be dereferenced.

## Vestigal (may be subject to change)

### Object / Kernel

- **method_missing** - `respond_to?` on MRuby always returns true, versus MRI, which only returns true if `method_missing` is implemented on the class

- **defined?** - `defined?` does not work on mruby, which means e.g. `defined?(Foo::Bar)` must be written as `Object.const_defined?(:Foo) && Foo.const_defined?(:Bar)`

- **Heredoc** - Heredoc support in mruby is limited to `<<TEXT`; `<<~TEXT` is not supported

- **Exit** Kernel's `#exit` in MRuby does not raise a SystemExit exception, `raise SystemExit.new(1)` must be used

- **Warn** Kernel's `#warn` is not available in MRuby without introducing a backfill

### Exceptions

- **Backtrace locations** - In MRI, both call stacks and backtraces (which are just call stacks stored in an exception) come in two variants, `caller`/`backtrace` (which return an array of strings) and `caller_locations`/`backtrace_locations` (which return an array of `Thread::Backtrace::Location` objects). MRuby only has `caller` and `backtrace`, not `caller_locations` and `backtrace_locations`

### Time

- **Time.new** - The final two arguments to `Time.new` differ between MRI and mruby:
  - In MRI, the penultimate argument is a decimal value for seconds (e.g. `1.11` means one second and 11 milliseconds), and the final argument is the time zone offset.
  - In mruby, the penultimate argument is an integer value for seconds, and the final argument is an integer value for the number of microseconds. In the above example of `1.11`, you'd pass `1` and `110000` as the final two arguments to mruby.
- **iso8601** - only UTC timestamps are possible, because MRuby's Time class cannot return the local offset, e.g. `-06:00` for GMT-6. The implementation of iso8601 in this project converts local times to UTC

### Struct

- **Struct instance variables** - Struct classes cannot have attributes like `attr_accessor`, since mruby does not allow instance variables on core classes.

### Array

- **Array * method** In MRI, `Array#*` is a multi purpose method: `[0] * 3` is `[0, 0, 0]`, and `['a', 'b', 'c'] * '+'` is `"a+b+c"`. The method isn't in MRuby, and backfilling support would be more trouble than it's worth, as all the different purposes of the method would have to be ported.

### Symbol

- **String methods** Even with `mruby-symbol-ext` compiled in to MRuby, Symbol does not support all the string methods under MRuby that it does under MRI.

### File / IO

- **Pathname** MRI's `pathname` library has no equivalent in MRuby

### Thread

- While there is limited support for threads via the [mruby-thread](https://github.com/mattn/mruby-thread) MRuby gem, MRuby has no GVL, and thus cannot support multiple threads sharing the same interpreter.
- Native threads / actors are still possible with the [ZeroMQ MRuby gem](https://github.com/zeromq/mruby-zmq)

### Other Standard Libraries

- **requiring** - The mruby equivalent of Ruby's standard libraries cannot be required; they must be compiled in to mruby itself. This means e.g. `require 'json'` must be "guarded" by a check for mruby, e.g. `require 'json' unless RUBY_ENGINE == 'mruby'`. This also applies to `require 'test_bench'` and `require 'test_bench/fixture'`, as they are compiled in to MRuby, too.

- **Tempfile directory** Tempfiles in MRuby must have a directory specified when a filename is given. Supplying a directory in this case doesn't cause a problem for MRI

## License

The `mruby-ruby-compat` library is released under the [MIT License](https://github.com/test-bench/mruby-ruby-compat/blob/master/MIT-License.txt).
