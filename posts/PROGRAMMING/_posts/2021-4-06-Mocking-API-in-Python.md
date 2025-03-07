---
layout: post
title: Mocking API in Python
published: true
comments: true
excerpt_separator: <!--more-->
tags: Python Pytest
---

They said that cool guys test their code. So I have tried. You can find here some samples for quick mocking API in your project. I've used it in my [console client for todoist](https://jakubszwajka.github.io/Todoist-Console-Client/).

<!--more-->

### Quick background

The CUT (_class under test_ - ConsoleClient here) is responsible for handling all logic of my app. It has it's own object to handle communication with API (todoist here).

It works like this:

1. Tell the ConsoleClient that we want to work with our tasks from todoist.
2. ConsoleClient goes to factory class and says, "sup homie?, gimme todoist api handler".
3. All api handlers based on one interface class so ConsoleClient don't has to bother about it anymore.

Logic complicated or not, should has some tests, that's the only way to deliver like a boss... I hope so.

So let's point the **problem**. First let's assume we want to write test case for ConsoleClient logic only! Imagine some date checking, filtering, printing etc. Second, tests should be fast, we don't want to create client, connect to API etc. We just need to cheet out ConsoleClient and make some _dump_ API handler. Let's do this!

### Prepare our api client

As I mentioned, all API clients have common interface, use it! Ladies and gentlemen, the fanciest API client ever below!

```python
class MockedClient(Todo_interface):
    def __init__(self, configuration):
        super().__init__(configuration)

        self.mocked_items = [{
            'id' : i,
            'content' : f'sample_content_{i}',
            'checked' : 1,
            'due' : {
                'date' : datetime.today().strftime('%Y-%m-%d')
                }
            } for i in range(5)]

    def getClient(self):
        pass

    def getItems(self):
        return self.mocked_items
```

### Insert it into code

Here comes the magic you came for. Use decorator _@mock.patch.object( Class_we_are_interested_in, "and_its_method_name")_. It basically patches the method of class with mock object.

In simple steps based on example below:

1. take ProviderFactory class
2. mock the specified method
3. pass changed ProviderFactory class to _setUp_ method as a _mock_client_factory_

Now it is showtime for our _MockedClient_. Specify its instance as a returning value of mocked method and voila!  
After that, when in _**init**_ method of ConcoleClient, _ProviderFactory.getClientHandler('some client's name')_ will be triggered, it will return out fantastic mock and use it as API handler later.

```python
from unittest import mock, TestCase

class TestCase(TestCase):
    @mock.patch.object(ProviderFactory, 'getClientHandler')
    def setUp(self, mock_client_factory):
        self.mocked_client = MockedClient( {'some_config': 'value'})
        mock_client_factory.return_value = self.mocked_client

        self.cut = ConsoleClient('todoist')
```

### what you should remember

1. Check the [docs](https://docs.python.org/3/library/unittest.mock.html#patch-object) for details or if anything fails.

2. Order of decorators and passing them to method.

In python decorators are applied from bottom up. This should correspond to mocked objects you pass to method. Check out example below.

```python

@mock.patch.object( some_class_1 , 'some_method_1')
@mock.patch.object( some_class_2 , 'some_method_2')
def test_blah_blah( self, mock_some_class_2, mock_some_class_1 ):
    ✨✨✨✨✨✨✨
    ✨TEST   MAGIC✨
    ✨✨✨✨✨✨✨

```
