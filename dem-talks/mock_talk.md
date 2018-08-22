# For Those About to Mock

## Mock or Stub?

Lots of room for confusion and conflicting viewpoints.

One way to think about it:

* Stubs don’t do anything, they just return a value
    * eg, a response body we want to test parsing, or
    * a status code that we need for a method to work
* Mocks don’t do anything either, but they pretend to!
    * specifically, they can pretend to provide an interface
    * if our test doesn’t care what the interface does, then who cares, right?

Another perspective:

* Stubs are used for state testing (TDD)
* Mocks are used for behavior testing (BDD)

Mocks are especially useful dynamic objects that can behave for all intents and purposes like real objects.

They return values on being called, can produce iterations, and even throw exceptions.  Additionally, they record calls made to them so they can be examined later.


## When to Mock

Mocks help you when the unit under test...

* ...calls out to libraries you don't own (or maybe have disowned).
* ...calls into the system to do something to files.
* ...does something internet-y.
* ...does something database-y.
* ...has branching logic that is triggered by an Exception.
* ...needs to talk to something that doesn’t exist yet, but you have a spec for.

## Mocking in Python

### Basic Mocking

Make a mock from Mock

```python
>>> from mock import Mock
>>> basic_mock = Mock()
>>> basic_mock
<Mock id='4404923280'>
```

Ask a mock for an attribute and it won't complain.  You can even assign it a value.

```python
>>> basic_mock.pizza
<Mock name='mock.pizza' id='4407112848'>
>>> basic_mock.pizza = 'mystic'
>>> basic_mock.pizza
'mystic'
```

Why not a method call?

```python
>>> basic_mock.brexit()
<Mock name='mock.brexit()' id='4407113296'>
>>> basic_mock.brexit.return_value = True
>>> basic_mock.brexit()
True
```

Let's make it behave badly.

```python
>>> basic_mock.nu_metal()
<Mock name='mock.nu_metal()' id='4407113424'>
>>> basic_mock.nu_metal.side_effect = AttributeError("Not even once.")
>>> basic_mock.nu_metal()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Library/Python/2.7/site-packages/mock/mock.py", line 1062, in __call__
    return _mock_self._mock_call(*args, **kwargs)
  File "/Library/Python/2.7/site-packages/mock/mock.py", line 1118, in _mock_call
    raise effect
AttributeError: Not even once.
```

Side effects can also be used to produce a series of iterations.

```python
>>> mopes = ['morrissey', 'nick cave', 'conor oberst']
>>> basic_mock.mopes.side_effect = mopes
>>> basic_mock.mopes()
'morrissey'
>>> basic_mock.mopes()
'nick cave'
>>> basic_mock.mopes()
'conor oberst'
```

Mocks don't have to return anything when called.

```python
>>> basic_mock.cheese('log')
<Mock name='mock.cheese()' id='4407652752'>
>>> basic_mock.cheese('shoppe')
<Mock name='mock.cheese()' id='4407652752'>
>>> basic_mock.cheese('camembert')
<Mock name='mock.cheese()' id='4407652752'>
```

Mocks remember what they were called with.

```python
>>> basic_mock.mock_calls
[call.brexit(),
 call.nu_metal(),
 call.mopes(),
 call.mopes(),
 call.mopes(),
 call.cheese('log'),
 call.cheese('shoppe'),
 call.cheese('camembert')]
>>> basic_mock.cheese.mock_calls
[call('log'), call('shoppe'), call('camembert')]
>>> basic_mock.cheese.call_args
call('camembert')
>>> basic_mock.cheese.call_args_list
[call('log'), call('shoppe'), call('camembert')]
>>> basic_mock.cheese.call_count
3
```

Call objects can also be pulled apart a bit.

```python
>>> basic_mock.pizza('pepperoni', 'pineapple', 'raisins', cheese='extra', sauce=False)
<Mock name='mock.pizza()' id='4461223568'>
>>> basic_mock.pizza.call_args
call('pepperoni', 'pineapple', 'raisins', cheese='extra', sauce=False)
>>> cargs, ckwargs = basic_mock.pizza.call_args
>>> cargs
('pepperoni', 'pineapple', 'raisins')
>>> ckwargs
{'cheese': 'extra', 'sauce': False}
```

