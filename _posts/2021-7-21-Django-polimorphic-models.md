---
layout: post
title: Django polymorphic - playing with models inheritance in Django
published: true
comments: true
excerpt_separator: <!--more-->
---

Inspired by some Reddit question, I tried to find the most efficient way of solving model relations problem and making them useful for future extensions. While playing with models relations, I found that [django-polymorphic](https://django-polymorphic.readthedocs.io/en/latest/) models can be a very useful extension. Let's dive into the problem and solve it with django-polymorphic.

<!--more-->

### What we know:

- We have some products.
- Products will have some sort of specification.
- Products can be of different types, so specifications will be different.
- Specification of the product will store information about the product, components, and their specification.

### Example.

PC that has:

- RAM memory 32 GB DDR4
- CPU (Intel Core i5 6cores 4.5GH)
- Power supply (100 W)

Laptop:

- RAM memory 16 GB DDR4
- CPU (Intel Core i5 6cores 4.5GH)
- Camera FaceTime HD 720p

The first thing I see is that the specification model can be different in every product type. So maybe common for every laptop but different for PCs.

Another thing to notice is that some specifications will have common components. Look at the CPU above. I think that making every component an object is a good idea. One model for CPUs, one for RAM, etc.

Let's start simple.
Models for components:

```python
class Cpu(models.Model):
    name = models.CharField(max_length=30)
    numberOfCores = models.IntegerField(default=4)
    frequency = models.FloatField(default=4.5)

    def __str__(self):
        return f"{self.name} {self.numberOfCores} - cores  {self.frequency}GH"


class RamMemmory(models.Model):
    quantity = models.IntegerField(default=4)
    type = models.CharField(max_length=10)

    def __str__(self):
        return f"{self.quantity} GB {self.type}"


class PowerSupply(models.Model):
    name = models.CharField(max_length=30)
    power = models.IntegerField(default=100)

    def __str__(self):
        return f"{self.name} {self.power}W"

class Camera(models.Model):
    name = models.CharField(max_length=30)
    resolution = models.IntegerField(default=720)

    def __str__(self):
        return f"{self.name} {self.resolution}p"

```

Now let's connect them in the specification. Inheritance is a good idea here. One thing to remember is that I can't set SpecsAbstract as abstract, because the abstract model does not have Foreign Key. I left the abstract in the name to know that I will inherit from it later.

```python
class SpecsAbstract(models.Model):
    name = models.CharField(max_length=100)
    description = models.CharField(max_length=100)

    def __str__(self):
        return f"{self.name} - {self.description}"

class PcSpecification(SpecsAbstract):
    cpu = models.ForeignKey(Cpu, on_delete=models.SET_NULL, null=True)
    ram = models.ForeignKey(RamMemmory, on_delete=models.SET_NULL, null=True)
    powerSupply = models.ForeignKey(PowerSupply, on_delete=models.SET_NULL, null=True)


class LaptopSpecification(SpecsAbstract):
    cpu = models.ForeignKey(Cpu, on_delete=models.SET_NULL, null=True)
    ram = models.ForeignKey(RamMemmory, on_delete=models.SET_NULL, null=True)
    camera = models.ForeignKey(Camera, on_delete=models.SET_NULL, null=True)
```

Connect everything in Product model.

```python
class Product(models.Model):
    name = models.CharField(max_length=100)
    spec = models.ForeignKey(SpecsAbstract, on_delete=models.RESTRICT)

    def __str__(self):
        return f"{self.name}"
```

And that's it. Now you can add different components to different specifications and connect them to Products.

But here goes a single trick. If you try to get specification object instance from product instance, Django will return the base model (SpecsAbstract type).

```txt
>>> Product.objects.first( ).spec
<SpecsAbstract: sample specification of laptop - test>
```

The solution for this is to use [django-polymorphic](https://django-polymorphic.readthedocs.io/en/latest/). Instead of the base class, it will return the child model of the product specification.

```txt
>>> Product.objects.first( ).spec
<LaptopSpecification: sample specification of laptop - test>
```

To fix this just install django-polymorphic `pip install django-polymorphic`, add it in `settings.py` and change the SpecsAbstract class.

```python
from polymorphic.models import PolymorphicModel

class SpecsAbstract(PolymorphicModel):
    name = models.CharField(max_length=100)
    description = models.CharField(max_length=100)

    def __str__(self):
        return f"{self.name} - {self.description}"
```

### What else?

A few useful functionalities of django-polymorphic models.

- Quering

QuerySet will be Polymorphic now. Instead of returning related SpecsAbstract instances, child models instances will be returned.

```txt
>>> SpecsAbstract.objects.all( )
<PolymorphicQuerySet [
    <PcSpecification: sample specification of PC - test>,
    <LaptopSpecification: sample specification of laptop - test>
    ]>
```

- instance_of and not_instance_of

Get all child models instances of the given class, or all others without the given class.

```txt
>>> SpecsAbstract.objects.instance_of(PcSpecification)
<PolymorphicQuerySet [<PcSpecification: sample specification of PC - test>]>

>>> SpecsAbstract.objects.not_instance_of(PcSpecification)
<PolymorphicQuerySet [<LaptopSpecification: sample specification of laptop - test>]>
```

- non_polymorphic

In special cases, if you want your QuerySet to act like vanilla Django, you can use non_polymorphic method.

```txt
>>> SpecsAbstract.objects.all( ).non_polymorphic( )
<PolymorphicQuerySet [
    <SpecsAbstract: sample specification of PC - test>,
    <SpecsAbstract: sample specification of laptop - test>
    ]>
```

And much more. Check the [docs](https://django-polymorphic.readthedocs.io/en/stable/advanced.html) if you are interested.
