---
layout: post
title: title to fill
published: false
comments: true
excerpt_separator: <!--more-->
---

```python

from core.models import Student, StudentGroup, Teacher
from rest_framework import status
from unittest import mock
from .common import UserModel, urlCore, CommonTestCaseWrapper
from core.models.NotificationModel import Notification


# Create your tests here.


@mock.patch.object(Notification, "sendEmail")
class StudentApiTestCase(CommonTestCaseWrapper):
    @classmethod
    @mock.patch.object(Notification, "sendEmail")
    def setUpTestData(cls, emailMock):
        cls.admin = UserModel.objects.create_superuser(
            "myAdmin", "myadmin@test.com", "11"
        )

        teacher = Teacher.objects.create_teacher(
            email="sample_teacher@wsb.pl",
            password="1234",
            first_name="name",
            last_name="surname",
        )
        teacher.save()
        cls.teacherId = teacher.id

        group = StudentGroup(groupName="sample_group_1", teacher=teacher)
        group.save()
        cls.groupId_1 = group.id

        group = StudentGroup(groupName="sample_group_2", teacher=teacher)
        group.save()
        cls.groupId_2 = group.id

        group = StudentGroup(groupName="sample_group_3", teacher=teacher)
        group.save()
        cls.groupId_3 = group.id

        student = Student.objects.create_student(
            email="sample_student@wsb.pl", index_id="1111", password="1234"
        )
        student.save()
        cls.studentId = student.id

    def setUp(self):
        self.login(email="myadmin@test.com", password="11")

    def test_should_add_student_instance(self, emailMock):
        response = self.addSampleStudentInstance(
            email="another_student@wsb.pl", password="1111111", index_id="2222"
        )
        student = Student.objects.filter(index_id="2222").first()
        self.assertEqual(student.user.email, "another_student@wsb.pl")
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)



```