The Mock class has some handy asserts surrounding calls.

```python
>>> basic_mock.brexit.assert_called()
>>> basic_mock.bremain.assert_called()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Library/Python/2.7/site-packages/mock/mock.py", line 906, in assert_called
    raise AssertionError(msg)
AssertionError: Expected 'bremain' to have been called.

>>> basic_mock.bremain.assert_not_called()
>>> basic_mock.brexit.assert_not_called()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Library/Python/2.7/site-packages/mock/mock.py", line 897, in assert_not_called
    raise AssertionError(msg)
AssertionError: Expected 'brexit' to not have been called. Called 1 times.

>>> basic_mock.nu_metal.assert_called_once()
>>> basic_mock.cheese.assert_called_once()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Library/Python/2.7/site-packages/mock/mock.py", line 915, in assert_called_once
    raise AssertionError(msg)
AssertionError: Expected 'cheese' to have been called once. Called 3 times.

>>> basic_mock.nu_metal.assert_called_with()
>>> basic_mock.cheese.assert_called_with('camembert')
>>> basic_mock.nu_metal.assert_called_with('camembert')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Library/Python/2.7/site-packages/mock/mock.py", line 937, in assert_called_with
    six.raise_from(AssertionError(_error_message(cause)), cause)
  File "/Library/Python/2.7/site-packages/six.py", line 718, in raise_from
    raise value
AssertionError: Expected call: nu_metal('camembert')
Actual call: nu_metal()
```

### Magic Mocking

Make a mock from MagicMock

```python
>>> from mock import MagicMock
>>> magic_mock = MagicMock()
>>> magic_mock
<MagicMock id='4404952208'>
```

OK, but why would you do that?

Because you need to call magic methods, of course!

```python
>>> int(basic_mock)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: int() argument must be a string or a number, not 'Mock'

>>> int(magic_mock)
1

>>> with basic_mock as what:
...     what.what()
...
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: __exit__

>>> with magic_mock as what:
...     what.what()
...
<MagicMock name='mock.__enter__().what()' id='4405413328'>
```

Or perhaps you want to mock a generator.

```python
>>> faux_gen = MagicMock()
>>> faux_gen.__iter__.return_value = iter('moo')
>>> faux_gen.__iter__()
<iterator object at 0x106b1e850>
>>> list(faux_gen)
['m', 'o', 'o']
>>> faux_gen.__iter__().next()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

### Spec Mocking

Make a mock from another object (or an instance of an object)!

Using an object as a spec for a mock automatically makes a mock with all of that object's attributes and methods... which are also mocks.

```python
from mock import Mock
import requests

