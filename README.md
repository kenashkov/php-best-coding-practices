# PHP best coding practices

#### 1. Always when writing new code use strict type comparison '===' or '!=='
```php
if ($value === $another_value) {}
if ('some_string' === $some_string) {}
```
Existing code must not be modified from '==' to '===' without proper testing as this will introduce bugs.

This also means using strict comparison in the vairous PHP array functions like:
```php
if (in_array($some_value, $some_array, TRUE)) {}
```
List of array functions that support the strict parameter:
- array_keys()
- array_search()
- in_array()

#### 2. When comparing a value or property to a constant always put the constant first. This will avoid the error of having an assignment instead:

```php
if ($object->object_status = $object::STATUS_ACTIVE) { //BUG - this is assignment
}

if ($object::OBJECT_ACTIVE = $object->object_status) { //this will trigger an error
}

//and the correct code is
if ($object::STATUS_ACTIVE === $object->object_status) {
}
```

#### 3. Never use short tags.
Always use <?php or <?=.

#### 4. Never put a closing ?> tag in the file. 
If there is closing tag and by mistake and there is a space or a tab after it this will trigger odd errors about "headers sent".

#### 5. Always give types to the arguments and return values
Even if the value is **mixed** provide this in a comment:
```php
function f1( /* mixed */ $arg1) : void {}
```
Unless the argument can be of any type it is better to provide the expected types in the comment:
```php
function f1( /* int|string|null */ $arg1) : void {}
```
Soon PHP will support [union types](https://github.com/nikic/php-rfcs/blob/union-types/rfcs/0000-union-types-v2.md) and these should be used when available.

#### 6. Always use typed properties
Typed properties are available since PHP 7.4. Whenever possible initialize them:
```php
class c1
{
    private string $name;
    private ?int $age;
    private string $from_planet = 'Earth';
}
```
### 7. Modifying value in a foreach loop

In order to modify the value in the foreach loop the value must be passed by reference:
```php
foreach ($data as &$record) {
    $record['some_key'] = 'some_value';
}
```
If after this loop a second loop is added that uses the same variable name for the value but this time without a reference it will overwrite the last element from the previous loop:
```php
foreach ($data as &$record) {
    $record['some_key'] = 'some_value';
}
foreach ($another_data as $record) {
    $record['some_key'] = 'another_value';//this will actually overwrite the last element from $data
}
```
To avoid this always after the loop unset the variable used for the value and passed by reference.
```php
foreach ($data as &$record) {
    $record['some_key'] = 'some_value';
}
unset($record);
//dont do $record = NULL; as this will actually set the last element of $data to NULL

foreach ($another_data as $record) {
    $record['some_key'] = 'another_value';//this will actually overwrite the last element from $data
}
```
OR
**RECOMMENDED** - follow the [Basic Coding Standard](https://github.com/AzonMedia/php-coding-standard/blob/master/Basic.md) and prefix all local variables that are references with '_':
```php
foreach ($data as &$_record) {
    $record['some_key'] = 'some_value';
}
//so even if there is a second loop that does not pass by reference then it will use $record, not $_record
foreach ($data as $record) {
    print $record['some_key'];
}
//and the last element will not be overwritten
```
This does not require extra code (`unset()`) which can be forgotten.

#### 8. References to array indexes

Whenever are references created to array indexes  it must be ensured that these indexes exist. The following code will produce the odd result of creating the index with a NULL value instead of throwing an error about non existing offset:
```php
$arr = [];
$a =& $arr[22];//this will not trigger an error - instead will produce $arr[22] = NULL;
```
The code must be like:
```php
$arr = [];
$arr[22] = 'something';
$a =& $arr[22];
```
or with a check:
```php
if (array_key_exists(22, $arr)) {
    $a =& $arr[22];
}
```
#### 9. Always use constants and never hardcoded values:
```php
$role_id = 4;
//vs
$role_id = Roles::SALES_MANAGEMENT;
```
Always define constants if there are no already defined ones and name them appropriately.

#### 10. Catching exceptions

Never catch ```\Exception``` or ```\Throwable``` unless you really need that and you know what you are doing. When such an exception is caught a proper logging must be in place (see the "Exceptions handling" section below).

#### 11. Exceptions handling

Whenever an exception is caught the catch block must not be empty - it must have either a comment explaining why the block is empty (meaning why this exception is safe to be ignored and not have handling code) or a logger/proper handling code.

#### 12. else-if and switch-case
The else-if blocks must end with an non empty else. Even if the logic does not require an else-if to have a final else which should contain:
- remain empty and just have a comment why it is safe to remain empty
- contain a logger
- throw a LogicException (for an unexpected value)
The same is valid for the switch-case - it must have a **default:** entry even if such is not required by the application logic.

#### 13. Future proof IF statements

Avoid declaring variables in an IF statement even if in both IF & ELSE the variable is declared.
```php
if ($cond1) {
    $var1 = 1;
} else {
    $var = 2;
}
```
The above code is not future proof as someone may add an `elseif` and forget to define the variable.
```php
if ($cond1) {
    $var1 = 1;
} elseif($cond2) {
    //does something else
} else {
    $var = 2;
}
```
The correct way is:
```php
$var = 2;//some default value
if ($cond1) {
    $var1 = 1;
}
```
This way even if someone adds a an else or an **elseif** the variable will be defined.
The code should be always thought of and planned so that there is a default value (this is the value that is set in the **else** block or in the above case - predefined before the **if**).
If this is not possible and it is preferred to have an undefined variable error than a wrong value then it is acceptable to have the variable defined in each else/if.

### 16. String comparison

The comparison of strings should be always done by converting both of them to lower case (unless the case matters).
This is valid even for hardcoded strings (because a hardcoded string like **'false'** may get converted to **'FALSE'** during code replace or refactoring be it manual or automatic.

```php
if ( $node === 'true') {
}
```
Needs to become:
```php
if ( strtolower($node) === strtolower('true') ) {
}
```

### 17. Closures

If the closure will not need to use $this it should be declared as static one:
```php
$Function = static function() {
//do something
};
```
This is done so that it is avoided having one more reference to $this. 
Tthis may prevent object destruction based on the scope as this reference may become a circular reference if this closure is assigned to an object property.
In this case it will be collected when the GC is triggered.