## Exception cause (Exception#cause) (an error that is raised before)

```ruby
A = Class.new(StandardError)
B = Class.new(StandardError)
C = Class.new(StandardError)

def fail_with_A
  raise A
end

def fail_with_B
  fail_with_A
rescue
  raise B
end

def fail_with_C
  fail_with_B
rescue
  raise C
end

def fail_causes
  fail_with_C
rescue => error
  puts "Error chain: #{error.cause.cause} -> #{error.cause} -> #{error}"
end

fail_causes # => Error chain: A => B => C
```

## Dont forget about proc usage style priorities!

```ruby
def x(*, &block)
  block.call(__method__) if block_given?
end

def y(*, &block)
  block.call(__method__) if block_given?
end

x y { |a| puts a }
# => :y

x y do |a|
  puts a
end
# => :x
```

## ~Ideal time measurement (monotonic)

```ruby
time_start = Process.clock_gettime(Process::CLOCK_MONOTONIC)
# ... your code...
time_end   = Process.clock_gettime(Process::CLOCK_MONOTONIC)

time_end - time_start
```

## Object unfreeze (O.O)

```ruby
require 'fiddle'

class Object
  def unfreeze
    Fiddle::Pointer.new(object_id * 2)[1] &= ~(1 << 3)
  end
end

frost = [].freeze
frost.frozen? # => true
frost << 123 # FrozenError: can't modify frozen Array
# ---
frost.unfreeze
frost.frozen? # => false
frost << 123 # => [123]
# ---
frost.freeze
frost.frozen? # => true
frost << 123 # => FrozenError: can't modify frozen Array
```

## Your current pathname, LOL (without Pathname o.o)

```ruby
Pathname.new(__FILE__).realpath # NO! >:[=]
__dir__ # YES ;)
```

## Restore original gem code after any forgotten change

```ruby
gem pristine qonfig # restore one gem
gem pristine --all # restore all gems
```

## Any object as an exception

```ruby
class Numeric
  def exception
    RuntimeError.new("Singularity o.O <#{self}>")
  end
end

raise 42 # => RuntimeError: Singularity o.O <42>
```

## Object allocations tracer (where this object came from?)

```ruby
# tracer.rb

require 'objspace'

module Tracer
  class << self
    include ObjectSpace

    def start
      trace_object_allocations_start
    end

    def where_this_came_from(object)
      "#{allocation_sourcefile(object)}:" \
      "#{allocation_sourceline(object)}"
    end
  end
end

Tracer.start

object = Object.new

another_object = Object.new

puts Tracer.where_this_came_from(object)         # => tracer.rb:22
puts Tracer.where_this_came_from(another_object) # => tracer.rb:24
```

## Ultra Minimal Rack App

```ruby
# ./config.ru

require 'rack'

class MinimalRackApp
  def call(env)
    sleep(10) # simulate simple activity
    Rack::Response.new(env).finish
  end
end

app = Rack::Builder.new do
  run MinimalRackApp.new
end

run app
```

## Email Regexp (stdlib)

```ruby
URI::MailTo::EMAIL_REGEXP
```

## Inplace `rescue`

```ruby
failing_code() rescue $!.backtrace.grep /pattern/
```

## Inline Bundler

```ruby
require 'bundler/inline'

gemfile do
  source 'https://rubygems.org'
  gem 'any_cache'
end

AnyCache.build
```


## Safe `require`

```ruby
def safe_require(name)
  require name
rescue LoadError
end
```

## Tail Call Optimization

```ruby
RubyVM::InstructionSequence.compile_option = {
  tailcall_optimization: true,
  trace_instruction: false,
}

RubyVM::InstructionSequence.new(<<~EOS).eval
  def fibo(req = 1, start = 1, l = 0, r = 1)
    return r if start == req
    fibo(req, start + 1, r, l + r)
  end
EOS
```

---

## Method definition wiht *_eval scoping rules

#### Class: `def`
- defines method on **instance**-level:

```ruby
class User
  def first_name
    'Name'
  end
end

User.isntance_methods(false) # => [:first_name]
```

#### Class: `class_eval` + `def`
- defines method on **instance**-level:

```ruby
class User
  class_eval do
    def first_name
      'Name'
    end
  end
end

User.isntance_methods(false) # => [:first_name]
```

#### Class: `instance_eval` + `def`
- defines method on **class**-level:

```ruby
class User
  instnace_eval do
    def first_name
      'Name'
    end
  end
end

User.methods(false) # => [:first_name]
User.singleton_methods(false) # => [:first_name]
```

#### Class: `define_method`
- defines method on **instance**-level:

```ruby
class User
  define_method :first_name do
    'Name'
  end
end

User.instance_methods(false) # => [:first_name]
```

#### Class: `class_eval` + `define_method`
- defines method on **instance**-level:

```ruby
class User
  class_eval do
    define_method :first_name do
      'Name'
    end
  end
end

User.instance_methods(false) # => [:first_name]
```

#### Class: `instance_eval` + `define_method`
- defines method on **instance**-level:

```ruby
class User
  instance_eval do
    define_method :first_name do
      'Name'
    end
  end
end

User.instance_methods(false) # => [:first_name]
```


#### Class: `def self.`
- defines method on **class**-level:

```ruby
class User
  def self.first_name
    'Name'
  end
end

User.methods(false) # => [:first_name]
User.singleton_methods(false) # => [:first_name]
```


