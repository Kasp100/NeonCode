# NeonCode: A programming language
A programming language designed for safety and quick development.

This is the documentation repository that describes the language.

# Language Features
NeonCode is object-oriented and general-purpose. It features types, interfaces, classes, etc.

The convention is to use snake_casing for everything (and UPPERCASE_SNAKE_CASING for constants).  
However, there's a special character (`#`) to distinguish pass-by-value and pass-by-reference types.

## Types of types

### Classes (`class`)
Classes are pass-by-reference types (or simply, reference types).

There are three types of classes:

#### Default (no extra keyword)
Non-serialisable, non-copyable.

#### Serialisable (keyword `serialisable` after type name in declaration)
Objects of a serialisable type can be transferred to the outside and back. For example, you may want to write an object to a file and be able to read it again later.

Objects of a serialisable type can be made without a constructor: if they are deserialised, the fields are initialised directly with the incoming data.

In serialisable classes, every field must also be of a serialisable type (all primitives are serialisable types) and cannot have `shared` or `external` fields.  

> Developers **cannot trust** the contents of objects of serialisable types.  
It is recommended to perform validation and transfer the data to a "safe" type that enforces constraints and ensures integrity.

It is the serialiser's task to ensure data integrity over the medium if needed.

Objects of a serialisable class are freely copyable using `copy`.

#### Copyable (keyword `copyable` after type name in declaration)
Copyable types allow their instances to be duplicated using the `copy` keyword.  
In copyables:
- Every field must either be `shared` or have a copyable type.
- `shared` fields retain their references during copying.
- All other fields are deeply copied.

> If custom copying behavior is required, you cannot use these keywords and must implement copying manually.

