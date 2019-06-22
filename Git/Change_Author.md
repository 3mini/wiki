## Change Author info

### Background

> ["분명 repo 에 commit 을 push 했는데, 왜 GitHub contributions 에 반영 되지 않을까?"](https://help.github.com/en/articles/why-are-my-contributions-not-showing-up-on-my-profile#commits)

문제는 repository 에 author 설정을 따로 하지 않았더니,

email address 가 git config global 에 있는 회사 계정으로 되어 있었다.

### ~~인간의 욕심은 끝이 없고 같은 실수를 반복한다~~

처음 실수 했을 때는 바로 알아차려서 reset 하고 force push 로 처리 한 뒤 정리를 안했는데,

같은 일이 또 발생했다. 두 번째는 며칠 뒤에 발견 했는데, commit 들의 date 를 유지하고 싶어서

찾아보다가 `git filter-branch` 로 해결하고 정리 하고자 한다.

### Solution

```bash
$ git clone --bare https://github.com/user/repo.git
$ cd repo.git
```

```bash
$ git filter-branch --env-filter '

OLD_EMAIL="__OLD__"
CORRECT_EMAIL="__NEW__"

if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```

```bash
$ git push --force --tags origin 'refs/heads/*'
```

```bash
$ cd ..
$ rm -rf repo.git
```

### Reference

- https://help.github.com/articles/changing-author-info/
