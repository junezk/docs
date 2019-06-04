# monodb + djangorestframework

为了在djangorestframework中使用mongodb, 需要mongoengine和django-rest-framework-mongoengine

```
pip install mongoengine
pip install django-rest-framework-mongoengine
```

## 序列化类这样写

```python

# -*- coding: utf-8 -*-
from rest_framework_mongoengine.serializers import DocumentSerializer
from models import *
 
 
class QuestionSerializer(DocumentSerializer):
    """
    使用ModelSerializer实现序列化
    """
    class Meta:
        model = Question
        fields = ('question', 'answer', 'reason')
        #depth = 2
 
    def create(self, validated_data):
        return Question.objects.create(**validated_data)
 
    def update(self, instance, validated_data):
        instance.question = validated_data.get('question', instance.question)
        instance.answer = validated_data.get('answer', instance.answer)
        instance.reason = validated_data.get('reason', instance.reason)
        instance.save()
        return instance
```

当然最笨的方法就是不使用mongoengine，利用pymongo自己封装model，并且不使用viewset实现视图。