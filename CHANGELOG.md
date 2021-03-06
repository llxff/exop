## [1.1.3] - 2018.10.02

### Changes

- a bug with `type` check was fixed: previously this check passed if a parameter's value is `nil`, now `nil` passes the check only if `:atom` type is specified

## [1.1.2] - 2018.10.01

### Changes

- now it is possible to return an :error-tuple of any length from an operation. Previously only a tuple
  of two elements was treated as error result, any other results treated as success and were wrapped into :ok-tuple
- now it is possible to provide 3-arity function to the `func` check: in previous verisons this check expected only a function with arity == 2 to invoke for checking (1. params passed to an operation, 2. param to check value), starting from this version you can provide a function with arity == 3 (in this case Exop will invoke your function with: 1. params passed to an operation, 2. param to check name, 3. param to check value)
- `coerce_with` can take a function of arity == 2 (not only with arity == 1), coercion function/2 will be invoked with args: 1. parameter name 2. parameter value (coercion function with arity == 1 still takes just a parameter value)
- some checks aliases were added

## [1.1.1] - 2018.09.20

### Changes

- some dialyzer warnings were fixed
- `in` & `not_in` checks error message fix (for example, atoms list displays as atoms list :) )
- you can now name your parameters with strings, not only atoms and even combine string- and atom-named parameters
- a policy action argument now can be a value of any type (previously only map was allowed)
- there is no need to use `Exop.Policy` in a policy module anymore (you still can use it and there is no need to rewrite exsisting policies): simply define a module with policy checks (actions) functions which are expected to take a single argument (any type) and return either `true` or `false`

## [1.1.0] - 2018.09.03

### Changes

- `required` check was revised. Now `nil` is treated as a value. It means that before this version
  `required` check returned an error if a parameter had a `nil` value. From this version
  this check fails only if parameter wasn't provided to an operation.

  To simulate previous behaviour (if you need to keep backward compatibility with parameters passed into an operation)
  you can do:

  ```elixir
  parameter :a, required: true, func: &__MODULE__.old_required/2

  def old_required(_params, nil), do: {:error, "is required"}
  def old_required(_params, param), do: true
  ```

- `ValidationError` now has an operation name in its message (better for debugging and logging)

## [1.0.0] 🎉 - 2018.07.25

### Changes

A bunch of changes were made in this release with some brand new functionality,
so I decided to bump version to 1.0.0, finally.
Exop has been working since 2016 on various projects, in production.
I think it means something and I can say it is production-ready :)

- `Exop.Chain` helps you to organize a number of operations into an invocation chain
- `Exop.Fallback` handles your operations fails (error-tuple results)
- `Exop.Policy` was simplified
- minor codebase updates
- docs were reviewed

## [0.5.1] - 2018.07.16

### Changes

- An operation's `process/1` now takes a map of parameters (contract) instead of a keywords list
  (This will help you to pattern-match them)
- Credo was wiped out (with annoying warning on `@lint`)
- Code was formatted with elixir formatter

## [0.5.0] - 2018.02.22

### Changes

- New `list_item` parameter check.

## [0.4.8] - 2017.12.28

### Changes

- Fixed `required` check when a struct was checked with `inner`.

## [0.4.7] - 2017.11.29

### Changes

- Fixed `required` check when `false` was treated as the absence of a parameter.

## [0.4.6] - 2017.10.20

### Changes

- Does not validate nil parameters if they are not required. For example:

  ```elixir
  defmodule Operation do
    use Exop.Operation

    parameter :value, type: %MyStruct{}

    # ...
  end

  # Old versions
  Operation.run([]) # {:error, {:validation, %{value: ["is not expected struct"]}}

  # This version
  Operation.run([]) # {:ok, ...}
  ```

  In previous versions such code returns validation error, because `nil` is not a `MyStruct` struct (even if it is not required by default).

  In current version such behaviour is fixed and Exop will not run validations for nil parameters if they are not required.

## [0.4.5] - 2017.08.15

### Changes

- `equals` check:
  Checks whether a parameter's value exactly equals given value (with type equality).

  ```elixir
  parameter :some_param, equals: 100.5
  ```

## [0.4.4] - 2017.06.28

### Changes

- `run/1` output:
  If your `process/1` function returns a tuple `{:ok, _your_result_}` Exop will not wrap this output into former `{:ok, _output_}` tuple.

  So, if `process/1` returns `{:ok, :its_ok}` you'll get exactly that tuple, not `{:ok, {:ok, :its_ok}}`.

  (`run!/1` acts in the same manner with respect of it's bang nature)

## [0.4.3] - 2017.06.15

### Changes

- Log improvements:
  Logger provides operation's module name if a contract validation failed. Like:
  ```
  14:21:05.944 [warn]  Elixir.ExopOperationTest.Operation errors:
  param1: has wrong type
  param2: has wrong type
  ```
- Docs
  ExDocs (hex docs) were mostly provided.
  Will be useful for some IDEs/editors plugins usage (with docs helpers) and Dash users (like me).

## [0.4.2] - 2017.05.30

### Changes

- `func/2` check:
  Custom validation function now receives two arguments: the first is a contract of an operation (parameters with their values),
  the second - the actual parameter value to check. So, now you can validate a parameter depending on other parameters values.

## [0.4.1] - 2017.05.29

### Changes

- `func/1` check:
  You can provide your error message if custom validation function returns `{:error, "Your validation message"}`.
  In other cases `false` is treaded as validation fail with default message `"isn't valid"`, everything else - validation success.
- `run/1` output:
  If your `process/1` function returns a tuple `{:error, _error_reason_}` Exop will not wrap this output into former `{:ok, _output_}` tuple.

  So, if `process/1` returns `{:error, :ugly_error}` you'll get exactly that tuple, not `{:ok, {:error, :ugly_error}}`.

  (`run!/1` acts in the same manner with respect of it's bang nature (will return unwrapped value, an exception or your error tuple))
