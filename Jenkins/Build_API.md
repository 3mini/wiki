## Build API

### Environment

Jenkins ver. 2.204.1

### Background

Parameterized Build 인 Job 을 params 를 바꿔가며 N 번 반복해야 되는 상황이 생겼다.

[Remote access API](https://wiki.jenkins.io/display/JENKINS/Remote+access+API) 를 따라 **buildWithParameters** API 를 이용해서 request 를 보냈으나, 제대로 parameters 가 전달되지 않았다.

Stack Overflow 를 찾아 보다 buildWithParameters 를 쓰지 않고 **build** API 를 이용해서 해결.

(String Parameter 외에 Choice Parameter 도 있어서 그런걸까 🤔)

### Action

```bash
curl -X POST JENKINS_URL/job/JOB_NAME/build \
    --user USER:TOKEN \
    --data-urlencode json='{"parameter": [{"name":"PARAM1", "value":"'"$VALUE1"'"}, {"name":"PARAM2", "value":"'"$VALUE2"'"}, {"name":"PARAM3", "value":"'"$VALUE3"'"}]}'
```
