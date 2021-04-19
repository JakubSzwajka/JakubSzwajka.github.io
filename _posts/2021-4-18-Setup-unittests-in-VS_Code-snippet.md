---
layout: post
title: Set up unittests in VS_code snippet
published: false
excerpt_separator: <!--more-->
---


Stuff we will use: 

1. Pytest
2. Pytest-cov
4. Coverage Gutters — VS Code extension

Python extension will be needed too, but I assume you already have it if you read it. 


Pytest is using unittest module so don't bother if you have been using unittest only.

### Installation 

```
    pip install pytest 
    pip install pytest-cov
```

To install extension: type “Coverage Gutters” in VS Code extension panel. 

![Coverage_gutters_extension](https://github.com/JakubSzwajka/JakubSzwajka.github.io/blob/master/_posts/_images/setup_unit_tests_1.png?raw=true)


### We want them to work quick


### Add dummy test 

Move your code to 'src' directory and tests to 'test' directory.

```python 
class MagicModule():
    def __init__(self):
        pass 

    def makeMagic(self):
        return "dummy response"
```

```python 
import unittest
from src.main_module import MagicModule 

class FooTestCase(unittest.TestCase):
    def setUp(self):
        self.cut = MagicModule()

    def test_foo_test(self):
        
        resp = self.cut.makeMagic()
        self.assertEqual( resp, "dummy response")

```

### Time to run it 

The way to run those tests is typing ``pytest`` in console. You can use '-q' flag too for shorter response. You should see something like this. 

```
======================================================== test session starts ========================================================
platform win32 -- Python 3.8.2, pytest-6.2.3, py-1.10.0, pluggy-0.13.1
rootdirectory: C:\DEV\SAMPLE_PROJECT
plugins: cov-2.11.1
collected 1 item

test\test_main_funcionality.py .                                                                                               [100%] 

========================================================= 1 passed in 0.26s =========================================================
```

But I still want to it work faster. I think the best way to remember that we should run tests and write them is to use muscle memory! Just use shortcut. I set it to ``Alt+t``. If you have Python extension go to Keyboard Shortcuts settings (F1 + type keyboard shortcuts) and then type 'Run tests'. You should find a 'Python: Run All Tests'. Set it to whatever shortcut you want. 

Now press your shortcut. When run for the first time you should see something like this. 

![coverage_gutters_extension](https://github.com/JakubSzwajka/JakubSzwajka.github.io/blob/master/_posts/_images/setup_unit_tests_2.png?raw=true)

Follow the steps to configure it for workspace. I choosed pytest and test folder which contains my tests. 

Run it again and check if no errors occur. 

### ERROR 

Here is the place where I bump into Stack Overflow. I had error message telling me that ``src.main_module.MagicModule`` cannot be imported in ``test_main_funcionality.py`` file. WTF? It worked 3 minutes ago. 

Found solution [here](https://stackoverflow.com/questions/41748464/pytest-cannot-import-module-while-python-can). Just add '__init__.py' file to test directory. 

When added it should work. You should see all tests green. 

![coverage_gutters_extension](https://github.com/JakubSzwajka/JakubSzwajka.github.io/blob/master/_posts/_images/setup_unit_tests_3.png?raw=true)

### Coverage

Ok, we have our shortcut for running tests. Now let's add one to run them with coverage.

You can just simply use ``pytest --cov=src test --cov-report term``. It will generate a '.coverage' file too. The output should be like: 

```
======================================================== test session starts ========================================================
platform win32 -- Python 3.8.2, pytest-6.2.3, py-1.10.0, pluggy-0.13.1
rootdirectory: C:\DEV\SAMPLE_PROJECT
plugins: cov-2.11.1
collected 1 item

test\test_main_funcionality.py .                                                                                               [100%] 

----------- coverage: platform win32, python 3.8.2-final-0 -----------
Name                 Stmts   Miss  Cover
----------------------------------------
src\main_module.py       5      0   100%
----------------------------------------
TOTAL                    5      0   100%


========================================================= 1 passed in 0.41s =========================================================
```

But still, I want it faster! First thing to do is to visualize them. Here comes an extension we installed previously. The Coverage Gutters. It will take data from '.coverage' file and visualize it into your code. *Remember, it needs XML file with those data*. Use ``pytest --cov=src test --cov-report xml:coverage.xml`` to generate such. If you still want to view results in console, you can add ``--cov-report term`` too. 

Remember to click 'watch' button on the bottom bar. Now you can see your coverage. 

![coverage_gutters_extension](https://github.com/JakubSzwajka/JakubSzwajka.github.io/blob/master/_posts/_images/setup_unit_tests_4.png?raw=true)

### Make some custom shortcut for this command

So far I didn't found the official method to add such custom commands, but I found a walkaround. 

Go to Keyboard Shortcuts again. Open them in JSON file. Add your extra key binding. My is ``ctrl+alt+t``. It will send command to integrated terminal.

```json
{
        "key": "ctrl+alt+t",
        "command": "workbench.action.terminal.sendSequence",
        "args": {
            "text": "pytest --cov=src test --cov-report xml:coverage.xml --cov-report term"
        }
    }
```

Now you just need to hit enter. If I find the way to automate hitting it too, I will let you know 😉. 