### Primitives and pass-by-value
Primitives are pass-by-value types (meaning they're copied rather than passed by a reference) and can be serialised.

All primitive types have their names starting with `#` to signal pass-by-value. Furthermore, any pass-by-value/primitive type **must** have its name start with `#`.

|  Type Name  |   Alias   |                                     Description                                     |
|-------------|-----------|-------------------------------------------------------------------------------------|
| `#uint`     | `#nat`    | Unsigned integer with its size optimised for the architecture being compiler for *  |
| `#int`      |           | Signed integer with its size optimised for the architecture being compiler for *    |
| `#char`     |           | 32 bit single character in UTF-32                                                   |
| `#bool`     | `#bit`    | Boolean, with values `true` or `false`                                              |
| `#fp32`     | `#float`  | Floating point number with single precision (32 bit)                                |
| `#fp64`     | `#double` | Floating point number with double precision (64 bit)                                |
| `#uint64`   | `#nat64`  | 64 bit unsigned integer                                                             |
| `#int64`    | `#long`   | 64 bit signed integer                                                               |
| `#uint32`   | `#nat32`  | 32 bit unsigned integer                                                             |
| `#int32`    |           | 32 bit signed integer                                                               |
| `#uint16`   | `#nat16`  | 16 bit unsigned integer                                                             |
| `#int16`    |           | 16 bit signed integer                                                               |
| `#uint8`    | `#byte`   | 8 bit unsigned integer                                                              |
| `#int8`     |           | 8 bit signed integer                                                                |

(* For example, on a 64 bit architecture like x86-64, the size will be 64 bits.)

All built-in primitive types are numeric except for booleans (`#bool`) and characters (`#char`).  
Numeric means that arithmetic operations can be performed on them by default.
(You can also create custom grammar allowing you to create custom numeric types, whether they're primitive or not.)

Each of the numeric types can be written in decimal, hexadecimal, and binary notation.
E.g. `21` = `0x15` = `0b10101`

Booleans can be `true` or `false`.
Characters can be any character in UTF-32.

---

## Immutability by Default
When specifying a type, you need to prefix it with `mut:` if you intend on mutating the object or array.  
However, an object or array may be mutated from another scope if it is `shared` and the type itself is mutable.  
Developers need to clearly state their intentions by using `mut:` or not.

For mutable types:
- Conversion from mutable to immutable occurs automatically when assigning to a reference **without `mut:`**.
- For copyable types, `copy` produces a mutable copy.

However you cannot use `mut:` on primitive types. Their "mutating" is always through reassignment.

---

## Const by Default
Variables and fields cannot be reassigned by default.

### Keyword `var`
To enable reassignment after the initial assignment of a variable or field, use `var`.

> NeonCode doesn't use "final" to ensure a reference or value isn't changed. Instead, `var` is used to explicitly allow reassignment.

`var` works with both **pass-by-value and pass-by-reference types**.

---

## Field Types

### Default (no keyword)

1. The field cannot be reassigned after its initial assignment (lacking `var`).
2. For **mutable** reference types:
	- The container (an object of this class) has unique ownership of the referenced object.
	- Mutation of the field or the referenced object is considered mutating the container.
	- Assigning a shared reference to this field requires copying the object, making it a unique reference
3. For **immutable** reference types, the referenced object or array may have multiple references to it.

---

### Shared (`shared`) for reference types
The `shared` keyword can be used with **mutable reference types** and not primitive types.

- `shared` allows an object or array to be referenced and potentially mutated from multiple locations.  
- Mutating shared objects reflects in all references but is **not** considered mutating their containing objects.

> Use `shared` cautiously to avoid unintended side effects.

---

### Unique Reference (without `shared`)

For references without `shared`:
- The compiler ensures the object or array is uniquely owned by the containing object.
- Mutating the referenced object or array is considered mutating the container.

That said, each object can reference to itself through `this` which can be passed.

---

### External (`external`) for reference types
The `external` keyword can be used with reference types and not primitive types.

- An `external` field is treated as a unique reference where mutations are **not** considered mutations of the containing object.
- `external` cannot be combined with `shared`, as the above behaviour is the default for shared references.

---

### Parameters
Parameters default to `shared` since they originate outside the scope.  
To prevent unintended side effects, copy the object if its type supports it.

You can also directly copy the object or array inside the parameters by stating `copy` before the type specification.

---

### Constants (`const`)
Constants are static fields in a class, non-reassignable, and immutable.
Conventionally you make them all caps.

> Beware that using constants makes your program less flexible. A field may be more effective.

---

## Array types

### Compile-Time Static Length Array
This is the most basic type of array and is the most performant.

**Syntax**: `array<type type, #uint length>`

This is a built-in type.

For example, `array<#char,16>` is an array of length 16 containing characters. It is useful to create an alias if a fixed length array suits your needs.
In case of an array of a mutable type, the array uses `shared` semantics: there is no copying involved in setting any item.

**A bit about type parameters**: Type parameters like `#uint length` in `array` must be determined at **compile time**. E.g. they cannot come from a field or a variable, with the exception of **constants**.

### Runtime Static Length Array
The length of this type of array is determined at **runtime**.

**Syntax**: `array<type type>`

This is also a built-in type.

Once initialised, its size cannot be changed though its values can.

---

## Mutable declarations
`mut` has two usages:

### 1. To allow a method to mutate the data of the object.

`mut` is placed after the closing curly bracket, making the method **mutating**.
This allows mutating operations to occur. This must also be declared in interfaces.

### 2. To allow a class to have mutating methods.

`mut` is placed after the class name, making it a mutable type.

---

## Developer Guidelines

1. Avoid using `var` if the field or variable will never be reassigned.
2. Avoid using `shared` if the object or array is never referenced elsewhere.
3. Avoid using `mut:` if the object or array will never be mutated from the scope its referenced by.
4. Avoid using `mut` if the method does not mutate the object, or the class is an unmutable type.

---

## Custom Grammar

NeonCode allows custom grammar using the `grammar` keyword. This indicates the definition of a **grammar set**. Grammar sets are scoped like classes.  
Custom grammar will be parsed if the `parse` keyword is used. This makes the parser take the custom grammar into account.  
It is also possible to use grammar from other packages (just like imports, e.g. `parse math::big_decimal_arithmetic;`).  

A **rule** inside grammar sets is a special kind of **function**.

Rules follow the following syntax:
- Optionally start with their **subordination**, an integer to determine the precedence of each rule. (Where **0** is the most precedence / least subordination.)
- Followed by their **return type**.
- Followed by their individual grammar, where curly brackets indicate parameters to be passed.
- Ending in their **function body**

It is important to note the limitation of custom grammar:  
The **grammar set** itself needs to be parsed before being used with `parse`.

### Example
```
pkg temperature;

grammar temperature_notation
{
	0 temperature (#double value)°K
	{
		ret temperature(value);
	}

	0 temperature (#double value)°C
	{
		ret temperature(value+273.15);
	}

	0 temperature (#double value)°F
	{
		ret temperature((value+459.67)/1.8);
	}

}

grammar temperature_arithmetic
{
	1 temperature (temperature a)*(temperature b)
	{
		ret temperature(a.get_value_kelvin()*b.get_value_kelvin());
	}

	1 temperature (temperature a)/(temperature b)
	{
		ret temperature(a.get_value_kelvin()/b.get_value_kelvin());
	}

	2 temperature (temperature a)+(temperature b)
	{
		ret temperature(a.get_value_kelvin()+b.get_value_kelvin());
	}

	2 temperature (temperature a)-(temperature b)
	{
		ret temperature(a.get_value_kelvin()-b.get_value_kelvin());
	}

}

parse temperature_arithmetic;
parse temperature_notation;

class temperature serialisable
{
	#double value_kelvin;

	constructor(#double set_value_kelvin)
	{
		value_kelvin = set_value_kelvin;
	}

	public #double get_value_kelvin()
	{
		ret value_kelvin;
	}

}
```

---

## NeonCode Examples

```
pkg main;

import std::date;

public class person
{
	date birthdate; // Assuming that a person cannot change their birthdate.
	var string name; // Assuming that a person may change their name.

	public constructor(date init_birthdate, string init_name)
	{
		birthdate = init_birthdate;
		name = init_name;
	}

	public date get_birthdate()
	{
		ret birthdate;
	}

	public void set_name(string new_name) mut
	{
		name = new_name;
	}

	public string get_name()
	{
		ret name;
	}

}
```

```
pkg main;

public class bank_account
{
	string name;
	mut:list<transaction> transactions_history;
	shared bank keeper;

	public constructor(bank set_keeper)
	{
		keeper = set_keeper;
		transactions_history = ();
	}

	public void perform_transaction(transaction tr) mut
	{
		transactions_history.add(tr);
	}

	public list<transaction> get_transactions_history()
	{
		ret transactions_history; // An immutable view can always be returned
	}

	public #float get_balance()
	{
		var #float balance = 0;

		foreach transactions_history (transaction tr)
		{
			balance += tr.get_balance_diff(this);
		}

		ret balance;
	}

}

public class transaction
{

	shared bank_account source;
	shared bank_account destination;

	#float amount;

	string<255> comment;

	public constructor(bank_account set_source, bank_account set_destination, #float set_amount, string<255> set_comment) {
		source = set_source;
		destination = set_destination;
		amount = set_amount;
		comment = set_comment;
	}

	public #float get_balance_diff(bank_account for_account)
	{
		if(for_account == source)
		{
			ret -amount;
		}
		if(for_account == destination)
		{
			ret amount;
		}
		ret 0;
	}

}


public class bank
{
	var string name;

	constructor(string set_name)
	{
		name = set_name;
	}

	public string get_name()
	{
		ret name;
	}

}

```

```
pkg main;

public class s_license_plate serialisable
{
	array<#char,7> chars;

	public constructor(license_plate original)
	{
		chars = original.get_chars();
	}

	public optional<license_plate> create()
	{
		try
		{
			ret optional::of(license_plate(chars))
		}
		catch (invalid_char e)
		{
			ret optional::empty();
		}

	}

}

public class license_plate
{
	const #nat LENGTH = 7;
	const array<#char> VALID_CHARS = ('A','B','C'); // To be expanded

	array<#char,LENGTH> chars;

	public constructor(array<#char,LENGTH> set_chars) throws invalid_char, invalid_length
	{
		mut:array_builder<#char> validated_chars = array_builder::with_capacity(LENGTH);

		foreach set_chars (#char c)
		{
			if(!is_valid(c))
			{
				throw invalid_char(c);
			}
			else
			{
				validated_chars.add(c);
			}
		}

		chars = array::from_builder(validated_chars); // This may throw invalid_length.

	}

	#bool is_valid(#char check_char)
	{
		foreach VALID_CHARS (#char c)
		{
			if(c == check_char)
			{
				ret true;
			}
		}

		ret false;
	}

}


```
