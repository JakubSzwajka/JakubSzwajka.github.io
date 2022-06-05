---
layout: post
title: I don’t want to use TestCase class anymore!
published: true
comments: true
excerpt_separator: <!--more-->
---

I’m a big fan of classes and sometimes I do OOP, even if it is not necessary. But hey, everyone has their style!

## The problem

I don't know why, but I had this opinion that when testing views I should go with the class approach. The problem appears when I wanted to load fixtures in a single test. YOU CAN'T! And there is even a note in docs.

> ``unittest.TestCase`` methods cannot directly receive fixture arguments as implementing that is likely to inflict on the ability to run general unittest.TestCase test suites. The above ``usefixtures`` and ``autouse`` examples should help to mix in pytest fixtures into unittest suites. You can also gradually move away from subclassing from ``unittest.TestCase`` to *plain asserts* and then start to benefit from the full pytest feature set step by step.


AAnd a quick note. Do not hate me because of test naming. It is just to show you sth.

## Solution 1

Ok. So let's start with usefixtures. Basically, you can import fixtures by their name with string value and it works.

```python
# /test_views.py

class ExerciseViewTestCase(TestCase):
	@pytest.mark.django_db
	@pytest.mark.usefixtures('exercise')
	def test_endpoint_should_return_list_of_exercises(self):
	    response = self.client.get(reverse('classes:exercise-list'), **self.get_header())
	    assert response.status_code == status.HTTP_200_OK
			assert response.data != []
```

Also if you want to use this exercise feature across multiple tests in this class you can go with:

```python
# /test_views.py

@pytest.mark.usefixtures('exercise_instance')
class ExerciseViewTestCase(TestCase):
    @pytest.mark.django_db
    def test_endpoint_should_return_list_of_exercises(self):
        response = self.client.get(reverse('classes:exercise-list'), **self.get_header())
        assert response.status_code == status.HTTP_200_OK
				assert response.data != []
```

I had this exercise in response. But hey! I wanted to know if it is exactly this exercise I'm importing with fixtures, so I needed its instance in an argument!

The problem is as they said.. "…cannot directly receive fixture arguments…". 🙁

The walk-around which is for me a MUCH BETTER/CLEANER/SEXIER solution now is to get rid of TestCase class.

## Solution 2

So what do we need: 

- Client
- Logged user
- Exercise fixture

Superuser and its Client can be a fixture too.

```python
# /conftest.py

@pytest.fixture()
def superuser():
    return UserModel.objects.create_superuser(
        username='test_user',
        email=EMAIL,
        password=TEST_PASSWORD
    )

@pytest.fixture()
def superuser_client(superuser):
    client = Client()
    
    token = client.post('/rest-auth/login/', data={
        'username':EMAIL,
        'password':TEST_PASSWORD
    }).data.get('key')
    
    client.defaults.update({'HTTP_AUTHORIZATION': f'Token {token}'})
    return client
```

I had the token authentication so after logging I needed to update the defaults argument in the Client instance.

After this, I can import as an argument a superuser_client. It will be logged (with token set) and ready to make requests.

Let's go back to this sexy test.

```python
# /test_views.py 

@pytest.mark.django_db
def test_endpoint_should_return_list_of_exercises(client_with_superuser, exercise_instance):
    response = client_with_superuser.get(reverse('classes:exercise-list'))
    assert response.status_code == status.HTTP_200_OK
    assert response.data[0].get('id') == exercise_instance.id
```

Smooth...

Cheers!

