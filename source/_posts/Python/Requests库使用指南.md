---
title: Requests库使用指南
date: 2022-02-08 15:40:20
tags:
  - requests
categories:
  - Python
---

> 注：本文翻译自[Python’s Requests Library (Guide)](https://realpython.com/python-requests/)。

在Python中，`requests`库是处理HTTP请求的标准。它使得复杂的HTTP请求变得简单、易用，用户只需要专注于处理与Web服务之间的交互过程，并获取所需的数据。

通过本文，你可以学到许多`requests`库中的有用功能，并且可以根据实际的使用场景，定制和优化这些功能。你还会学到如何有效地使用`requests`库，避免由于网络的原因影响应用的响应速度。

在本指南中，你将学到以下内容：

- 最常见的HTTP方式
- 定制HTTP请求中的报头和数据，使用查询字符和消息内容
- 检查请求和响应中的数据
- 发送认证请求
- 配置请求，防止应用的响应速度变慢

<!-- more -->

## 开始使用requests

首先，需要安装`requests`库：

```shell
$ pip install requests
```

安装完成后，需要先导入`requests`库：

```python
import requests
```

## GET请求

HTTP有很多种请求方式，例如GET和POST，不同的请求方式，所执行的操作是不同的。其中，GET是最常用的HTTP请求。GET主要是用来获取Web数据。如果发起一个GET请求，就会调用`requests.get()`。

以下例子中，你可以看到通过调用`get()`发送GET请求给Github的[Root REST API](https://developer.github.com/v3/#root-endpoint)。

```python
>>> requests.get('https://api.github.com')
<Response [200]>
```
 
## 响应

通过发送GET请求以后，我们可以得到一个Response的对象。将该对象存储到一个变量中后，我们可以查看该对象的属性和方法：

```python
>>> response = requests.get('https://api.github.com')
```

在以上例子中，我们将响应对象存储到`response`变量中。

### 状态码（Status Codes）

从响应`Response`中首先可以获取状态码，通过状态码来判断请求是否成功。

例如，如果状态码是`200 OK`，那么说明请求是成功的，但是如果状态码是`404 NOT FOUND`，则说明未找到需要获取的资源。关于状态码的详细说明，请参考文档：[many other possible status codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)

通过访问响应`response`的属性`.status_code`，可以看到服务器返回的状态码：

```python
>>> response.status_code
200
```

`.status_code`返回的是`200`，说明请求时成功的，并且服务器返回了你需要获取的数据。

有些情况下，你可能想要通过状态码来作为判断条件：

```python
if response.status_code == 200:
    print('Success!')
elif response.status_code == 404:
    print('Not Found.')
```

以上例子中，如果服务器返回`200`的状态码，那么程序就会打印`Success!`。如果服务器返回`404`的状态码，程序就会打印`Not Found.`。

`requests`提供了一种更为简便的方法。如果直接使用`Response`实例作为判断的条件，假设状态码在`200`和`400`之间，那么表达式就会被评估为`True`，否则就会被评估为`False`。

因此，你可以将之前的例子改写如下：

```python
if response:
    print('Success!')
else:
    print('An error has occurred.')
```

> 技术细节：This [Truth Value Test](https://docs.python.org/3/library/stdtypes.html#truth-value-testing) is made possible because [`__bool__()` is an overloaded method](https://realpython.com/operator-function-overloading/#making-your-objects-truthy-or-falsey-using-bool) on `Response`.
>
> This means that the default behavior of `Response` has been redefined to take the status code into account when determining the truth value of the object.

记住这种方式并不是检验状态码是否等于`200`，而是检验状态码是否在`200`到`400`之间，例如`204 NO CONTENT`和`304 NOT MODIFIED`，也被认为是请求成功的。

例如，`204`是说明请求是成功的，但是返回的消息正文并没有任何内容。

因此，如果只是为了知道请求是否成功，可以使用这种简便的方法，否则，建议还是根据具体的状态码来判断。

有时候，可能你不希望用`if`语句来检查状态码，而是希望在请求失败的情况下，引发异常。那么你就可以使用`.raise_for_status()`：

```python
import requests
from requests.exceptions import HTTPError

for url in ['https://api.github.com', 'https://api.github.com/invalid']:
    try:
        response = requests.get(url)

        # If the response was successful, no Exception will be raised
        response.raise_for_status()
    except HTTPError as http_err:
        print(f'HTTP error occurred: {http_err}')  # Python 3.6
    except Exception as err:
        print(f'Other error occurred: {err}')  # Python 3.6
    else:
        print('Success!')
```

如果你调用`.raise_for_status()`，就会对某些失败的状态码引发`HTTPError`的异常错误。如果状态码显示请求是成功的，那么程序就不会引发异常。

通过以上内容，你了解了很多关于如何处理状态码的知识。下面，你将看到如何查看服务器返回的具体数据内容。

### 内容（Content）

GET请求得到的响应经常包含很多有价值的信息，如消息正文中的payload。通过使用`Response`的属性和方法，你可以用不同的格式来查看payload。

用bytes的格式查看响应的内容，你可以使用`.content`：

```python
>>> response = requests.get('https://api.github.com')
>>> response.content
b'{"current_user_url":"https://api.github.com/user","current_user_authorizations_html_url":"https://github.com/settings/connections/applications{/client_id}","authorizations_url":"https://api.github.com/authorizations","code_search_url":"https://api.github.com/search/code?q={query}{&page,per_page,sort,order}","commit_search_url":"https://api.github.com/search/commits?q={query}{&page,per_page,sort,order}","emails_url":"https://api.github.com/user/emails","emojis_url":"https://api.github.com/emojis","events_url":"https://api.github.com/events","feeds_url":"https://api.github.com/feeds","followers_url":"https://api.github.com/user/followers","following_url":"https://api.github.com/user/following{/target}","gists_url":"https://api.github.com/gists{/gist_id}","hub_url":"https://api.github.com/hub","issue_search_url":"https://api.github.com/search/issues?q={query}{&page,per_page,sort,order}","issues_url":"https://api.github.com/issues","keys_url":"https://api.github.com/user/keys","notifications_url":"https://api.github.com/notifications","organization_repositories_url":"https://api.github.com/orgs/{org}/repos{?type,page,per_page,sort}","organization_url":"https://api.github.com/orgs/{org}","public_gists_url":"https://api.github.com/gists/public","rate_limit_url":"https://api.github.com/rate_limit","repository_url":"https://api.github.com/repos/{owner}/{repo}","repository_search_url":"https://api.github.com/search/repositories?q={query}{&page,per_page,sort,order}","current_user_repositories_url":"https://api.github.com/user/repos{?type,page,per_page,sort}","starred_url":"https://api.github.com/user/starred{/owner}{/repo}","starred_gists_url":"https://api.github.com/gists/starred","team_url":"https://api.github.com/teams","user_url":"https://api.github.com/users/{user}","user_organizations_url":"https://api.github.com/user/orgs","user_repositories_url":"https://api.github.com/users/{user}/repos{?type,page,per_page,sort}","user_search_url":"https://api.github.com/search/users?q={query}{&page,per_page,sort,order}"}'
```

通过`.content`读取到的是字节格式的payload，如果你想要将其转换成UTF-8编码的字符串，你可以使用`.text`：

```python
>>> response.text
'{"current_user_url":"https://api.github.com/user","current_user_authorizations_html_url":"https://github.com/settings/connections/applications{/client_id}","authorizations_url":"https://api.github.com/authorizations","code_search_url":"https://api.github.com/search/code?q={query}{&page,per_page,sort,order}","commit_search_url":"https://api.github.com/search/commits?q={query}{&page,per_page,sort,order}","emails_url":"https://api.github.com/user/emails","emojis_url":"https://api.github.com/emojis","events_url":"https://api.github.com/events","feeds_url":"https://api.github.com/feeds","followers_url":"https://api.github.com/user/followers","following_url":"https://api.github.com/user/following{/target}","gists_url":"https://api.github.com/gists{/gist_id}","hub_url":"https://api.github.com/hub","issue_search_url":"https://api.github.com/search/issues?q={query}{&page,per_page,sort,order}","issues_url":"https://api.github.com/issues","keys_url":"https://api.github.com/user/keys","notifications_url":"https://api.github.com/notifications","organization_repositories_url":"https://api.github.com/orgs/{org}/repos{?type,page,per_page,sort}","organization_url":"https://api.github.com/orgs/{org}","public_gists_url":"https://api.github.com/gists/public","rate_limit_url":"https://api.github.com/rate_limit","repository_url":"https://api.github.com/repos/{owner}/{repo}","repository_search_url":"https://api.github.com/search/repositories?q={query}{&page,per_page,sort,order}","current_user_repositories_url":"https://api.github.com/user/repos{?type,page,per_page,sort}","starred_url":"https://api.github.com/user/starred{/owner}{/repo}","starred_gists_url":"https://api.github.com/gists/starred","team_url":"https://api.github.com/teams","user_url":"https://api.github.com/users/{user}","user_organizations_url":"https://api.github.com/user/orgs","user_repositories_url":"https://api.github.com/users/{user}/repos{?type,page,per_page,sort}","user_search_url":"https://api.github.com/search/users?q={query}{&page,per_page,sort,order}"}'
```

因为将`bytes`转换成`str`需要指定编码的方式，`requests`会根据响应中的`hearders`内容来猜测编码的方式，或者，你也可以通过设置`.encoding`指定编码的方式：

```python
>>> response.encoding = 'utf-8' # Optional: requests infers this internally
>>> response.text
'{"current_user_url":"https://api.github.com/user","current_user_authorizations_html_url":"https://github.com/settings/connections/applications{/client_id}","authorizations_url":"https://api.github.com/authorizations","code_search_url":"https://api.github.com/search/code?q={query}{&page,per_page,sort,order}","commit_search_url":"https://api.github.com/search/commits?q={query}{&page,per_page,sort,order}","emails_url":"https://api.github.com/user/emails","emojis_url":"https://api.github.com/emojis","events_url":"https://api.github.com/events","feeds_url":"https://api.github.com/feeds","followers_url":"https://api.github.com/user/followers","following_url":"https://api.github.com/user/following{/target}","gists_url":"https://api.github.com/gists{/gist_id}","hub_url":"https://api.github.com/hub","issue_search_url":"https://api.github.com/search/issues?q={query}{&page,per_page,sort,order}","issues_url":"https://api.github.com/issues","keys_url":"https://api.github.com/user/keys","notifications_url":"https://api.github.com/notifications","organization_repositories_url":"https://api.github.com/orgs/{org}/repos{?type,page,per_page,sort}","organization_url":"https://api.github.com/orgs/{org}","public_gists_url":"https://api.github.com/gists/public","rate_limit_url":"https://api.github.com/rate_limit","repository_url":"https://api.github.com/repos/{owner}/{repo}","repository_search_url":"https://api.github.com/search/repositories?q={query}{&page,per_page,sort,order}","current_user_repositories_url":"https://api.github.com/user/repos{?type,page,per_page,sort}","starred_url":"https://api.github.com/user/starred{/owner}{/repo}","starred_gists_url":"https://api.github.com/gists/starred","team_url":"https://api.github.com/teams","user_url":"https://api.github.com/users/{user}","user_organizations_url":"https://api.github.com/user/orgs","user_repositories_url":"https://api.github.com/users/{user}/repos{?type,page,per_page,sort}","user_search_url":"https://api.github.com/search/users?q={query}{&page,per_page,sort,order}"}'
```

如果你仔细看一下响应的内容，你会发现实际上是序列化的`JSON`内容。因此，你可以通过`.text`的方式先获取字符串，然后再通过[`json.loads()`](https://realpython.com/python-json/#deserializing-json)的方式获取字典。其实，还有一种更为简单的方法，可以使用`.json()`：

```python
>>> response.json()
{'current_user_url': 'https://api.github.com/user', 'current_user_authorizations_html_url': 'https://github.com/settings/connections/applications{/client_id}', 'authorizations_url': 'https://api.github.com/authorizations', 'code_search_url': 'https://api.github.com/search/code?q={query}{&page,per_page,sort,order}', 'commit_search_url': 'https://api.github.com/search/commits?q={query}{&page,per_page,sort,order}', 'emails_url': 'https://api.github.com/user/emails', 'emojis_url': 'https://api.github.com/emojis', 'events_url': 'https://api.github.com/events', 'feeds_url': 'https://api.github.com/feeds', 'followers_url': 'https://api.github.com/user/followers', 'following_url': 'https://api.github.com/user/following{/target}', 'gists_url': 'https://api.github.com/gists{/gist_id}', 'hub_url': 'https://api.github.com/hub', 'issue_search_url': 'https://api.github.com/search/issues?q={query}{&page,per_page,sort,order}', 'issues_url': 'https://api.github.com/issues', 'keys_url': 'https://api.github.com/user/keys', 'notifications_url': 'https://api.github.com/notifications', 'organization_repositories_url': 'https://api.github.com/orgs/{org}/repos{?type,page,per_page,sort}', 'organization_url': 'https://api.github.com/orgs/{org}', 'public_gists_url': 'https://api.github.com/gists/public', 'rate_limit_url': 'https://api.github.com/rate_limit', 'repository_url': 'https://api.github.com/repos/{owner}/{repo}', 'repository_search_url': 'https://api.github.com/search/repositories?q={query}{&page,per_page,sort,order}', 'current_user_repositories_url': 'https://api.github.com/user/repos{?type,page,per_page,sort}', 'starred_url': 'https://api.github.com/user/starred{/owner}{/repo}', 'starred_gists_url': 'https://api.github.com/gists/starred', 'team_url': 'https://api.github.com/teams', 'user_url': 'https://api.github.com/users/{user}', 'user_organizations_url': 'https://api.github.com/user/orgs', 'user_repositories_url': 'https://api.github.com/users/{user}/repos{?type,page,per_page,sort}', 'user_search_url': 'https://api.github.com/search/users?q={query}{&page,per_page,sort,order}'}
```

`.json()`返回的结果是一个字典，因此你可以通过键来访问字典中的值。

除了状态码和消息正文，你还可以查看响应中的Headers。

## 报头（Headers）

响应中的报头通常包含很多有用的信息，例如内容类型，缓存内容的时间限制。

可以通过`.headers`来查看报头的内容：

```python
>>> response.headers
{'Server': 'GitHub.com', 'Date': 'Mon, 10 Dec 2018 17:49:54 GMT', 'Content-Type': 'application/json; charset=utf-8', 'Transfer-Encoding': 'chunked', 'Status': '200 OK', 'X-RateLimit-Limit': '60', 'X-RateLimit-Remaining': '59', 'X-RateLimit-Reset': '1544467794', 'Cache-Control': 'public, max-age=60, s-maxage=60', 'Vary': 'Accept', 'ETag': 'W/"7dc470913f1fe9bb6c7355b50a0737bc"', 'X-GitHub-Media-Type': 'github.v3; format=json', 'Access-Control-Expose-Headers': 'ETag, Link, Location, Retry-After, X-GitHub-OTP, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset, X-OAuth-Scopes, X-Accepted-OAuth-Scopes, X-Poll-Interval, X-GitHub-Media-Type', 'Access-Control-Allow-Origin': '*', 'Strict-Transport-Security': 'max-age=31536000; includeSubdomains; preload', 'X-Frame-Options': 'deny', 'X-Content-Type-Options': 'nosniff', 'X-XSS-Protection': '1; mode=block', 'Referrer-Policy': 'origin-when-cross-origin, strict-origin-when-cross-origin', 'Content-Security-Policy': "default-src 'none'", 'Content-Encoding': 'gzip', 'X-GitHub-Request-Id': 'E439:4581:CF2351:1CA3E06:5C0EA741'}
```

`.headers`返回的是字典类型的**对象**，因此，你可以像访问字典一样来获取值：

```python
>>> response.headers['Content-Type']
'application/json; charset=utf-8'
```

headers对象有一个特殊的地方。由于HTTP标准定义headers不区分大小写，所以，我们访问headers的时候，不需要担心大小写：

```python
>>> response.headers['content-type']
'application/json; charset=utf-8'
```

以上例子中，无论使用`content-type`还是`Content-Type`，你都能得到相同的结果。

## 查询字符串参数（Query String Parameters）

通过在URL中传递查询字符串参数，可以定制GET请求。在方法`get()`中，可以传递参数`params`。例如，你可以使用GitHub的查询API来查找`requests`库。

```python
import requests

# Search GitHub's repositories for requests
response = requests.get(
    'https://api.github.com/search/repositories',
    params={'q': 'requests+language:python'},
)

# Inspect some attributes of the `requests` repository
json_response = response.json()
repository = json_response['items'][0]
print(f'Repository name: {repository["name"]}')  # Python 3.6+
print(f'Repository description: {repository["description"]}')  # Python 3.6+
```

在方法`.get()`中，通过将字典`{'q': 'requests+language:python'}`传递给参数`params`，你可以改变从查询API返回的结果。

你也可以用字典作为参数`params`的值，也可以使用以元组为成员的数组作为参数`params`的值：

```python
>>> requests.get(
...     'https://api.github.com/search/repositories',
...     params=[('q', 'requests+language:python')],
... )
<Response [200]>
```

甚至，你还可以传递bytes作为值传递给参数：

```python
>>> requests.get(
...     'https://api.github.com/search/repositories',
...     params=b'q=requests+language:python',
... )
<Response [200]>
```

查询字符串对于自定义GET请求非常有用。同时，你还可以通过添加或者修改报头来自定义GET请求。

## 请求报头（Request Headers）

在方法`.get()`中，你可以通过将字典作为值传递给参数`headers`的方式，来自定义报头。例如，在之前的查询例子中，你可以通过在`Accept`报头中指定`text-match`类型：

```python
import requests

response = requests.get(
    'https://api.github.com/search/repositories',
    params={'q': 'requests+language:python'},
    headers={'Accept': 'application/vnd.github.v3.text-match+json'},
)

# View the new `text-matches` array which provides information
# about your search term within the results
json_response = response.json()
repository = json_response['items'][0]
print(f'Text matches: {repository["text_matches"]}')
```

`Accept`报头可以告诉服务器，你的应用可以处理的消息正文类型。在本例中，你使用的报头值是`application/vnd.github.v3.text-match+json`，这是GitHub网站`Accept`报头的专有属性，正文内容是一种特殊的JSON格式。

在学习其它自定义请求的方式之前，我们先简单了解一下其它的HTTP方法。

## 其它的HTTP方法

除了GET以前，HTTP还支持`POST`, `PUT`, `DELETE`, `HEAD`, `PATCH`和`OPTIONS`。在`requests`库中给，这些方法的使用方法和`get()`相似：

```python
>>> requests.post('https://httpbin.org/post', data={'key':'value'})
>>> requests.put('https://httpbin.org/put', data={'key':'value'})
>>> requests.delete('https://httpbin.org/delete')
>>> requests.head('https://httpbin.org/get')
>>> requests.patch('https://httpbin.org/patch', data={'key':'value'})
>>> requests.options('https://httpbin.org/get')
```

以下例子中，每一个函数调用都会对httpbin服务发起相应的HTTP方法。每一种方法，你都可以使用和之前相同的方式，查看服务器响应的内容：

```python
>>> response = requests.head('https://httpbin.org/get')
>>> response.headers['Content-Type']
'application/json'

>>> response = requests.delete('https://httpbin.org/delete')
>>> json_response = response.json()
>>> json_response['args']
{}
```

报头、响应正文、状态码和其它信息都会包含在每一种HTTP方法的`Response`中。接下来，你将更为详细地了解`POST`，`PUT`和`PATCH`方法，以及它们之间的区别。

## 消息正文

根据HTTP的规格说明，`POST`，`PUT`和`PATCH`都是通过将数据传递给消息正文，而不是通过将查询字符串传递给参数的方式。通过使用`requests`，你可以将`payload`传递给`data`参数。

你可以将字典，元组为成员的列表或者文件对象传递给参数`data`。在请求的正文中，根据具体的类型需要，你可以采用不同的数据类型：

例如，如果你请求的内容类型为`application/x-www-form-urlencoded`，那么你就可以将字典作为表格数据，传递给参数`data`。

```python
>>> requests.post('https://httpbin.org/post', data={'key':'value'})
<Response [200]>
```

你也可以使用以元组为成员的列表：

```python
>>> requests.post('https://httpbin.org/post', data=[('key', 'value')])
<Response [200]>
```

但是，如果你需要发送JSON格式的数据，你可以使用参数`json`。当你将值传递给参数`json`后，`requests`库会序列化你的数据和正确的`Content-Type`报头。

[httpbin.org](https://httpbin.org/)是由`requests`库的开发者 [Kenneth Reitz](https://realpython.com/interview-kenneth-reitz/)创建的一个用于测试的网站。该网站可以用来测试`requests`发起的各种请求。例如，你可以用它来测试POST请求：

```python
>>> response = requests.post('https://httpbin.org/post', json={'key':'value'})
>>> json_response = response.json()
>>> json_response['data']
'{"key": "value"}'
>>> json_response['headers']['Content-Type']
'application/json'
```

## 检查请求

在发送请求到服务器之前，`requests`库会先将请求准备好。请求的准备包括验证报头以及序列化JSON内容。

你可以通过`.request`来访问准备请求（`PrepareRequest`）：

```python
>>> response = requests.post('https://httpbin.org/post', json={'key':'value'})
>>> response.request.headers['Content-Type']
'application/json'
>>> response.request.url
'https://httpbin.org/post'
>>> response.request.body
b'{"key": "value"}'
```

检查准备请求可以让你访问各种信息，包括payload, URL, headers, authentication等等。

## 认证

认证可以让服务器知道你是谁。通常情况下，你可以通过`Authorization`报头提供你的身份信息，或者也可以通过自定义的报头来提供身份信息。在`requests`中，你可以使用参数`auth`来传递你的身份信息。

下面以GitHub的[Authenticated User](https://developer.github.com/v3/users/#get-the-authenticated-user) API为例。为了访问该API，你需要通过一个元组来传递用户名和密码：

```python
>>> from getpass import getpass
>>> requests.get('https://api.github.com/user', auth=('username', getpass()))
<Response [200]>
```

如果身份信息正确，request请求就会返回成功过。如果request请求中没有包含身份信息，服务器就会返回`401 Unauthorized`：

```python
>>> requests.get('https://api.github.com/user')
<Response [401]>
```

当你通过以上这种方式传递身份信息的时候，实际上`requests`使用的是HTTP的 [Basic access authentication scheme](https://en.wikipedia.org/wiki/Basic_access_authentication)。

因此，你还可以使用`HTTPBasicAuth`来传递身份信息：

```python
>>> from requests.auth import HTTPBasicAuth
>>> from getpass import getpass
>>> requests.get(
...     'https://api.github.com/user',
...     auth=HTTPBasicAuth('username', getpass())
... )
<Response [200]>
```

除了Basic认证方式以外，`requests`还支持其它认证方式，例如`HTTPDigestAuth`和`HTTPProxyAuth`。

你甚至还可以提供自定义的认证方式。但是，你必须首先创建一个继承`AuthBase`的子类。然后，你需要定义`__call__()`函数：

```python
import requests
from requests.auth import AuthBase

class TokenAuth(AuthBase):
    """Implements a custom authentication scheme."""

    def __init__(self, token):
        self.token = token

    def __call__(self, r):
        """Attach an API token to a custom auth header."""
        r.headers['X-TokenAuth'] = f'{self.token}'  # Python 3.6+
        return r


requests.get('https://httpbin.org/get', auth=TokenAuth('12345abcde-token'))
```

以上例子中，自定义的类`TokenAuth`会收到一个Token，然后将收到的token放到`x-TokenAuth`报头中。

不健全的认证机制会导致安全隐患，所以除非有特殊需求，否则应该使用成熟的认证机制，比如Basic或者OAuth。

## SSL证书验证

任何时候，当发送和接收的信息比较机密，安全往往是最重要的。因此，当你访问安全站点的时候，都会通过SSL建立安全连接，这时候对服务器的SSL证书进行验证显得尤为重要。

默认情况下，`requests`已经为你执行了SSL证书的验证。但是，在某些情况下，你可能希望改变这种默认的行为。

如果你想要关闭SSL证书的验证，你可以将参数`verify`设置为`False`：

```python
>>> requests.get('https://api.github.com', verify=False)
InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
  InsecureRequestWarning)
<Response [200]>
```

当你正在访问一个不安全的站点时，`requests`会发出警告信息提醒你。

> 注意：`requests`使用[`certifi`](http://docs.python-requests.org/en/master/user/advanced/#ca-certificates)来提供SSL证书的颁发机构（ Certificate Authorities）。这可以让`requests`知道有哪些可以信任的证书颁发机构。因此，你需要定期更新`certifi`来保证安全性。

## 性能

当在生产环境中使用`requests`的时候，特别需要考虑性能的影响。`requests`库的一些特性，例如超时控制（timeout control），会话（sessions）和重试限制（retry limits）可以使得应用运行得更流畅。

### Timeouts

当你向服务发送请求以后，你得应用将会等待服务端的响应，得到响应以后，才会执行下一步操作。如果应用等待的时间太长，请求就会一直处于等待状态，导致用户体验变差，或者后台任务挂起。

默认情况下，`requests`将会无限制地等待响应，所以你应该指定一个超时的时间值。你可以通过参数`timeout`来设置超时的时间值。`timeout`的值可以是整型或者浮点型，时间的单位是秒。

```python
>>> requests.get('https://api.github.com', timeout=1)
<Response [200]>
>>> requests.get('https://api.github.com', timeout=3.05)
<Response [200]>
```

以上例子中，第一个请求的超时时间是1秒，第二个请求的超时时间是3.05秒。

你还可以传递一个[元组](http://docs.python-requests.org/en/master/user/advanced/#timeouts)给参数`timeout`，元组中的第一个值是TCP连接的超时时间（客户端和服务器之间建立连接的时长，通常建议设置为比3的倍数大一点），第二个值是数据传输的超时时间（建立连接后，等待响应的超时时间）：

```python
>>> requests.get('https://api.github.com', timeout=(2, 5))
<Response [200]>
```

如果请求建立连接的时间在2秒以内，建立连接后，接收数据的时间在5秒以内，那么就可以正常获取到响应。但是，如果请求超时，那么函数就会抛出一个`Timeout`的异常。

```python
import requests
from requests.exceptions import Timeout

try:
    response = requests.get('https://api.github.com', timeout=1)
except Timeout:
    print('The request timed out')
else:
    print('The request did not time out')
```

你的程序可以捕获这个`Timeout`的异常，并做相应的处理。

### The Session Object

到目前为止，你都是使用`requests`的`get()`和`post()`来发送请求。这些函数隐藏了一些细节，例如连接是如何建立以及管理的。

实际上，这些细节都是由类`Session`来负责处理。如果你需要微调你的请求，或者改进性能，你需要直接使用`Session`的实例。

会话主要是在一些列的请求中，保持持久的参数。例如，如果你想要在多个请求中使用相同的认证信息，你就可以使用会话：

```python
import requests
from getpass import getpass

# By using a context manager, you can ensure the resources used by
# the session will be released after use
with requests.Session() as session:
    session.auth = ('username', getpass())

    # Instead of requests.get(), you'll use session.get()
    response = session.get('https://api.github.com/user')

# You can inspect the response just like you did before
print(response.headers)
print(response.json())
```

每次你用session发起请求，一旦该会话在初始化时，设定了认证的身份信息，那么该身份信息就可以在多个requests中重复使用。

通过session改进性能，主要是通过保持长连接的方式。当你的应用通过Session来发起请求，它就会在连接池中长时间保持该连接。当应用想要再次连接到同一个服务器的时候，就可以重复使用连接池中的连接，而不需要再新建连接。

### Max Retries

当请求失败的时候，你可能希望应用重新发起同样的请求。但是，`requests`默认并不会帮你重新发送请求。为了实现这个功能，你需要使用[Transport Adapter](http://docs.python-requests.org/en/master/user/advanced/#transport-adapters)。

Transport Adapters可以让你为每一个需要访问的服务定义一组配置信息。例如，你想要所有访问`https://api.github.com`的请求失败后重复三次，你可以创建一个Transport Adapter，设置参数`max_retries`的值为3，并将其挂载到当前的会话上：

```python
import requests
from requests.adapters import HTTPAdapter
from requests.exceptions import ConnectionError

github_adapter = HTTPAdapter(max_retries=3)

session = requests.Session()

# Use `github_adapter` for all requests to endpoints that start with this URL
session.mount('https://api.github.com', github_adapter)

try:
    session.get('https://api.github.com')
except ConnectionError as ce:
    print(ce)
```

当你挂载github_adapter到会话的时候，会话会遵循该配置的设置。

## 总结

通过以上内容，你现在可以：

- 发送不同的HTTP请求，例如：GET, POST, PUT
- 通过修改报头，认证，查询字符串和消息这个问，自定义请求
- 检查发送给服务器的数据，以及从服务器接收的数据
- SSL证书验证
- 通过max_retries, timeout, Sessions和Transport Adapters，更有效的使用`requests`
