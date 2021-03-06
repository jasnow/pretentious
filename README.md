[![Gem Version](https://badge.fury.io/rb/pretentious.svg)](https://badge.fury.io/rb/pretentious)

# Ruby::Pretentious

Do you have a pretentious boss or development lead that pushes you to embrace BDD/TDD but for reasons hate it or them?
here is a gem to deal with that. Now you CAN write your code first and then GENERATE tests later!! Yes you heard that
right! To repeat, this gem allows you to write your code first and then automatically generate tests using the code
you've written in a straightfoward manner!

On a serious note, this gem allows you to generate tests template much better than those generated by default
for various frameworks. It is also useful for "recording" current behavior of existing components in order
to prepare for refactoring.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'pretentious'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install pretentious

## Usage

First Create an example file (etc. example.rb) and define the classes that you want to test, if the class is
already defined elsewhere just require them. Below is an example:

```ruby
class Fibonacci

  def fib(n)
    return 0 if (n == 0)
    return 1 if (n == 1)
    return 1 if (n == 2)
    return fib(n - 1) + fib(n - 2)
  end

  def self.say_hello
    "hello"
  end

end
```

Inside a Pretentious.spec_for(...) block. Just write boring code that instantiates an object as well as calls the methods
like how you'd normally use them. Finally Specify the classes that you want to test:

```ruby
class Fibonacci

  def fib(n)
    return 0 if (n == 0)
    return 1 if (n == 1)
    return 1 if (n == 2)
    return fib(n - 1) + fib(n - 2)
  end

  def self.say_hello
    "hello"
  end

end

Pretentious.spec_for(Fibonacci) do


  instance = Fibonacci.new

  (1..10).each do |n|
    instance.fib(n)
  end

  Fibonacci.say_hello

end
```

Save your file and then switch to the terminal to invoke:

    $ ddtgen example.rb

This will automatically generate rspec tests for Fibonacci under /spec of the current working directory.

You can actually invoke rspec at this point, but the tests will fail. Before you do that you should edit
spec/spec_helper.rb and put the necessary requires and definitions there.

For this example place the following into spec_helper.rb:

```ruby
#inside spec_helper.rb

class Fibonacci

  def fib(n)
    return 0 if (n == 0)
    return 1 if (n == 1)
    return 1 if (n == 2)
    return fib(n - 1) + fib(n - 2)
  end

  def self.say_hello
    "hello"
  end

end
```

The generated spec file should look something like this

```ruby
require 'spec_helper'

RSpec.describe Fibonacci do

  context 'Scenario 1' do
    before do


      @fixture = Fibonacci.new

    end

    it 'should pass current expectations' do

      n = 1
      n_1 = 2
      n_2 = 3
      n_3 = 4
      n_4 = 5

      # Fibonacci#fib when passed n = 1 should return 1
      expect( @fixture.fib(n) ).to eq(1)

      # Fibonacci#fib when passed n = 2 should return 1
      expect( @fixture.fib(n_1) ).to eq(1)

      # Fibonacci#fib when passed n = 3 should return 2
      expect( @fixture.fib(n_2) ).to eq(2)

      # Fibonacci#fib when passed n = 4 should return 3
      expect( @fixture.fib(n_3) ).to eq(3)

      # Fibonacci#fib when passed n = 5 should return 5
      expect( @fixture.fib(n_4) ).to eq(5)

    end
  end

    it 'should pass current expectations' do


      # Fibonacci::say_hello when passed  should return hello
      expect( Fibonacci.say_hello ).to eq("hello")

    end
end
```

awesome!

You can also try this out with built-in libraries like MD5 for example ...

```ruby
#example.rb

Pretentious.spec_for(Digest::MD5) do
  sample = "This is the digest"
  Digest::MD5.hexdigest(sample)
end
```

You should get something like:

```ruby
require 'spec_helper'

RSpec.describe Digest::MD5 do

    it 'should pass current expectations' do

      sample = "This is the digest"

      # Digest::MD5::hexdigest when passed "This is the digest" should return 9f12248dcddeda976611d192efaaf72a
      expect( Digest::MD5.hexdigest(sample) ).to eq("9f12248dcddeda976611d192efaaf72a")

    end
end
```

Only RSpec is supported at this point. But other testing frameworks should be trivial to add support to.

## Handling complex parameters and object constructors

No need to do anything special, just do as what you would do normally and the pretentious gem will figure it out.
see below for an example:

```ruby
    class TestClass1

      def initialize(message)
        @message = message
      end

      def print_message
        puts @message
      end

      def something_is_wrong
        raise StandardError.new
      end
    end

    class TestClass2
      def initialize(message)
        @message = {message: message}
      end

      def print_message
        puts @message[:message]
      end
    end

    class TestClass3

      def initialize(testclass1, testclass2)
        @class1 = testclass1
        @class2 = testclass2
      end

      def show_messages
        @class1.print_message
        @class2.print_message
        "awesome!!!"
      end

    end
```

We then instantiate the class using all sorts of weird parameters:

```ruby
  Pretentious.spec_for(TestClass3) do
      another_object = TestClass1.new("test")
      test_class_one = TestClass1.new({hello: "world", test: another_object, arr_1: [1,2,3,4,5, another_object],
                                      sub_hash: {yes: true, obj: another_object}})
      test_class_two = TestClass2.new("This is message 2")

      class_to_test = TestClass3.new(test_class_one, test_class_two)
      class_to_test.show_messages
  end
```

Creating tests for TestClass3 should yield

```ruby
RSpec.describe TestClass3 do

  context 'Scenario 1' do
    before do

      var_2167529620 = "test"
      another_object = TestClass1.new(var_2167529620)
      args = {hello: "world", test: another_object, arr_1: [1, 2, 3, 4, 5, another_object], sub_hash: {yes: true, obj: another_object}}
      test_class_one = TestClass1.new(args)
      args_1 = "This is message 2"
      test_class_two = TestClass2.new(args_1)

      @fixture = TestClass3.new(test_class_one, test_class_two)

    end

    it 'should pass current expectations' do


      # TestClass3#show_messages when passed  should return awesome!!!
      expect( @fixture.show_messages ).to eq("awesome!!!")

    end
  end
```

Note that creating another instance of TestClass3 will result in the creation of another Scenario

## Things to do after

Since your tests are already written, and hopefully nobody notices its written by a machine, you may just leave it
at that. Take note of the limitations though.

But if lest your conscience suffers, it is best to go back to the specs and refine them, add more tests and behave like
a bdd'er/tdd'er.

## Limitations

Computers are bad at mind reading (for now) and they don't really know your expectation of "correctness", as such
it assumes your code is correct and can only use equality based matchers. It can also only reliably match
primitive data types and hashs and arrays to a degree. More complex expectations are unfortunately left for the humans
to resolve.

Also do note that it tries its best to determine how your fixtures are created, as well as the types
of your parameters and does so by figuring out (recursively) the components that your object needs. Failure can happen during this process.

Finally, Limit this gem for test environments only.

## Bugs

This is the first iteration and a lot of broken things could happen

## Contributing

1. Fork it (https://github.com/jedld/pretentious.git)
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
