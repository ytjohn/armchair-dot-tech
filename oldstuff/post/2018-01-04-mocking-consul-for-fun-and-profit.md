---

title: Mocking Consul for fun and profit.
author: ytjohn
date: 2018-01-04 12:45:06

layout: post

slug: mocking-consul-for-fun-and-profit

---

I've been creating a fun microservice tool that provides a single API frontend and merges data from multiple backends. Since the app itself relies entirely on external data, I was wondering how in the world I would write unit tests for it. It's written in python using the amazing [apistar](https://github.com/encode/apistar) framework. All of my external data so far is gathered using the [requests](http://docs.python-requests.org/en/master/) library. The answer for this, turns out to [requests-mock](http://requests-mock.readthedocs.io/). Requests-mock will allow you create mock responses to requests. 

The documentation is pretty straightforward, but I was having some trouble wrapping my head around how I would use it to test the code in my app. To start simple, I decided to mock [consul](https://www.consul.io/), which is one of my datasources.

## Get a value into consul

First off, let's go ahead and setup. Go to https://www.consul.io/ and download the consul binary for your OS.

1. Start consul in dev mode/foreground: `consul agent -dev`
2. Insert a key: `consul kv put foo bar`
3. Let's request that key with curl. Be verbose, because there are some headers you'll want later.

```
$ curl http://127.0.0.1:8500/v1/kv/foo -v
* Connected to localhost (127.0.0.1) port 8500 (#0)
&gt; GET /v1/kv/foo HTTP/1.1
&gt; Host: localhost:8500
&gt; User-Agent: curl/7.54.0
&gt; Accept: */*
&gt;
&lt; HTTP/1.1 200 OK
&lt; Content-Type: application/json
&lt; X-Consul-Index: 7
&lt; X-Consul-Knownleader: true
&lt; X-Consul-Lastcontact: 0
&lt; Date: Thu, 04 Jan 2018 11:04:20 GMT
&lt; Content-Length: 158
&lt;
[
    {
        &quot;LockIndex&quot;: 0,
        &quot;Key&quot;: &quot;foo&quot;,
        &quot;Flags&quot;: 0,
        &quot;Value&quot;: &quot;YmFy&quot;,
        &quot;CreateIndex&quot;: 7,
        &quot;ModifyIndex&quot;: 7
    }
]
```

You may be wondering why "Value" is `YmFy`. That's because consul uses base64 encoding. Running `echo YmFy | base64 -d` will give you the string `bar`. 

## Read consul with python requests

Awesome, now I can write a fancy python script to show off my foo. Create a file called `requestkey.py`. 

```
import base64
import json
import requests

URL = &quot;http://127.0.0.1:8500/v1/kv&quot;

def requestkey(key):
  url = &quot;{}/{}&quot;.format(URL, key)
  r = requests.get(url)
  data = r.json()
  v = base64.b64decode(data[0][&#039;Value&#039;])
  # FYI v will return a byte instead of a string (b&#039;foo&#039;), we&#039;ll decode that to a string
  return (v.decode())

if __name__ == &#039;__main__&#039;:
  v = requestkey(&#039;foo&#039;)
  print(&quot;Here is my foo: {}&quot;.format(v))
```

Run it:

```
$ python requestkey.py
FETCHED: foo RESULT: b&#039;bar
```

### Live unit test

Let's create a test.py file. This will ensure the value of foo is equal to bar. 

```
import unittest
from requestkey import requestkey

class TestStringMethods(unittest.TestCase):

    def test_foo(self):
        v = requestkey(&#039;foo&#039;)
        self.assertEqual(v, &#039;bar&#039;)

if __name__ == &#039;__main__&#039;:
    unittest.main()
```

And run it:

```
$ python test.py
.
----------------------------------------------------------------------
Ran 1 test in 0.011s

OK
```

This works great! But what if the consul server on the CICD server running my test.py has a different value for foo? Or no consul server at all? 

```
(py3) jh1:mocktests ytjohn$ consul kv delete foo
Success! Deleted key: foo
(py3) jh1:mocktests ytjohn$ python test.py
$ consul kv put foo bard
Success! Data written to: foo
(py3) jh1:mocktests ytjohn$ python test.py
F
======================================================================
FAIL: test_foo (__main__.TestStringMethods)
----------------------------------------------------------------------
Traceback (most recent call last):
  File &quot;test.py&quot;, line 8, in test_foo
    self.assertEqual(v, &#039;bar&#039;)
AssertionError: &#039;bard&#039; != &#039;bar&#039;
- bard
?    -
+ bar


----------------------------------------------------------------------
Ran 1 test in 0.012s

FAILED (failures=1)
```

This is a problem. My code is still perfectly fine, but because the live data has changed, my test fails. That is what we hope to solve.

## Let's mock consul.

As I alluded at the top of my post, I hope to solve this with [requests-mock](http://requests-mock.readthedocs.io/). There's some fancy things I see like registering URIs, but to start, I am just going use the Mocker example they have. It's a good thing I did a curl request earlier to see what the actual response will be. 

```
import json
import requests_mock
import unittest

from requestkey import requestkey

class TestStringMethods(unittest.TestCase):

    def setUp(self):
       self.baseurl = &#039;http://127.0.0.1:8500/v1/kv&#039;

    def test_foo(self):
        key = &#039;foo&#039;
        url = &#039;{}/{}&#039;.format(self.baseurl, key)
        response = [{
                    &quot;LockIndex&quot;: 0,
                    &quot;Key&quot;: &quot;foo&quot;,
                    &quot;Flags&quot;: 0,
                    &quot;Value&quot;: &quot;YmFy&quot;,
                    &quot;CreateIndex&quot;: 7,
                    &quot;ModifyIndex&quot;: 7}]

        with requests_mock.Mocker() as m:
          m.get(url, text=json.dumps(response))
          v = requestkey(&#039;foo&#039;)
          self.assertEqual(v, &#039;bar&#039;)

if __name__ == &#039;__main__&#039;:
    unittest.main()
```

Let's try our test:

```
(py3) jh1:mocktests ytjohn$ consul kv get foo
bard
(py3) jh1:mocktests ytjohn$ python test.py
.
----------------------------------------------------------------------
Ran 1 test in 0.010s

OK
```

This is great. I can develop locally, store working examples in my test code, and test against that. 

## Requests is fun, but what about python-consul?

The truth is, I don't talk to consul using requests and base 64 decoding. For some reason, I thought it would
be easier for you to follow along if I did straight requests. But in reality, most people are going to use [python-consul](https://python-consul.readthedocs.io). In fact, here is my `getkey.py` file doing just that.

```
import consul

def getkey(key):
  c = consul.Consul() # consul defaults to 127.0.0.1:8500
  index, data = c.kv.get(key, index=None)
  return data[&#039;Value&#039;]

if __name__ == &#039;__main__&#039;:
  v = getkey(&#039;foo&#039;)
  print(&quot;Here is my foo: {}&quot;.format(v))
```

Now I'm going to rewrite my test.py to test both of these. I moved my response into the setUp
class, renamed test_foo to test_request and added a test_get to test my getkey function.

```
import json
import requests_mock
import unittest

from requestkey import requestkey
from getkey import getkey

class TestStringMethods(unittest.TestCase):

    def setUp(self):
       self.baseurl = &#039;http://127.0.0.1:8500/v1/kv&#039;
       self.response_foo = [{
                    &quot;LockIndex&quot;: 0,
                    &quot;Key&quot;: &quot;foo&quot;,
                    &quot;Flags&quot;: 0,
                    &quot;Value&quot;: &quot;YmFy&quot;,
                    &quot;CreateIndex&quot;: 7,
                    &quot;ModifyIndex&quot;: 7}]

    def test_request(self):
        key = &#039;foo&#039;
        url = &#039;{}/{}&#039;.format(self.baseurl, key)

        with requests_mock.Mocker() as m:
          m.get(url, text=json.dumps(self.response_foo))
          v = requestkey(&#039;foo&#039;)
          self.assertEqual(v, &#039;bar&#039;)

    def test_get(self):
        key = &#039;foo&#039;
        url = &#039;{}/{}&#039;.format(self.baseurl, key)

        with requests_mock.Mocker() as m:
          m.get(url, text=json.dumps(self.response_foo))
          v = getkey(&#039;foo&#039;)
          self.assertEqual(v, &#039;bar&#039;)

if __name__ == &#039;__main__&#039;:
    unittest.main()
```

Let's see how this does:

```
$ python test.py
E.
======================================================================
ERROR: test_get (__main__.TestStringMethods)
----------------------------------------------------------------------
Traceback (most recent call last):
  File &quot;test.py&quot;, line 35, in test_get
    v = getkey(&#039;foo&#039;)
  File &quot;/Users/ytjohn/vsprojects/unsafe/mocktests/getkey.py&quot;, line 7, in getkey
    index, data = c.kv.get(key, index=None)
  File &quot;/Users/ytjohn/.venvs/py3/lib/python3.6/site-packages/consul/base.py&quot;, line 538, in get
    params=params)
  File &quot;/Users/ytjohn/.venvs/py3/lib/python3.6/site-packages/consul/std.py&quot;, line 22, in get
    self.session.get(uri, verify=self.verify, cert=self.cert)))
  File &quot;/Users/ytjohn/.venvs/py3/lib/python3.6/site-packages/consul/base.py&quot;, line 227, in cb
    return response.headers[&#039;X-Consul-Index&#039;], data
  File &quot;/Users/ytjohn/.venvs/py3/lib/python3.6/site-packages/requests/structures.py&quot;, line 54, in __getitem__
    return self._store[key.lower()][1]
KeyError: &#039;x-consul-index&#039;

----------------------------------------------------------------------
Ran 2 tests in 0.016s

FAILED (errors=1)
```

Oh what disaster! My fancy getkey code is failing tests!.  What is `x-consul-index` anyways? Well, it looks
to be `response.headers['X-Consul-Index']`, which is a header we saw in the curl request. Fortunately,
mock allows you to provide headers as well.

1. Add headers to setUp: `self.headers = {'X-Consul-Index': "7"}` (yes, value must be a string)
2. Add the header response to your mockup: `m.get(url, text=json.dumps(self.response_foo), headers=self.headers_foo)`

```
$ python test.py
..
----------------------------------------------------------------------
Ran 2 tests in 0.012s

OK
```

Outstanding.  And for completion, here is the final `test.py`:

```
import json
import requests_mock
import unittest

from requestkey import requestkey
from getkey import getkey

class TestStringMethods(unittest.TestCase):

    def setUp(self):
       self.baseurl = &#039;http://127.0.0.1:8500/v1/kv&#039;
       self.headers_foo = {&#039;X-Consul-Index&#039;: &quot;7&quot;}
       self.response_foo = [{
                    &quot;LockIndex&quot;: 0,
                    &quot;Key&quot;: &quot;foo&quot;,
                    &quot;Flags&quot;: 0,
                    &quot;Value&quot;: &quot;YmFy&quot;,
                    &quot;CreateIndex&quot;: 7,
                    &quot;ModifyIndex&quot;: 7}]

    def test_request(self):
        key = &#039;foo&#039;
        url = &#039;{}/{}&#039;.format(self.baseurl, key)

        with requests_mock.Mocker() as m:
          m.get(url, text=json.dumps(self.response_foo))
          v = requestkey(&#039;foo&#039;)
          self.assertEqual(v, &#039;bar&#039;)

    def test_get(self):
        key = &#039;foo&#039;
        url = &#039;{}/{}&#039;.format(self.baseurl, key)

        with requests_mock.Mocker() as m:
          m.get(url, text=json.dumps(self.response_foo), headers=self.headers_foo)
          v = getkey(&#039;foo&#039;)
          self.assertEqual(v, &#039;bar&#039;)

if __name__ == &#039;__main__&#039;:
    unittest.main()
```


## What is the point?

There isn't much pointing to having a block of code produce a static value and then check to see if it is that value. However, when we start taking actions based on values (live, maintenance, true, false, -1), we can definitely check to see if our code behaves an expected way based on a collection of sample data we store. I can also check for how I handle incomplete data. A big part of my microservice correlates devices with network interfaces, ip addresses, and vlans. Not every interface has an ip, not every ip has a vlan. Not every network has a default gateway. I have to determine which ip is "primary". So as I collect examples of devices with different configurations, I should be able to register urls and responses for each device. If my code is expecting a vlan to be a number, but instead I receive a "None" - will I handle that or will I throw an exception error?

Looking forward, I can envision having sample json data stored with functions to provide the desired response and headers needed.