>>> mock_requests = Mock(requests)
>>> dir(requests)
['ConnectTimeout', 'ConnectionError', 'DependencyWarning', 'FileModeWarning', 'HTTPError', 'NullHandler', 'PreparedRequest', 'ReadTimeout', 'Request', 'RequestException', 'Response', 'Session', 'Timeout', 'TooManyRedirects', 'URLRequired', '__author__', '__build__', '__builtins__', '__copyright__', '__doc__', '__file__', '__license__', '__name__', '__package__', '__path__', '__title__', '__version__', 'adapters', 'api', 'auth', 'certs', 'codes', 'compat', 'cookies', 'delete', 'exceptions', 'get', 'head', 'hooks', 'logging', 'models', 'options', 'packages', 'patch', 'post', 'put', 'request', 'session', 'sessions', 'status_codes', 'structures', 'utils', 'warnings']
>>> dir(mock_requests)
['ConnectTimeout', 'ConnectionError', 'DependencyWarning', 'FileModeWarning', 'HTTPError', 'NullHandler', 'PreparedRequest', 'ReadTimeout', 'Request', 'RequestException', 'Response', 'Session', 'Timeout', 'TooManyRedirects', 'URLRequired', '__author__', '__build__', '__builtins__', '__copyright__', '__doc__', '__file__', '__license__', '__name__', '__package__', '__path__', '__title__', '__version__', 'adapters', 'api', 'assert_any_call', 'assert_called_once_with', 'assert_called_with', 'assert_has_calls', 'attach_mock', 'auth', 'call_args', 'call_args_list', 'call_count', 'called', 'certs', 'codes', 'compat', 'configure_mock', 'cookies', 'delete', 'exceptions', 'get', 'head', 'hooks', 'logging', 'method_calls', 'mock_add_spec', 'mock_calls', 'models', 'options', 'packages', 'patch', 'post', 'put', 'request', 'reset_mock', 'return_value', 'session', 'sessions', 'side_effect', 'status_codes', 'structures', 'utils', 'warnings']
```

This only goes one level deep, so if you need the sub-object to be as fully mocked, you have to add it in yourself.

```python
>>> dir(requests.status_codes)
['LookupDict', '__builtins__', '__doc__', '__file__', '__name__', '__package__', '_codes', 'code', 'codes', 'title', 'titles']
>>> dir(mock_requests.status_codes)
['assert_any_call', 'assert_called_once_with', 'assert_called_with', 'assert_has_calls', 'attach_mock', 'call_args', 'call_args_list', 'call_count', 'called', 'configure_mock', 'method_calls', 'mock_add_spec', 'mock_calls', 'reset_mock', 'return_value', 'side_effect']
```

But the sub-objects do behave like proper mocks.  They absorb and record calls to the object, and can have return values and side effects.

```python
>>> mock_requests.get
<Mock name='mock.get' id='4406393104'>
>>> mock_requests.get('sandwich')
<Mock name='mock.get()' id='4406392976'>
>>> mock_requests.get.call_args
call('sandwich')
```

Mocks specced off an object also pass instance tests against that object!

```python
>>> str_mock = Mock(str)
>>> isinstance(str_mock, str)
True
>>> isinstance(str_mock, Mock)
True
```

### More Esoteric Mocks

If you need a mock object that *shouldn't* be called, use NonCallableMock:

```python
>>> from mock import NonCallableMock
>>> aloof_mock = NonCallableMock()
>>> aloof_mock.return_value = 'coy'
>>> aloof_mock()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'NonCallableMock' object is not callable
```

If you need a mock object with \_\_get\_\_ and \_\_set\_\_ methods, use PropertyMock.  Note that these have some rough edges.  Specifically, setters and getters don't *actually* work:

```python
>>> from mock import PropertyMock
>>> props = PropertyMock(return_value='Word.')
>>> type(basic_mock).props = props
>>> basic_mock.props
'Word.'
>>> basic_mock.props = 'Word is bond.'
>>> basic_mock.props
'Word.'
>>> basic_mock.props.mock_calls
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'str' object has no attribute 'mock_calls'
>>> props.mock_calls
[call(), call('Word is bond.'), call()]
```

### Patching Mocks

The monkey way:

This test definitely passes, but we can mess things up really easily if we forget the teardown.

```python
import unittest
import requests
from taisync import synchronizer, errors
from taisync.tests.mocks.request_mocks import REQUEST_MOCKS

class Test_GetBuildResult(unittest.TestCase):
    """Tests for get_build_result()."""

    def setUp(self):
        self.real_requests = requests
        self.mock_requests = REQUEST_MOCKS['requests']
        self.response = REQUEST_MOCKS['response']
        synchronizer.requests = self.mock_requests  # here be the monkey patch...

    def tearDown(self):
        synchronizer.requests = self.real_requests  # ...but if we don't do this, woe betide unto us!
        self.response.status_code = None

    def test_unauthorized_status_code_returns_correct_error(self):
        """Tests error thrown by validate_request_status() when user is unauthorized."""
        # local setup
        self.response.status_code = 401
        synchronizer.requests.get.return_value = self.response

        # runtime
        with self.assertRaises(RuntimeError) as raises:
            synchronizer.get_build_result(self.build_key)
        self.assertEqual(errors.UNAUTHORIZED_USER_ERROR, str(raises.exception))
