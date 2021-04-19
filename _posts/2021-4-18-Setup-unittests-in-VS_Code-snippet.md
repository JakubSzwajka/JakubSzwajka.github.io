---
layout: post
title: Muscle-memmory-and-TDD-in-VS-Code
published: true
excerpt_separator: <!--more-->
---

In my recent work I found that if I don't set up everything for triggering unit tests quickly, there is no chance for any TDD. This post is snippet for future me, how to set up everything in VS Code. 

<!--more-->

**Stuff we will use:**

1. Pytest
2. Pytest-cov
4. [Coverage Gutters](https://marketplace.visualstudio.com/items?itemName=ryanluker.vscode-coverage-gutters) — VS Code extension

[Python extension](https://marketplace.visualstudio.com/items?itemName=ms-python.python) will be needed too, but I assume you already have it if you read it. Pytest is using unittest module so don't bother if you have been using unittest only.

### Installation 

```txt
pip install pytest 
pip install pytest-cov
``` 

To install extension: type “Coverage Gutters” in VS Code extension panel.  Install it. 

![Coverage_gutters_extension](https://github.com/JakubSzwajka/JakubSzwajka.github.io/blob/master/_posts/_images/setup_unit_tests_1.png?raw=true)


### We want them to work quick

Whole point is to that if you start coding, there will always be something more important to do or check than setup first unit test. Let's do it as first and foremost thing. After that, find shortcuts that you like. I will show you how to set up them. 

### Add dummy test 

Move your code to 'src' directory and tests to 'test' directory.

```python 
# src/main_module.py

class MagicModule():
    def __init__(self):
        pass 

    def makeMagic(self):
        return "dummy response"
```

```python 
# test/test_main_funcionality.py

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

The way to run those tests is typing ``pytest`` in console. You can use ``-q`` flag too for shorter response. You should see something like this. 

```txt
=========================== test session starts ===========================
platform win32 -- Python 3.8.2, pytest-6.2.3, py-1.10.0, pluggy-0.13.1
rootdirectory: C:\DEV\SAMPLE_PROJECT
plugins: cov-2.11.1
collected 1 item

test\test_main_funcionality.py .                                     [100%] 

============================ 1 passed in 0.26s ============================
```

But I still want to it work faster. I think the best way to remember that we should run tests and write them is to use muscle memory! Just use shortcut. I set it to ``Alt+t``. If you have Python extension go to Keyboard Shortcuts settings (F1 + type 'keyboard shortcuts') and then type 'Run tests'. You should find a 'Python: Run All Tests'. Set it to whatever shortcut you want. 

Now press your shortcut. When run for the first time you should see something like this.

![coverage_gutters_extension](https://github.com/JakubSzwajka/JakubSzwajka.github.io/blob/master/_posts/_images/setup_unit_tests_2.png?raw=true)

Follow the steps to configure it for workspace. I choosed pytest and test folder which contains my tests. 

Run it again and check if no errors occur. 

### ERROR 

Here is the place where I bump into problem. I had error message telling me that ``src.main_module.MagicModule`` cannot be imported in ``test_main_funcionality.py`` file. WTF? It worked 3 minutes ago. 

Found solution [here](https://stackoverflow.com/questions/41748464/pytest-cannot-import-module-while-python-can). Just add ``__init__.py`` file to test directory. 

When added it should work. You should see all tests green. 

![coverage_gutters_extension](https://github.com/JakubSzwajka/JakubSzwajka.github.io/blob/master/_posts/_images/setup_unit_tests_3.png?raw=true)

### Coverage

Ok, we have our shortcut for running tests. Now let's add one to run them with coverage.

You can just simply use ``pytest --cov=src test --cov-report term``. It will generate a ``.coverage`` file too. The output should be like: 

```txt
========================== test session starts ==========================
platform win32 -- Python 3.8.2, pytest-6.2.3, py-1.10.0, pluggy-0.13.1
rootdirectory: C:\DEV\SAMPLE_PROJECT
plugins: cov-2.11.1
collected 1 item

test\test_main_funcionality.py                                     [100%] 

----------- coverage: platform win32, python 3.8.2-final-0 -----------
Name                 Stmts   Miss  Cover
----------------------------------------
src\main_module.py       5      0   100%
----------------------------------------
TOTAL                    5      0   100%


=========================== 1 passed in 0.41s ===========================
```

But still, I want it faster! First thing to do is to visualize them. Here comes an extension we installed previously. The Coverage Gutters. It will take data from '.coverage' file and visualize it into your code. *Remember, it needs XML file with those data*. Use ``pytest --cov=src test --cov-report xml:coverage.xml`` to generate such. If you still want to view results in console, you can add ``--cov-report term`` too. 

Remember to click ``watch`` button on the bottom bar. Now you can see your coverage. 

![coverage_gutters_extension](https://github.com/JakubSzwajka/JakubSzwajka.github.io/blob/master/_posts/_images/setup_unit_tests_4.png?raw=true)

### Make some custom shortcut for this command

So far, I have not found the official method to add such custom commands, but I found a walk around. 

Go to Keyboard Shortcuts again. Open them in JSON file. Add your extra key binding. My is ``ctrl+alt+t``. It will send command to integrated terminal. Remember to add ``\n`` at the end. This will trigger the command. 

```json
{
        "key": "ctrl+alt+t",
        "command": "workbench.action.terminal.sendSequence",
        "args": {
            "text": "pytest --cov=src test --cov-report xml:coverage.xml --cov-report term\n"
        }
    }
```

Now there is no excuse for not doing TDD 👌. 