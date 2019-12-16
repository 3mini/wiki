## Custom Git hooks

### Environment

GitLab 12.5.2 (49482945d28)

### Background

JavaScript 를 쓰면서 흔히 만났던 에러들은 [TypeScript](https://www.typescriptlang.org/) 를 도입 하면서 일부 해결이 되었다.

최소한 compile 되지 않는 코드는 push 할 수 없도록 custom hooks 를 이용해서 제한하고 있다.

### Action

1. project’s repository directory 로 이동 

    (e.g., `/var/opt/gitlab/git-data/repositories/<group>/<project>.git`)

2. 여기에 `custom_hooks` 라는 directory 생성

3. 그 안에 extension 없이 `pre-receive` 파일 생성 후 executable + owner 설정

```bash
#!/bin/bash

if test -n "$GIT_PUSH_OPTION_COUNT"
then
  i=0
  while test "$i" -lt "$GIT_PUSH_OPTION_COUNT"
  do
    eval "value=\$GIT_PUSH_OPTION_$i"
    case "$value" in
    force)
      echo "Force push"
      exit 0
      ;;
    esac
    i=$((i + 1))
  done
fi

set -e

zero_commit="0000000000000000000000000000000000000000"
PATH=/host_opt/node/bin:$PATH
while read oldrev newrev refname; do
  # Pass delete operation. ($newrev is zero commit)
  if [[ $newrev == $zero_commit ]]; then
    continue
  fi
  # Handle only branch. Pass tags (refs/tags/*) and tracking branch (refs/remotes/*).
  if [[ ! $refname =~ ^refs/heads/ ]]; then
    continue
  fi
  WORKSPACE=$(mktemp -d -t "prerecv.XXXXXXXXXX")
  git archive $newrev | tar xf - -C "$WORKSPACE"
  pushd "$WORKSPACE" > /dev/null
  {
    # npm install
    set +e
    tsc --noEmit
    RET=$?
    set -e
  }
  popd > /dev/null
  rm -rf "$WORKSPACE"
  if [ $RET -ne 0 ]; then
    echo "================================================================= "
    echo "██████╗ ███████╗     ██╗███████╗ ██████╗████████╗███████╗██████╗  "
    echo "██╔══██╗██╔════╝     ██║██╔════╝██╔════╝╚══██╔══╝██╔════╝██╔══██╗ "
    echo "██████╔╝█████╗       ██║█████╗  ██║        ██║   █████╗  ██║  ██║ "
    echo "██╔══██╗██╔══╝  ██   ██║██╔══╝  ██║        ██║   ██╔══╝  ██║  ██║ "
    echo "██║  ██║███████╗╚█████╔╝███████╗╚██████╗   ██║   ███████╗██████╔╝ "
    echo "╚═╝  ╚═╝╚══════╝ ╚════╝ ╚══════╝ ╚═════╝   ╚═╝   ╚══════╝╚═════╝  "
    echo "================================================================= "
    echo ""
    echo "Compile $refname failed. See log above."
    exit $RET
  fi
done
exit 0
```

### Reference

- https://docs.gitlab.com/ee/administration/custom_hooks.html