```

But wait! There's another way!

The decorator way:

This test passes, and the decorator takes care of the mess of monkey patching!

```python
import unittest
from mock import patch
from taisync import synchronizer, errors
from taisync.tests.mocks.request_mocks import REQUEST_MOCKS

@patch('taisync.synchronizer.requests', REQUEST_MOCKS['requests'])
class Test_GetBuildResult(unittest.TestCase):
    """Tests for get_build_result()."""

    def setUp(self):
        self.response = REQUEST_MOCKS['response']

    def tearDown(self):
        self.response.status_code = None

    def test_unauthorized_status_code_returns_correct_error(self):
        """Tests error thrown by validate_request_status() when user is unauthorized."""
        # local setup
        self.response.status_code = 401
        synchronizer.requests.get.return_value = self.response

        # runtime
        with self.assertRaises(RuntimeError) as raises:
            synchronizer.get_build_result(self.build_key)
        self.assertEqual(errors.UNAUTHORIZED_USER_ERROR, str(raises.exception))
```


## Anatomy of a Mock

Let's take a closer look at setting up that requests mock.

```python
# request_mocks.py

from mock import Mock

import requests
from requests import Response

REQUEST_MOCKS = {
    'requests': Mock(requests),
    'response': Mock(Response)
}

```

Yeah, really.  That was it.

OK, fine.  It's not always that easy.  How about something more high maintenance?

```python
# jira_mocks.py

from copy import deepcopy

from mock import Mock
from jira.client import JIRA
from jira.resources import Issue, IssueType, Status, Project

from taisync import config, issuestates

