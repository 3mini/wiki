## Bad Request

### Background

Jenkins 에 [Artifact Manager](../Jenkins/Artifact_Manager.md) 로 AmazonS3 를 설정한 이후,

[Flutter](https://flutter.dev/) 로 빌드 apk 들을 쉽게 다운 받을 수 있게 만든 Application 에서 에러가 발생

### Error

response 의 `statusCode` 와 `reasonPhrase` 가 **400 (Bad Request)**

### Troubleshooting

- `HttpClientRequest` 와 `HttpClientResponse` 에 있는 정보로는 원인을 파악하기 어려웠음

- `curl` 을 이용해 Jenkins 로 request 하면, AmazonS3 로 redirect (`302 Found`)

- Location 의 query string 에는 `X-Amz-Security-Token, X-Amz-Algorithm, X-Amz-Date, X-Amz-SignedHeaders, X-Amz-ExpiresX-Amz-Credential, X-Amz-Signature` 들이 존재

- 다시 그 location 에 request 하거나, 브라우저 address bar 에 입력하면 response 가 잘 내려옴

왜 Application 에서는 실패하는지 알 수 없어서 헤맸다.

### curl -L

[-L](https://curl.haxx.se/docs/manpage.html#-L) 를 추가해서 Jenkins 에게 request 를 했더니, 

```
<Error>
	<Code>InvalidArgument</Code>
    <Message>Only one auth mechanism allowed; only the X-Amz-Algorithm query parameter, Signature query string parameter or the Authorization header should be specified</Message>
    <ArgumentName>Authorization</ArgumentName>
    <ArgumentValue>Basic XXXXX</ArgumentValue>
    <RequestId>YYYYY</RequestId>
    <HostId>ZZZZZ</HostId>
</Error>
```

request 의 `Authorization Header` 가 문제였다.

### Expectation

Authorization Header 에 있는 credentials 는 `initial host (Jenkins)` 에서만 쓰고,

 redirect 이후 different host 에는 쓰지 않을 것이라고 생각했었다. 🙃

### Solution

개인적으로 마음에 들지는 않지만..

HttpClientRequest 의 `followRedirects` 를 false 로 하고, 

HttpClientResponse 가 redirect 하라고 하면 `maxRedirects` 만 정해두고 loop 를 했다.

```dart
Future<File> httpDownload(...) async {
	...
	try {
		HttpClientRequest request = await client.getUrl(uri)
			..followRedirects = false
			..headers.persistentConnection = false;
		HttpClientResponse resp = await request.close();

		if (resp.isRedirect) {
			resp = await followRedirects(client, resp);
		}

		if (resp.statusCode != HttpStatus.ok) {
			throw new HttpException('status code: ${resp.statusCode}');
		}

		return _downloadToFile(...);
	} catch (e) {
		return new Future.error(e);
	} finally {
		client.close();
	}
}

Future<HttpClientResponse> followRedirects(
	HttpClient client, HttpClientResponse response) async {
	int maxRedirects = 5;
	HttpClientResponse nextResponse = response;

	for (int i = 0; i < maxRedirects && nextResponse.isRedirect; ++i) {
		String url = nextResponse.headers['location'].last;
		HttpClientRequest request = await client.getUrl(Uri.parse(url))
			..followRedirects = false
			..headers.persistentConnection = false;
		nextResponse = await request.close();
	}

	return nextResponse;
}
```
