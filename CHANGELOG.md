# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## Unreleased

### Added
- Implemented CFX Lua as a feature flag - `cfxlua`
  - Compound Operators: `+=, -=, *=, /=, <<=, >>=, &=, |=, and ^=`
  - Safe Navigation e.g. `t?.x?.y == nil`
  - In Unpacking e.g. `local a, b, c = t` is equivalent to `local a, b, c = t.a, t.b, t.c`
  - Set Constructors e.g. `t = { .x, .y }` is equivalent to `t = { x = true, y = true }`
  - C-Style Comments (single & multiline) e.g. `/* comment */`
  - Compile Time Jenkins' Hashes e.g. ``` `Hello, World!` -> 1395890823``` 

## [1.2.0] - 2025-01-09

### Added
- Luau: added support for user defined type functions (rfc: https://github.com/luau-lang/rfcs/blob/master/docs/user-defined-type-functions.md)

### Fixed
- Fixed a panic when parsing a malformed empty interpolated string, e.g. ``print(`{}`)``

## [1.1.2] - 2024-11-17

### Fixed
- Fixed `TokenReference::symbol("//=")` returning DoubleSlash + UnexpectedToken when `luau` and `lua53` are enabled together
- Fixed `\z` escapes not parsing in interpolated strings for `luau`

## [1.1.1] - 2024-11-16

### Fixed
- Fixed regression in trivia attachment causing trivia that begins with `\t` tab characters to be attached as leading trivia of the next token rather than trailing trivia of the current token.
- Fixed compilation failure due to bad flag definition when only `lua52` and `luau` are enabled together.

## [1.1.0] - 2024-10-12
### Added
- Added `access` fields that contain a `Option<TokenReference>` to `TypeInfo::Array` and `TypeField`.
- Added support for parsing table type field access modifiers, such as `{ read foo: number }`.

## [1.0.0] - 2024-10-08

### Added
- Added structs `TypeUnion` and `TypeIntersection` which both contain a field for a leading `TokenReference` (`|` or `&`), and a field which contains a `Punctuated<TypeInfo>`.
- Added support for parsing leading `|` and `&` in types.

### Changed
- **[BREAKING CHANGE]** Changed `TypeInfo::Union` and `TypeInfo::Intersection` to hold structs `TypeUnion` and `TypeIntersection` respectively.
- **[BREAKING CHANGE]** The `print()` function has been removed, use `ast.to_string()` instead.

## [1.0.0-rc.5] - 2024-07-06
### Changed
- **[BREAKING CHANGE]** The `types` module is now named `luau`.

## [1.0.0-rc.4] - 2024-07-06
### Fixed
- Fixed broken parsing of string interpolation.

## [1.0.0-rc.3] - 2024-07-05
### Added
- Added `|=` support to `LuaVersion`.

## [1.0.0-rc.2] - 2024-07-05
### Changed
- Changed `Error::error_message()` to return `Cow<'_, str>` instead of `Cow<'static, str>`.
- Changed `AstError::error_message()` to return `&str` instead of `Cow<'static, str>`.

## [1.0.0-rc.1] - 2024-07-05
### Added
- **full-moon now has the ability to return multiple errors.** Added `parse_fallible`, which will return a struct containing the best possible AST, and a vector of errors. Read the documentation for guarantees on the partial AST.
- The Lua version used to parse is no longer strictly based on features set, and can now be configured precisely using `LuaVersion`. `LuaVersion` is a bitfield that can attempt to parse multiple versions of Lua at once, or be used to pin down a specific version. `parse` will use the most completely available set possible (`LuaVersion::new()`), but `parse_fallible` accepts a `LuaVersion`.
- Added support for parsing Luau's floor division assignment `//=`
- Added `TokenizerErrorType::InvalidNumber` when a number fails to parse.

### Changed
- **[BREAKING CHANGE]** `parse` now returns a vector of errors.
- **[BREAKING CHANGE]** `TokenType::StringLiteral::multi_line` has been replaced with `TokenType::StringLiteral::multi_line_depth`. It serves the same purpose except instead of being an `Option<usize>`, it is now a standard `usize`. It is advised to simply check `quote_type == StringLiteralQuoteType::Brackets` to get the previous behavior.
- **[BREAKING CHANGE]** Flattened `AstError` into just what used to be `AstError::UnexpectedToken`.
- **[BREAKING CHANGE]** `Symbol::PlusEqual` and friends are now only available when using Luau.
- **[BREAKING CHANGE]** `Expression::Function((TokenReference, FunctionBody))` variant is now Boxed to ``Expression::Function(Box<(TokenReference, FunctionBody)>)` to reduce type sizes and reduce stack overflows
- **[BREAKING CHANGE]** Associativity of union / intersection types are fixed: they are parsed as left associative, whilst originally parsed as right associative
- **[BREAKING CHANGE]** LuaJIT parsing support is available as a separate feature flag `luajit`, rather than mixed with `lua52`. Added `LuaVersion::luajit()` to support this.
- **[BREAKING CHANGE]** `Symbol::Ellipse` and other references of `ellipse` have been renamed to `Symbol::Ellipsis` / `ellipsis`
- Shebangs provide their trailing trivia more accurately to the rest of full-moon.
- Attempting to display `StringLiteralQuoteType::Brackets` now returns an error rather than being marked as unreachable.
- Significantly optimized the entire codebase, helping both time to parse and wasting less stack, especially in debug mode.
- `Punctuated<T>` now implements `Default` for all `T`, rather than if `T: Default`.

### Removed
- Removed `TokenizerErrorType::UnexpectedShebang`.
- Removed `stacker` feature flag, as rewrites to the parser should make it unnecessary.

### Fixed
- Fixed comments with Unicode characters having positions that report their `character` as bytes.

## [0.19.0] - 2023-11-10
### Added
- Added support for parsing Luau's floor division `//`

### Changed
- Flattened `Expression::Value` to all be variants of `Expression` directly, as this was not used anywhere else. The extra `type_assertion` field has been moved into a new variant `Expression::TypeAssertion`. None of these variants are boxed.
- The following fields/variants have been changed from `Expression` to `Box<Expression>`: `Prefix::Expression`, `Var::Expression`, `IfExpression::condition`, `IfExpression::if_expression`, `IfExpression::else_expression`.
- When using serde, `Expression` will no longer act untagged.

### Fixed
- Fixed parsing of string interpolation double brace for Luau code
- Fixed failure to parse `\z` escapes in strings in Luau mode

## [0.18.1] - 2023-03-19
### Fixed
- Fixed `print` on a LocalAssignment in Lua 5.4 or Luau mode not including the variable name when no type specifier or attribute is included

## [0.18.0] - 2023-03-12
### Added
- Added `first` method to `Punctuated`.

### Fixed
- Fixed parse failed with Chinese token in comment and bump deps.
- Fixed `\` escapes in strings for Lua 5.2+.
- Fixed incorrect line number positions when a token contains consecutive newlines (typically multi line strings or comments)

## [0.17.0] - 2023-01-04
### Added
- `full_moon::Error` and `full_moon::ast::Ast` now implement Serialize and Deserialize.
- Added optional "stacker" feature which uses the [stacker](https://docs.rs/stacker/latest/stacker/index.html) crate to conditionally expand stack size to avoid stack overflows, a known problem with full-moon.
- Added support for string interpolation under the `roblox` feature flag
- Added support for Lua 5.2 fractional hexidecimal / hexidecimal with exponents, and LuaJIT number suffixes (LL/ULL/i) under the `lua52` feature flag

### Fixed
- Support instantiated generics with no parameters, e.g. `Foo<>`
- Support `\z` escapes (followed by line breaks) in strings for Lua 5.2+

## [0.16.2] - 2022-09-22
### Fixed
- Fixed bracketed strings `[[]]` which contain another `]` causing parse errors.

## [0.16.1]
### Fixed
- Fixed comments starting with `--[` and `--(` causing parse errors if they weren't multiline.

## [0.16.0] - 2022-09-21
### Added
- Added support for Lua 5.3 under the `lua53` feature flag. This adds in new binary and unary operators.
- Added support for Lua 5.4 under the `lua54` feature flag. This adds in variable attributes.

### Fixed
- Fixed issue with lexing strings when it matches the wrong escape sequence.
- Fixed panic when calling `TypeDeclaration::new()` under the `roblox` feature flag

### Changed
- Switched from using peg to logos for lexing.

## [0.15.1] - 2022-02-17
### Fixed
- Fixed parsing of indexed type information that contains generic packs

## [0.15.0] - 2022-01-30
### Added
- Added support for parsing generic type packs, variadic type packs, and explicit type packs in generic arguments for a type under the `roblox` feature flag (`type X<S...> = Y<(string, number), ...string, S...>`)
- Added support for string and boolean singleton types under the `roblox` feature flag (`type Element = { ["$$typeof"]: number, errorCaught: true, which: "Query" | "Mutation" | "Subscription" }`
- Added support for default types in a generic type declaration under the `roblox` feature flag (`type Foo<X = string> = X`)

### Changed
- **[BREAKING CHANGE]** renamed `TypeInfo::GenericVariadic` to `TypeInfo::GenericPack` to better represent its syntax (only affects `roblox` feature flag)
- **[BREAKING CHANGE]** replaced `GenericDeclarationParameter` to allow default types. The only `GenericDeclarationParameter` enum is now `GenericParameterInfo`, and `GenericDeclaratinoParameter` is a struct of a ParameterInfo and optional default type. (only affects `roblox` feature flag)

### Fixed
- Improved the parsing of generic type declarations to ensure generic types are always before generic type packs

## [0.14.0] - 2021-11-04
### Added
- Added `token()` methods on `BinOp`, `UnOp` and `CompoundOp` to get the token associated with the operator
- Added handling of UTF8 BOM(Byte order mark)
- Added support for variadic generics and generic variadic type packs under the `roblox` feature flag (`type Foo<T...> = () -> T...`)
- Added support for generic declarations in callback type specifiers under the `roblox` feature flag
- Added support for generic declarations for anonymous functions under the `roblox` feature flag
- Added support for [if expression](https://github.com/Roblox/luau/blob/master/rfcs/syntax-if-expression.md) syntax under the `roblox` feature flag

### Changed
- **[BREAKING CHANGE]** `generics` has been removed from `FunctionDeclaration` and `LocalFunction`, and is now available in one place under `FunctionBody`.

### Fixed
- Fixed the parsing of `goto` as an identifier when the `lua52` flag is *not* enabled

## [0.13.1] - 2021-07-07
### Fixed
- Fixed semicolon used as delimeters in Luau type tables leading to a syntax error under the `roblox` feature flag
- Fixed nested type arrays failing to parse under the `roblox` feature flag
- Fixed parsing of callback types as return types when they contained named parameters under the `roblox` feature flag

## [0.13.0] - 2021-06-26
### Changed
- **[BREAKING CHANGE]** The ast lifetime has been removed from practically everything.
- **[BREAKING CHANGE]** As a result of the above, `Owned` has been removed.

## [0.12.1] - 2021-06-19
### Fixed
- Fixed issue with parsing complex function type arguments which began with an indetifier under the `roblox` feature flag

## [0.12.0] - 2021-06-15
### Added
- Added support for parsing generic functions under the `roblox` feature flag
- Added support for parsing named function type arguments under the `roblox` feature flag

### Fixed
- Fixed regression in parsing single types within parentheses under the `roblox` feature flag.

## [0.11.0] - 2021-05-12
### Added
- Made `TokenizerError` fields accessible through methods
- Added support for typing variadic symbols in function parameters under the `roblox` feature flag
- Added support for the `...T` variadic type annotation under the `roblox` feature flag. Variadic types are only permitted as a return type (standalone or in a tuple), or as a parameter in a callback type annotation

### Fixed
- Fixed invalid parsing of tuple types under the `roblox` feature flag. Tuple types are only permitted as the return type of function bodies or callback type annotations.

### Changed
- Switched from using nom to peg for lexing.

### Fixed
- full_moon will now allocate about 200% less stack space in debug mode.

## [0.10.0] - 2021-03-26
### Added
- Added `block.iter_stmts_with_semicolon()` which returns an iterator over tuples containing the statement and an optional semicolon.
- Added `Expression::BinaryOperator{ lhs, binop, rhs }` which now handles binary operation expressions, with support for precedence.
- Added `operator.precedence()` to `BinOp` and `UnOp`. This returns the precedence value from a scale of 1-8 for the operator, where 8 is highest precedence.
- Added `binop.is_right_associative()` to `BinOp`. This returns whether the binary operator is right associative.
- Added a `lua52` feature flag for Lua 5.2 specific syntax
- Added support for `goto` and labels when using the `lua52` feature flag
- Added initialiser methods for all Luau-related structs available under the roblox feature flag.
- Added `block.last_stmt_with_semicolon()` which returns a tuple containing the last stmt in the block, and an optional token reference.

### Changed
- Updated dependency cfg_if to v1.0
- **[BREAKING CHANGE]** Moved binary operations to `Expression::BinaryOperator`. `binop` has been removed from `Expression::Value`
- **[BREAKING CHANGE]** Renamed `Value::ParseExpression` to `Value::ParenthesesExpression`
- **[BREAKING CHANGE]** Enums are now be marked as `non_exhaustive`, meaning matches on them must factor in the `_` case.
- Changed the assertion operator from `as` to `::` under the roblox feature flag. `AsAssertion` has been renamed to `TypeAssertion`, with `as_token` renamed to `assertion_op`.
- **[BREAKING CHANGE]** Changed how newline trailing trivia is bound to tokens. A token now owns any trailing trivia on the same line up to and including the newline character.
See [#125](https://github.com/Kampfkarren/full-moon/pull/125) for more details.
- **[BREAKING CHANGE]** The following names have been changed for consistency.
	- `GenericFor::expr_list` -> `GenericFor::expressions`
	- `GenericFor::with_expr_list` -> `GenericFor::with_expressions`
	- `Assignment::expr_list` -> `Assignment::expressions`
	- `Assignment::with_expr_list` -> `Assignment::with_expressions`
	- `Assignment::var_list` -> `Assignment::variables`
	- `Assignment::with_var_list` -> `Assignment::with_variables`
	- `LocalAssignment::expr_list` -> `LocalAssignment::expressions`
	- `LocalAssignment::with_expr_list` -> `LocalAssignment::with_expressions`
	- `LocalAssignment::name_list` -> `LocalAssignment::names`
	- `LocalAssignment::with_name_list` -> `LocalAssignment::with_names`
	- `Block::iter_stmts` -> `Block::stmts`
	- `Block::iter_stmts_with_semicolon` -> `stmts_with_semicolon`
	- `VarExpression::iter_suffixes` -> `VarExpression::suffixes`
	- `FunctionCall::iter_suffixes` -> `FunctionCall::suffixes`
	- `FunctionBody::func_body` -> `FunctionBody::body`
- **[BREAKING CHANGE]** All uses of `Cow<'a, TokenReference<'a>>` have been removed, though `Cow<'a, str>` remain.

### Fixed
- Fixed the start position of tokens at the beginning of a line to not be at the end of the previous line.
- TokenReference equality now checks for leading and trailing trivia to be the same.
- Allow numbers to have a trailing decimal `.`.

### Removed
- Removed `BinOpRhs`. This is now part of `Expression::BinaryOperator`.
- Removed `visit_bin_op` and related visitors. Binary operations should be handled in the expression visitor
- Removed all deprecated members.

## [0.9.0] - 2020-12-21
### Added
- Added support for retrieving the `Punctuated` sequence of fields in a `TableConstructor`

### Changed
- `TableConstructor::iter_fields` is now deprecated in favour of `parameters().iter`

### Fixed
- Fixed visit_numeric_for running twice.

## [0.8.0] - 2020-12-21
### Added
- Added `with_XXX` methods to Roblox-related structs under the `roblox` feature flag
- Added support for retrieving the `Punctuated` sequence of parameters in a `FunctionBody`
- Added support for types within generic and numeric for loops under the `roblox` feature flag
- Added support for shorthand array type notation (`type Foo = { number }`) under the `roblox` feature flag

### Fixed
- Fixed parse error for exponents with an explicit positive sign (eg. `1e+5`)

### Changed
- Use intra doc links, remove unnecessary linking for some items in docs.
- `FunctionBody::iter_parameters` is now deprecated in favour of `punctuated().iter`

## [0.7.0] - 2020-11-06
### Added
- Added support for exporting types (`export type Foo = { bar: number }`) under the `roblox` feature flag
- Added support for using types from other modules (`local x: module.Foo`) under the `roblox` feature flag
- Added support for parsing a shebang

### Fixed
- Fixed type declaration of objects not supporting trailing commas
- Fixed an issue where `continue` was not treated similar to `return` or `break`. It is now moved to `LastStmt` instead of a `Stmt`
- Fixed long comments and long strings containing multi-byte characters.

### Changed
- `TableConstructor` now uses `Punctuated` internally; `TableConstructor::iter_fields` returns an iterarator over `Field`'s.

## [0.6.2] - 2020-07-11
### Fixed
- Fixed an error related with `visit_compound_op` and the `roblox` feature flag

## [0.6.1] - 2020-07-05
### Fixed
- Fixed `visit_un_op` not being called correctly

## [0.6.0] - 2020-06-02
### Added
- Added support for `continue` under `roblox` feature flag
- Added support for compound assignments (eg. `x += 1`) under the `roblox` feature flag
- Added support for intersectional types (`string & number`) under the `roblox` feature flag
- Added support for underscores as numeric seperators (eg. `1_123_531`) under the `roblox` feature flag

### Fixed
- Fixed old function return type syntax under `roblox` feature flag
- Fixed `leading_trivia` not being populated correctly within `TokenReference`s inside `extract_token_references` due to CRLF line endings
- Fixed parse error for numbers with negative exponents
- Fixed parse error for fractional numbers with no integer part

## [0.5.0] - 2020-04-21
### Added
- Added `TokenReference::with_token`
- Added `Token::new`
- Added `Puncutated::from_iter`
- Added `TokenType::spaces` and `TokenType::tabs`
- Added `Punctuated::last`
- Display is now implemented on all nodes (Closes #26)

### Changed
- TokenReference has been completely rewritten to now be a representation of an inner token and its leading/trailing trivia.
- `ignore` is now deprecated in favor of `is_trivia`
- `last_stmts` is now deprecated in favor of `last_stmt`

### Fixed
- Fixed a bug where illogical punctuated sequences were created when function bodies had both named parameters and varargs

## [0.4.0-rc.14] - 2020-01-27
### Fixed
- Fixed serde being used even when the `serde` feature flag was not active
- Fixed tabs in comments leading to a parser error.

## [0.4.0-rc.13] - 2020-01-18
### Added
- Added support for Roblox typed Lua syntax under the `roblox` feature flag

### Changed
- Changed internal parser which should lead to better performance

## [0.4.0-rc.12] - 2019-11-13
### Added
- Added a `roblox` feature flag for Roblox specific syntax
- Added binary literals when using `roblox` feature flag

### Fixed
- Fixed strings with escaped new lines being unparseable

### Removed
- Removed leftover `dbg!` line

## [0.4.0-rc.11] - 2019-10-27
### Fixed
- Fixed a bug where subtraction without spaces and the right hand side being a number would cause a parsing error (e.g. `x-7`)

## [0.4.0-rc.10] - 2019-10-20
### Added
- Added a `similar` method to `node::Node` to check if two nodes are semantically equivalent

## [0.4.0-rc.9] - 2019-10-11
### Fixed
- Fixed performance issue with `Node::start_position` and `Node::end_position`

## [0.4.0-rc.8] - 2019-10-07
### Removed
- Removed dependency on crossbeam.

## [0.4.0-rc.7] - 2019-10-03
### Added
- Added `visit_XXX_end` methods for when completing a visit on a node

## [0.4.0-rc.6] - 2019-10-01
### Fixed
- Fixed unexpected parsing issues with UTF-8 strings

## [0.4.0-rc.5] - 2019-09-30
### Fixed
- Fixed tokens not being visited for `TokenReference`

## [0.4.0-rc.4] - 2019-09-30
### Fixed
- Fixed a massive performance bug
- Fixed `visit_ast` not following tokens

## [0.4.0-rc.3] - 2019-09-11
### Added
- Added `Owned` implementation for errors

## [0.4.0-rc.2] - 2019-09-11
### Changed
- Changed signature of `Node::surrounding_ignore_tokens` to have more lenient lifetimes

## [0.4.0-rc.1] - 2019-09-08
### Added
- Added `node::Node` which contains `start_position` and `end_position` methods to obtain the full range of a node, as well as `surrounding_ignore_tokens` to get surrounding comments/whitespace.
- Added `Punctuated` and `Pair`, replaced lots of uses of `Vec<_>` with `Punctuated<_>`
- Added `ContainedSpan`, a way to represent structures like `(...)` and `[...]`
- Added `Return` and `visit_return`
- Added `Expression::Parentheses`
- Added `Owned` trait to get a `'static` lifetime version of nodes
- Added `TokenKind` to get the kind of a token without any additional data
- Added mutation methods: `set_start_position`, `set_end_position`, `set_token_type` for `TokenReference` objects
- Added `Ast::update_positions` to update all the position structs if you mutate it

### Changed
- Fields of `Token` and `Position` have been made private with public accessors
- Changed signatures from `Token` to `TokenReference`, which dereference to tokens
- Changed `If::else_if` to use a new `Vec<ElseIf>`
- Changed `Value::Function` to also include the function token
- `LastStmt::Return` no longer uses an enum struct, and now uses `Return`
- Changed `visit_do` to use a new `Do` struct instead of `Block`
- Changed `FunctionArgs::Parentheses` to be a struct item

### Fixed
- Fixed unexpected parsing issues with UTF-8 strings

## [0.3.0] - 2019-05-24
### Added
- Added std::fmt::Display and std::error::Error implementations for Error, AstError, and TokenizerError
- Documented all public APIs

### Changed
- Properties are no longer `pub` and now have public accessor methods
- License has been changed from GPL to LGPL

### Fixed
- Fixed `TableConstructorField` parsing not correctly giving separator tokens ([#12](https://github.com/Kampfkarren/full-moon/issues/12))

## [0.2.0] - 2019-05-23
### Added
- Added VisitorMut trait which is passed mutable nodes. Mutation is not completely ready yet and may cause side effects
- Added Visit and VisitMut traits that all nodes implement, as well as some utility types such as Vec<T>

### Removed
- Removed Interpreter in favor of using the Visitor trait directly
- Removed "visitors" feature flag

### Changed
- BinOpRhs is no longer a type alias to a tuple and is instead its own struct

## [0.1.2] - 2019-04-05
### Added
- Added documentation to visitors::Visitor::Interpreter

## [0.1.1] - 2019-04-05
### Fixed
- visitors::Interpreter's constructor is no longer accidentally private

## [0.1.0] - 2019-04-04
### Added
- Initial commit
- Tokenizer and parser
- Visitors
