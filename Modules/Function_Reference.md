# Single-Row Functions {.unnumbered}

:::{.callout-tip}
Don't worry about memorizing these functions. Use this as a reference when you need it.
:::

Oracle SQL provides numerous built-in functions to manipulate data, perform calculations, and format results.

Below are some of the most commonly used functions, categorized by type.

## Single-Row Functions

:::{.callout-note}
These functions do **not** aggregate data. They operate element-wise on your inputs.
:::

### String Functions

String functions make it easy to manipulate textual data. Here are some common useful string functions.

| Function | Description | Example | Result |
|----------|-------------|---------|--------|
| `LOWER(str)` | Converts string to lowercase | `LOWER('HELLO')` | 'hello' |
| `UPPER(str)` | Converts string to uppercase | `UPPER('hello')` | 'HELLO' |
| `INITCAP(str)` | Capitalizes first letter of each word | `INITCAP('hello world')` | 'Hello World' |
| `SUBSTR(str, start, [length])` | Extracts substring | `SUBSTR('HELLO', 2, 3)` | 'ELL' |
| `LENGTH(str)` | Returns length of string | `LENGTH('HELLO')` | 5 |
| `CONCAT(str1, str2)` | Concatenates two strings (You can also use the `||` operator) | `CONCAT('Hello', ' World')` | 'Hello World' |
| `REPLACE(str, search, replace)` | Replaces all occurrences | `REPLACE('JACK AND JUE','J','BL')` | 'BLACK AND BLUE' |
| `TRIM([chars FROM] str)` | Removes specified characters | `TRIM(' hello ')` | 'hello' |
| `LPAD(str, length, [pad_str])` | Left-pad string | `LPAD('123', 5, '0')` | '00123' |
| `RPAD(str, length, [pad_str])` | Right-pad string | `RPAD('ABC', 5, 'XY')` | 'ABCXY' |

### Numeric Functions

Numeric functions make it easy to manipulate numerical data. Here are some common useful numerical functions.

| Function | Description | Example | Result |
|----------|-------------|---------|--------|
| `ROUND(n, [decimal])` | Rounds number to specified decimal places | `ROUND(125.315, 2)` | 125.32 |
| `TRUNC(n, [decimal])` | Truncates number to specified decimal places | `TRUNC(125.315, 2)` | 125.31 |
| `CEIL(n)` | Returns smallest integer greater than or equal to n | `CEIL(125.3)` | 126 |
| `FLOOR(n)` | Returns largest integer less than or equal to n | `FLOOR(125.3)` | 125 |
| `MOD(m, n)` | Returns remainder when m is divided by n | `MOD(11, 4)` | 3 |
| `ABS(n)` | Returns absolute value | `ABS(-15)` | 15 |
| `POWER(n, m)` | Returns n raised to the power of m | `POWER(3, 2)` | 9 |
| `SQRT(n)` | Returns square root | `SQRT(25)` | 5 |

### Conversion Functions

Conversion functions make it easy to transform data into different types. Here are some common useful conversion functions.

| Function | Description | Example | Result |
|----------|-------------|---------|--------|
| `TO_CHAR(value, [format])` | Converts number or date to string | `TO_CHAR(1234.56, '$9,999.99')` | '$1,234.56' |
| `TO_NUMBER(str, [format])` | Converts string to number | `TO_NUMBER('$1,234.56', '$9,999.99')` | 1234.56 |
| `CAST(expr AS type)` | Converts expression to specified datatype | `CAST('123' AS NUMBER)` | 123 |
| `NVL(expr1, expr2)` | Returns expr2 if expr1 is NULL | `NVL(NULL, 0)` | 0 |
| `NVL2(expr1, expr2, expr3)` | Returns expr2 if expr1 is NOT NULL, else expr3 | `NVL2(NULL, 'A', 'B')` | 'B' |
| `COALESCE(expr1, expr2, ...)` | Returns first non-NULL expression | `COALESCE(NULL, NULL, 'A', 'B')` | 'A' |
