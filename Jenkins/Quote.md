## Quote

### Environment

Jenkins ver. 2.176.1

### Background

어느 순간 부터 Unity Dashboard 에서 Crashes and Exceptions 가 쌓이지 않았다.

Unity connection issue 인 것 같다고 빌드할 때 라이센스 정보를 넣어 달라고 하셔서, 빌드 스크립트에 아래 옵션들을 추가 해야 했다.

```
-serial XXXXX -username YYYYY -password ZZZZZ
```

Jenkins Credentials 과 Agent 의 Environment Variables 을 이용해서 적용했더니, 일부 빌드들만 실패하기 시작했다.

```
[Pipeline] withEnv
[Pipeline] {
[Pipeline] sh
/Users/XXXXX/jenkins_workspace/workspace/YYY/ZZZZZ@tmp/durable-7c722216/script.sh: line 1: syntax error near unexpected token `)'
```

secret 을 잘못 설정 했나 확인 하다보니, quote 없이 password 에 `)` 가 들어가 있는 것이 문제였다.

### Action

**Single Quote** 를 추가해서 해결 (Double Quote 를 쓰지 않았던 이유는 password 에 `!` 도 들어갈 수 있어서)

```bash
bash buildscript.sh ... -p \'${UNITY_PASSWORD}\' ...
```

### Reference

- [Single Quotes](http://www.gnu.org/software/bash/manual/html_node/Single-Quotes.html)
    ```
    Enclosing characters in single quotes (‘'’) preserves the literal value of each character within the quotes. A single quote may not occur between single quotes, even when preceded by a backslash.
    ```

- [Double Quotes](http://www.gnu.org/software/bash/manual/html_node/Double-Quotes.html)
    ```
    Enclosing characters in double quotes (‘"’) preserves the literal value of all characters within the quotes, with the exception of ‘$’, ‘`’, ‘\’, and, when history expansion is enabled, ‘!’. When the shell is in POSIX mode (see Bash POSIX Mode), the ‘!’ has no special meaning within double quotes, even when history expansion is enabled. The characters ‘$’ and ‘`’ retain their special meaning within double quotes (see Shell Expansions). The backslash retains its special meaning only when followed by one of the following characters: ‘$’, ‘`’, ‘"’, ‘\’, or newline. Within double quotes, backslashes that are followed by one of these characters are removed. Backslashes preceding characters without a special meaning are left unmodified. A double quote may be quoted within double quotes by preceding it with a backslash. If enabled, history expansion will be performed unless an ‘!’ appearing in double quotes is escaped using a backslash. The backslash preceding the ‘!’ is not removed.

    The special parameters ‘*’ and ‘@’ have special meaning when in double quotes (see Shell Parameter Expansion).
    ```
