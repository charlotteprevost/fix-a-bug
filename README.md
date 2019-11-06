# 🛠 fix-a-bug

## Setup

Clone this repo.

```bash
git clone https://github.com/datamade/fix-a-bug.git && cd fix-a-bug
```

Install the requirements. (Feel free to use a virtual environment!)

```bash
pip install -r requirements.txt
```

Run the tests.

```bash
pytest -sv
```

## Challenge

The code in `fix_me.py` contains four errors that are preventing the test from
passing. Debug and correct each error. As you identify the errors, fill out
a description of the problem and an explanation of the root cause and your fix
in the section below.

### Error 1

**Description:**

After running the test file, the console provides us with this error:
on line:    `return self.DATA[current_index - 1]`
error:      `IndexError: list index out of range`
This means the index trying to be accessed does not exist within the range of
the list provided. The index range of [23, 45, 56, 67], for example, is 0 to 3
inclusive: anything strictly below 0 or above 3 is out of range.

**Explanation:**

We must handle the case where we are attempting to access a value outside
of a list's index range. Since get_previous() looks 'left,' we can simply
handle the case where there is no previous value (i.e. current index is 0)
by returning 0.
[cf. Fix 1]

### Error 2

**Description:**

We run the test file again and the console still shows the above error:
`IndexError: list index out of range`. We print out current_index for
verification and realize its value is that of the current list element.
We've got to make sure that get_previous() is fed the proper current_index.

**Explanation:**

To make sure that the value being passed in get_previous() is a valid index, we
look at the line where get_previous is called:
`transformed_value = sum(self.get_previous(index), value)`
Then we look at the line where `index` is defined:
`for value, index in enumerate(self.DATA):`
In Python, a `for in` 'loop' coupled with `enumerate` enables us to access
individual items of a list and their indices, the first word after `for` refers
to the list's item's index, the second refers to the item itself (and not the
opposite!).
[cf. Fix 2]

### Error 3

**Description:**

After Fix 1 and Fix 2, we run the test file again. The console chimes:
on line:    `transformed_value = sum(self.get_previous(index), value)`
error:      `TypeError: 'int' object is not iterable`
Python's `sum` function expects an iterable object as an argument (and an
optional starting value) and returns the total sum of elements in the iterable.

**Explanation:**

We need to calculate the sum of the current value and the previous value in the
list, not the sum of all its values (which is what `sum` would do if actually 
fed an iterable object).
[cf. Fix 3]

### Error 4

**Description:**

When running the code after all the above fixes, we realise that the self.DATA
that Transformer points to is that which was inherited by ParentA. It seems
that a child inheriting a field value from multiple parents (with different
values for that field) will simply use the first field value it inherits and
ignore subsequent inherited fields with the same name.

**Explanation:**

There are multiple ways we can approach this depending on how we want to scale
the overall code. Multiple inheritance is (usually) useful when parents have
different fields and methods to offer the child. Here both parents offer the
same field with a different value, and the child by default uses the first.

Fix 4-A-1: If we want `Transformer` to have access to both parents' `DATA`, we
can amend the nomenclature of both fields and refer to them individually.
ParentA's DATA would be `DATA_A`, and ParentB's DATA would be `DATA_B`. Since
the test looks for an output of [4, 9, 11], Transformer methods would use
`self.DATA_B`.

Fix 4-A-2: This ^ becomes tedious if more Parents are added. Another solution
is to assign the DATA value explicitly at the initialization of `Transformer`.
[cf. Fix 4-A-2]

Fix 4-B: This still isn't a great way of ensuring proper scaling though,
because `Transformer`'s logic is reusable. It could instead make more sense for
it to be the Parent and for ParentA/ParentB to be ChildA/ChildB.
The code would then look like this:
```
class Transformer(object):

  def get_previous(self, current_index):
    if current_index == 0:
      return 0
    return self.DATA[current_index - 1]

  def transform(self):
    transformed_data = []
    for index, value in enumerate(self.DATA):
      transformed_value = self.get_previous(index) + value
      transformed_data.append(transformed_value)
    return transformed_data

class ChildA(Transformer):
    DATA = [7, 8, 9]

class ChildB(Transformer):
    DATA = [4, 5, 6]
```
And in this case we would test Child instead of Transformer:
```
def test_transformed_data_is_expected_value(Child, expected_output):
    t = Child()
    assert t.transform() == expected_output
```