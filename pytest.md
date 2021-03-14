# Cheat Sheet: PyTest

PyTest is a tool for developing tests more easily than with the standard library's module `unittest`.
To use it first, you need to install it:

```bash
pip install pytest
```

## Testing

Tests are as simple as creating a function. Use the assert keyword to verify something:

```python
def test_add():
    x, y = 1, 2
     
    assert add(x, y) == 3
```

## Test structure

This is personal taste, but I like to structure my test in three blocks: setup, operation, assertion.
I separate each block by a newline, making it easy to identify where it starts and where it ends, not requiring comments that are also visual clutter.
This helps visually identifying each part of the test when visiting it later, for instance

```python
def test_make_pair():
    x, y = 1, 2
     
    pair = make_pair(x, y)
      
    assert pair == x, y
```

## Raises: Expecting exceptions

Sometimes you want to make sure that your code is going to fail with wrong parameters (Errors should never pass silently), in this cases you can use `pytest.raises`:

```python
def test_non_existing_key():
    options = {}
    with pytest.raises(KeyError):
        a = options["verbose"]  # this key clearly does not exists
```

The test above will fail if no KeyError exception is raised in the `raises` context.

## Fixture: Avoiding boilerplate code

When you test a class, you often need to instantiate the same object with the same parameters several times.
To avoid this repetitive code in your test (code that is not in fact testing anything), you can use `pytest.fixture`:

```python
@pytest.fixture
def underage_user():
    return User(name="Joseph", email="xxxx@gmail.com", age=14)
    
def test_serve_ads_underage(underage_user):
    with pytest.raises(ValueError):
        serve_ads(underage_user)
```

By putting an argument with the same name of a fixture (function with `pytest.fixture` decorator), pytest will call the function and pass the result as an argument to the test.

## Parametrize: Parametrizing test

When you want to test the same function with different arguments instead of writing the same tests several times, just changing the call's arguments, you can use the parametrize functionality.

```python
@pytest.mark.parametrize("x,y,expected", [(3, 5, 8), (8, -2, 6), (0, 3, 42)])
def test_add_numbers(x, y, expected):
    assert add(x,y) == expected
```

Similar case application to the fixture functionality: avoid repetitive code and, by doing so, improving readability and reduce the cost of refactoring tests.
They are similar in intent but differ in their essence.
**Fixtures are used to avoid writing the same code for creating the same object several times. In contrast, parametrization avoids writing similar tests several times.**

**Warning**: Don't over-parametrize test!
Group test by expected behaviors.
More tests are better than complex tests (Simple is better than complex).
Don't do this:

```python
@pytest.mark.parametrize(
    "x,y,should_raise_exception,error_type,expected",
    [(3, 5, False, None, 8),
     ("8", "3", False, None, 11),
     (8, "ha", True, ValueError, 6)])
def test_add(x, y, expected):
    if should_raise_exception:
        with pytest.raises(error_type):
            add_with_pre_conversion(x, y)
    else:
        assert add_with_pre_conversion(x, y) == expected
```

In the example above, we test the function's expected behavior, when it should and shouldn't work while specifying what error it should raise.
Imagine trying to figure out why this test fails in a major refactor 3 months after you first wrote the test.

## Mock

Mocking is a functionality from the standard library `unittest` that can also be enjoyed in pytest.
Mock allows us to replace a part of our system with an object that mimics the same behavior but in a controlled way.
Mocks are objects created for testing that have all attributes and functions that you may ask for it, and unless you specify otherwise it will return another mock objects:

```python
>>> m = Mock()
>>> print(m.a)
<Mock name='mock.a' id='140103532413136'>
>>> print(m.is_underage())
<Mock name='mock.is_underage()' id='140103532442528'>
>>> m2 = Mock(age=18)
>>> print(m2.age)
18
```

This is useful for making unit tests to specific parts of your system or eliminating non-deterministic behaviors from tests. You can predefine with more ease what parameters an object has and what returns from its functions.
Mocks are available in `unittest.mock.Mock` or by using the `mocker` fixture from the `pytest-mock` plugin.
I prefer the latter because it undoes the mockings at the end of each test. Also, `pytest-mock` has the `Spy` feature, which allows making call assertions without changing the object/function's original behavior.

```python
def can_serve_targeted_ads(user):
    return not user.is_underage()
    
def test_can_serve_ads_use_age(mocker):
   mock_user = Mock(is_underage=Mock(return_value=False))
   
   assert can_serve_targeted_ads(mock_user) == True
   assert mock_user.is_underage.assert_called_once()
```

In this test, we create a mock user and verify if our `can_serve_ads` tested function is really checking if the user is underage.

**Warning**: notice that at no point are we completely decoupling our tested function from the actual User object.
When you are mocking, you are decoupling parts of your system. If every part of your system work by itself doesn't mean that it will work when you put them together.
Mocking is a double edge sword. At the same time that you can isolating and making sure that it will work for the expected arguments, you are isolating it from the rest of the system, meaning that it may not be receiving its expected arguments.

On top of unit tests that use mocks, make sure to include integration that tests a solid bond between the parts of the system.

Links:

- [Unittest.mock (standard library)](https://docs.python.org/3/library/unittest.mock.html)
- [Pytest-mock](https://docs.python.org/3/library/unittest.mock.html)

## conftest.py

Fixtures are used to avoiding repeated code, but what if you need the same fixture in multiple test files: set then in `conftest.py` file.
The fixture created in `conftest.py` can be used in all tests in that directory (and sub-directories).
There are other uses for the conftest file, but you can learn as you go.

## Marking

## Monkey Patching


