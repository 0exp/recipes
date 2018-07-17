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
