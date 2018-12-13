---
title: ML 인프런에서 배운거 정리
updated: 2018-12-13 19:00
---


### Tensorflow 기본적인 operations

강사님 코드 [여기](https://github.com/hunkim/DeepLearningZeroToAll){:target="_blank"} 참고해서 만들었습니다.


```sh
pip3 install --upgrade tensorflow
# or
pip3 install --upgrade tensorflow-gpu
```


```python
>>> import tensorflow as tf

>>> tf.__version__

>>> hello = tf.constant("Hello World!")

>>> sess = tf.Session()

>>> print(sess.run(hello))

>>> n1 = tf.constant(3.0, tf.float32)

>>> n2 = tf.constant(4.0)

>>> n3 = tf.add(n1, n2)

>>> sess.run([n1, n2])

>>> sess.run(n3)
```

#### Placeholder

```
>>> a = tf.placeholder(tf.float32)

>>> b = tf.placeholder(tf.float32)

>>> adder_node = a + b

>>> sess.run(adder_node, feed_dict={a: 3, b: 4.5}))
# 7.5


>>> sess.run(adder_node, feed_dict={a: [1,3], b: [2, 4]}))
# [ 3. 7.]
```

#### Type, Shape, Rank

![rank]( {{ site.baseurl }}/assets/img/ml_intro/rank.png)

![shape]( {{ site.baseurl }}/assets/img/ml_intro/shape.png)

![type]( {{ site.baseurl }}/assets/img/ml_intro/type.png)

![examples]( {{ site.baseurl }}/assets/img/ml_intro/examples.png)


<div class="divider"></div>


### Linear Regression

Hypothesis based on "H(x)=Wx+b"

Cost: (H(x)-y)^2 

![total_cost]( {{ site.baseurl }}/assets/img/ml_intro/linear_cost.png)

Goal: Minimize "cost(W,b)"

