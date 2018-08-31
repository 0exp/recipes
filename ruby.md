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