def setup_mocks():
    """Configures mock objects from the jira library for unit tests."""
    jira = Mock(JIRA)

    issue_type = Mock(IssueType)
    project = Mock(Project)
    status = Mock(Status)
    fields = Mock(status=status, issue_type=issue_type, project=project)
    raw = {
        'fields': {
            config.BUILD_KEY_FIELD: None,
            config.JOB_KEY_FIELD: None,
            config.BAMBOO_URL_FIELD: None,
            config.ERROR_FIELD: None
        }
    }

    issue = Mock(Issue)
    issue.fields = fields

    not_implemented_issue = Mock(issue)
    not_implemented_issue.fields.status.name = issuestates.NOT_IMPLEMENTED
    not_implemented_issue.key = u"NOT-IMPL"
    not_implemented_issue.raw = deepcopy(raw)

    # etc etc for every issue type...

    script_error_issue = Mock(issue)
    script_error_issue.fields.status.name = issuestates.SCRIPT_ERROR
    script_error_issue.key = u"SCRPT-ERR"
    script_error_issue.raw = deepcopy(raw)
    script_error_issue.raw['fields'][config.ERROR_FIELD] = u"You done goofed."

    mocks = {
        'jira': jira,
        'issues': {
            'not_implemented': not_implemented_issue,
            'in_progress': in_progress_issue,
            'not_running': not_running_issue,
            'passing': passing_issue,
            'passing_with_warnings': passing_with_warnings_issue,
            'failing': failing_issue,
            'script_error': script_error_issue
        },
        'transitions': {
            # ... a big hairy stub of a jira transactions dict ...
        }

    for issue in mocks['issues'].values():
        issue.fields.summary = u'Death Laser Test'
        issue.fields.project = u'DEATHSTAR'

    return mocks

JIRA_MOCKS = setup_mocks()
```

So yeah, it's not always cheap.  But these mocks could then be used in any test for that library.  The important thing is knowing when it's worth going to all this trouble.  If there's a lower effort way to run the same test with the same results, do that instead.


## A Practical Example

In this test against some external processing library, apply_preset() crashes because it tries to read the get_name attribute of a supplied `processor` instance:

AttributeError: MockProcessor instance has no attribute 'get_name'

```python
# processor/process_file_test.py

import unittest

from processor.preset import Preset

class ApplyPresetTest(unittest.TestCase):
    def test_alphabetical_order(self):
        """Make sure parameters are set in alphabetical order.

        The legacy ProcessFile did this, and some processing has parameter
        interdependency and requires this to get the same results."""

        class MockProcessor:  # note: not a Mock!
            def __init__(self):
                self.param_sets = []

            def set_param(self, key, _):
                self.param_sets.append(key)

        preset = Preset(
            'instance',
            {'a': 1, 'b': 2, 'c': 3, 'adsd': 4, 'iajsdjasj92929': 5},
            {})

        mock_processor = MockProcessor()
        apply_preset(mock_processor, preset)
```

Instead of stubbing out everything, let's give it a futureproof fix with a mock!

```python
from processor import Processor

class ApplyPresetTest(unittest.TestCase):
    def test_alphabetical_order(self):
        def set_param(key, _):
            """Appends its 1st arg to the param_sets attribute when called."""
            MockProcessor.param_sets.append(key)

        MockProcessor = Mock(Processor)
        MockProcessor.get_param_type.return_value = ParamType.ENUM
        MockProcessor.param_sets = []
        MockProcessor.get_param_range.return_value = (0, 6)
        MockProcessor.set_param = set_param
        MockProcessor.get_name.return_value = 'instance'

        preset = Preset(
            'instance',
            {'a': 1, 'b': 2, 'c': 3, 'adsd': 4, 'iajsdjasj92929': 5},
            {})

        mock_tdsp = MockProcessor
        apply_preset(mock_tdsp, preset)

        self.assertSequenceEqual(
            sorted(preset.parameters.keys()), mock_tdsp.param_sets)
```

Hooray!  The mock does mock things and we get to test what we actually care about: that apply_preset behaves as a user expects.

## Gotchas

### Teardowns are SUPER important for shared mocks

```python
import unittest
from mock_objects import common_mock

class TestA(unittest.TestCase):
    def setUp(self):
        self.mock = common_mock

    def test_common_mock_called(self):
        self.mock()
        assert self.mock.called  # True

class TestB(unittest.TestCase):
    def setUp(self):
        self.mock = common_mock

    def test_common_mock_not_called(self):
        assert not self.mock.called  # False... uh oh.

class TestC(unittest.TestCase):
    """branching_function calls mock.thing, which throws an error if a certain branch is followed."""
    def setUp(self):
        self.mock = common_mock
        self.mock.thing.side_effect = KeyError

    def test_error_thrown_in_branching_function(self):
        with self.assertRaises(KeyError):
            branching_function(self.mock, branch=True)  # this test passes

    def test_error_not_thrown_in_branching_function(self):
        branching_function(self.mock, branch=False)  # KeyError
```

The fix is super simple: add a teardown...

```python
def tearDown(self):
    self.mock.reset_mock()
    self.mock.thing.side_effect = None
```

...and put side effects closer to the test (but still tear them down, because they can leak into other test classes):

```python
class TestC(unittest.TestCase):
    def setUp(self):
        self.mock = common_mock

    def test_error_thrown_in_branching_function(self):
        self.mock.thing.side_effect = KeyError
        with self.assertRaises(KeyError):
            branching_function(self.mock, branch=True)
```

### Mocks already have name attributes

```python
>>> class Dog(object):
...     name = 'Mr. Sherman'
...
>>> new_dog = Dog()
>>> new_dog.name
'Mr. Sherman'
>>> mockdog = Mock(Dog)
>>> mockdog.name
<Mock name='mock.name' id='4476128912'>
```

Easy to work around, though:

```python
>>> mockdog.name = 'Benji'
>>> mockdog.name
'Benji'
```


### Mock asserts have no memory

Check yr asserts before you wreck yr asserts:

```python
>>> mistermock = Mock(type)
>>> mistermock
<Mock spec='type' id='4476127056'>
>>> mistermock.pants = Mock()
>>> mistermock.pants('green')
<Mock name='mock.pants()' id='4476128784'>
>>> mistermock.pants('jeans')
<Mock name='mock.pants()' id='4476128784'>
>>> mistermock.pants.assert_called_with('green')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Library/Python/2.7/site-packages/mock.py", line 835, in assert_called_with
    raise AssertionError(msg)
AssertionError: Expected call: pants('green')
Actual call: pants('jeans')
```
...wat?

Here's what's up:

```python
>>> from mock import call
>>> mistermock.pants.call_args
call('jeans')
>>> mistermock.pants.mock_calls
[call('green'), call('jeans')]
>>> mistermock.pants.call_args_list
[call('green'), call('jeans')]
>>> assert call('green') in (mistermock.pants.mock_calls or mistermock.pants.call_args_list)
>>> mistermock.pants.assert_has_calls([call('green')])
```

### Deriving mocks ain't easy

tl;dr: you can't copy mocks

```python
>>> mockstring = Mock(str)
>>> mockstring
<Mock spec='str' id='4476126160'>
>>> mockstring.voodoo = Mock()
>>> mockstring.voodoo
<Mock name='mock.voodoo' id='4476125776'>
>>> mockstring.voodoo.bond = 'Live and Let Die'
>>> mockstring.voodoo.bond
'Live and Let Die'
>>> mockstring.voodoo.doll.return_value = "CURSES!"
>>> mockstring.voodoo.doll
<Mock name='mock.voodoo.doll' id='4476126416'>
>>> mockstring.voodoo.doll()
'CURSES!'
>>> mockcopy = copy.copy(mockstring)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/copy.py", line 96, in copy
    return _reconstruct(x, rv, 0)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/copy.py", line 343, in _reconstruct
    y.__dict__.update(state)
AttributeError: 'str' object has no attribute '__dict__'
```

...uh oh.

```python
>>> mockcopy = copy.deepcopy(mockstring)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/copy.py", line 190, in deepcopy
    y = _reconstruct(x, rv, 1, memo)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/copy.py", line 343, in _reconstruct
    y.__dict__.update(state)
AttributeError: 'str' object has no attribute '__dict__'

```

...uh oh uh oh.

```python
>>> mockcopy = Mock(mockstring)
>>> mockcopy.voodoo
<Mock name='mock.voodoo' id='4476126800'>
```

...phwew.

```python
>>> mockcopy.voodoo.nation
<Mock name='mock.voodoo.nation' id='4476126864'>
```

......uh oh.

```python
>>> mockcopy.voodoo.doll
<Mock name='mock.voodoo.doll' id='4476126032'>
>>> mockcopy.voodoo.doll()
<Mock name='mock.voodoo.doll()' id='4476126096'>
```
...dangit.

But you can copy MagicMocks:

```python
>>> from mock import MagicMock
>>> magic = MagicMock()
>>> magic.hat = 'rabbit'
>>> magic.hat
'rabbit'
>>> magic.trick = MagicMock()
>>> magic.trick.return_value = "Surprise!"
>>> magic.trick()
'Surprise!'
>>> new_magic = copy.deepcopy(magic)
>>> new_magic
<MagicMock id='4476184976'>
>>> new_magic.hat
'rabbit'
>>> new_magic.trick()
'Surprise!'
```

But unfortunately there's some pitfalls when copying MagicMocks:

Copy in python is a reference.

You can't store mutable objects in your mocks because altering common test objects that at first glance appear to have no direct relationship to each other es no bueno:

```python
>>> class Pants(object):
...     name = 'Levis'
...
>>> class Animal(object):
...     genius = True
...
>>> peabody = MagicMock(Animal)
>>> peabody
<MagicMock spec='Animal' id='4476222864'>
>>> peabody.pants = MagicMock(Pants)
>>> peabody.pants
<MagicMock name='mock.pants' spec='Pants' id='4476221392'>
>>> peabody.pants.name
<MagicMock name='mock.pants.name' id='4476219920'>
>>> peabody.interests = ['history', 'travel']
>>> einstein = copy.copy(peabody)
>>> einstein
<__main__.Animal object at 0x10ace3d50>
>>> einstein.pants
<MagicMock name='mock.pants' spec='Pants' id='4476221392'>
>>> einstein.pants.name
<MagicMock name='mock.pants.name' id='4476219920'>
>>> einstein.interests
['history', 'travel']
>>> einstein.interests.append('science')
>>> peabody.interests
['history', 'travel', 'science']
```

deepcopy copies the original object if you've speced it off of something else:

```python
>>> sherman = copy.deepcopy(peabody)
>>> sherman
<__main__.Animal object at 0x10ace3c10>
>>> sherman.pants
<__main__.Pants object at 0x10ace3bd0>
>>> sherman.pants.name
'Levis'
```

so tl;dr: making derivative mocks off of a spec involves a lot of manual labor

### Mocking an instantiated object's methods is less than straightforward

This doesn't work like you think it does.

```python
>>> import subprocess
>>> from mock import Mock
>>> mock_subprocess = Mock(subprocess)
>>> mock_subprocess.Popen.communicate.return_value = ('stdout', 'stderr')
>>> process = mock_subprocess.Popen()
>>> process.communicate()
<Mock name='mock.Popen().communicate()' id='4503140368'>
```

Instead you create a mock of the instantiated object, and then manipulate its method calls.

``` python
>>> mock_popen_obj = Mock(subprocess.Popen)()
>>> mock_popen_obj.communicate.return_value = ('stdout', 'stderr')
>>> mock_subprocess.Popen.return_value = mock_popen_obj
>>> process = mock_subprocess.Popen()
>>> process.communicate()
('stdout', 'stderr')
```

### You can't mock an error and then raise it as a side effect.

```python
>>> problem_child = Mock()
>>> mock_key = Mock(KeyError)
>>> mock_key.shoutout = 'Holla.'
>>> problem_child.raises = Mock()
>>> problem_child.raises.side_effect = mock_key
>>> problem_child.raises()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Library/Python/2.7/site-packages/mock.py", line 955, in __call__
    return _mock_self._mock_call(*args, **kwargs)
  File "/Library/Python/2.7/site-packages/mock.py", line 1010, in _mock_call
    raise effect
TypeError: exceptions must be old-style classes or derived from BaseException, not Mock
```

...or can you?

```python
>>> mock_exception = Mock(BaseException)
>>> mock_exception.shoutout = "It's ya boy!"
>>> problem_child.raises.side_effect = mock_exception
>>> problem_child.raises()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Library/Python/2.7/site-packages/mock.py", line 955, in __call__
    return _mock_self._mock_call(*args, **kwargs)
  File "/Library/Python/2.7/site-packages/mock.py", line 1010, in _mock_call
    raise effect
TypeError: exceptions must be old-style classes or derived from BaseException, not Mock
```

Nope, you can't.  Sorry. Figure out another way to test that thing.


## Mocking Frameworks in Other Languages

Many languages have mock frameworks, and more are being developed!

* C++ has GMock
* Angular has ngMock
* Objective-C and Swift have OCMock
* Java and C# have EasyMock
* Java has mockito
* C# has Moq

YMMV though.  Read the docs.  Decide if mocking is the right thing for what you want to test.

## Resources

* [Mock docs](https://docs.python.org/dev/library/unittest.mock.html) and [getting started](https://docs.python.org/dev/library/unittest.mock-examples.html)
* [Mocks Aren't Stubs](http://martinfowler.com/articles/mocksArentStubs.html) by Martin Fowler
* [An Introduction to Mocking in Python](https://www.toptal.com/python/an-introduction-to-mocking-in-python)
* [Dr. Dobb's: Using Mocks in Python](http://www.drdobbs.com/testing/using-mocks-in-python/240168251)
* [GMock for Dummies](https://github.com/google/googletest/blob/master/googlemock/docs/ForDummies.md)
* [OCMock](http://ocmock.org/) and [OCMock Swift FAQ](http://ocmock.org/swift/)