#### Class: `class_eval` + `def self.`
- defines methdod on **class**-level:

```ruby
class User
  class_eval do
    def self.first_name
      'Name'
    end
  end
end

User.methods(false) # => [:first_name]
User.singleton_methods(false) # => [:first_name]
```

#### Class: `instance_eval` + `def self.`
- defines method on **class**-level:

```ruby
class User
  instance_eval do
    def self.first_name
      'Name'
    end
  end
end

User.methods(false) # => [:first_name]
User.singleton_methods(false) # => [:first_name]
```


#### Class: `define_singleton_method`
- defines method on **class**-level:

```ruby
class User
  define_singleton_method :first_name do
    'Name'
  end
end

User.methods(false) # => [:first_name]
User.singleton_methods(false) # => [:first_name]
```

#### Class: `class_eval` + `define_singleton_method`
- defines method on **class**-level:

```ruby
class User
  class_eval do
    define_singleton_method :first_name do
      'Name'
    end
  end
end

User.methods(false) # => [:first_name]
User.singleton_methods(false) # => [:first_name]
```

### class << self: `def`
- defines method on class level

```ruby
class User
  class << self
    def first_name
      'Name'
    end
  end
end

User.methods(false) # => [:first_name]
User.singleton_methods(false) # => [:first_name]
```

#### class << self: `class_eval` + `def`
- defines method on **class**-level:

```ruby
class User
  class << self
    class_eval do
      def first_name
        'Name'
      end
    end
  end
end

User.methods(false) # => [:first_name]
User.singleton_methods(false) # => [:first_name]
```

#### class << self: `instance_eval` + `def`
- defines method on **singleton-class**-level:

```ruby
class User
  class << self
    instance_eval do
      def first_name
        'Name'
      end
    end
  end
end

User.methods(false) # => []
User.singleton_methods(false) # => []
User.singleton_class.methods(false) # => [:first_name]
```

#### class << self: `define_method`
- defines method on **class**-level:

```ruby
class User
  class << self
    define_method :first_name do
      'Name'
    end
  end
end

User.methods(false) # => [:first_name]
User.singleton_methods(false) # => [:first_name]
```

#### class << self: `class_eval` + `define_method`
- defines methods on **class**-level:

```ruby
class User
  class << self
    class_eval do
      define_method :first_name do
        'Name'
      end
    end
  end
end

User.methods(false) # => [:first_name]
User.singleton_methods(false) # => [:first_name]
```


#### class << self: `instance_eval` + `define_method`
- defines method on **class**-level:

```ruby
class User
  class << self
    instance_eval do
      define_method :first_name do
        'Name'
      end
    end
  end
end

User.methods(false) # => [:first_name]
User.singleton_methods(false) # => [:first_name]
```

#### class << self: `def self.`
- defines method on **singleton-class**-level:

```ruby
class User
  class << self
    def self.first_name
      'Name'
    end
  end
end

User.methods(false) # => []
User.singleton_methods(false) # => []
User.singleton_class.methods(false) # => [:first_name]
```

#### class << self: `class_eval` + `def self.`
- defines method on **singleton-class**-level:

```ruby
class User
  class << self
    class_eval do
      def self.first_name
        'Name'
      end
    end
  end
end

User.methods(false) # => []
User.singleton_methods(false) # => []
User.singleton_class.methods(false) # => [:first_name]
```

#### class << self: `instance_eval` + `def self.`
- defines method on **singleton-class**-level:

```ruby
class User
  class << self
    instance_eval do
      def self.first_name
        'Name'
      end
    end
  end
end

User.methods(false) # => []
User.singleton_methods(false) # => []
User.singleton_class.methods(false) # => [:first_name]
```

#### class << self: `define_singleton_method`
- defines method on **singleton-class**-level:

```ruby
class User
  class << self
    define_singleton_method :first_name do
      'Name'
    end
  end
end

User.methods(false) # => []
User.singleton_methods(false) # => []
User.singleton_class.methods(false) # => [:first_name]
```

#### class << self: `class_eval` + `define_singleton_method`
- defines method on **singleton-class**-level:

```ruby
class User
  class << self

    class_eval do
      define_singleton_method :first_name do
        'Name'
      end
    end
  end
end

User.methods(false) # => []
User.singleton_methods(false) # => []
User.singleton_class.methods(false) # => [:first_name]
```


#### class << self: `instance_eval` + `define_singleton_method`
- defines method on **singleton-class**-level:

```ruby
class User
  class << self
    instance_eval do
      define_singleton_method :first_name do
        'Name'
      end
    end
  end
end

User.methods(false) # => []
User.singleton_methods(false) # => []
User.singleton_class.methods(false) # => [:first_name]
```

---

## Exception Tree

```ruby
- NoMemoryError
- ScriptError
  - LoadError
  - NotImplementedError
  - SyntaxError
- SecurityError
- SignalException
  - Interrupt
- StandardError – default for rescue
  - ArgumentError
    - UncaughtThrowError
  - EncodingError
  - FiberError
  - IOError
    - EOFError
  - IndexError
    - KeyError
    - StopIteration
  - LocalJumpError
  - NameError
    - NoMethodError
  - RangeError
    - FloatDomainError
  - RegexpError
  - RuntimeError – default for raise
    - FrozenError
  - SystemCallError
    - Errno::*
  - ThreadError
  - TypeError
  - ZeroDivisionError
- SystemExit
- SystemStackError
- fatal – impossible to rescue
```

---
