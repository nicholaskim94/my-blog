---
title: "[git] --force-with-lease 옵션으로 보다 안전하게 Rebase하기"
date: 2024-06-25 01:00:00 +09:00
modified: 2024-06-25 01:00:00 +09:00
tags: [git, version-control]
description: Shell adalah sebuah command-line interpreter; program yang berperan sebagai penerjemah perintah yang diinputkan oleh User yang melalui terminal, sehingga perintah tersebut bisa dimengerti oleh si Kernel.
image: "/git-push-force-with-lease/git-push-force.jpg"
---

<img src="/git-push-force-with-lease/git-push-force.jpg" alt="a git push meme" style="margin-left: auto; margin-right: auto; display: block;">


# Push가 안돼요!

보통 개발을 하다가 git을 활용해 remote server에 무언가를 push하려고 할때 만나게 되는 오류가 있습니다.  바로 아래의 오류를 만나는데요. Remote 서버에 여러분의 로컬 머신에 존재하지 않는 코드(커밋)가 있기 때문에 발생하는 오류입니다. 정확히 말하면 오류가 아니고 기능이라고 봄이 옳습니다. 이런 메세지가 나타날 경우 대부분의 경우에는 `git pull`을 하면 문제가 해결됩니다.

```
% git push
To https://github.com/nicholaskim94/my-blog.git
 ! [rejected]        develop -> develop (non-fast-forward)
error: failed to push some refs to 'https://github.com/nicholaskim94/my-blog.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```


# Rebase란?

그런데, 만약 remote server에 있는 코드(커밋)를 `git pull`하여 가져오고 싶지 않고, 제거하거나, 바꿔치려면 어떻게 해야할까요? Git은 이런 기능을 rebase 라는 커맨드를 통해서 제공합니다. Rebase는 사용자가 보기에는 커밋의 삭제나 수정처럼 보일 수 있지만, 실제로는 rebase의 대상이 되는 커밋들은 모조리 새로 생성됩니다. 

리베이스 하는 방법은 아래의 문서를 참고하시면 좋습니다
- [Git 공식: 브랜치 - Rebase 하기](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-Rebase-%ED%95%98%EA%B8%B0)
- [Atlassian Git Rebase](https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase)



# Rebase는 언제해야할까요?

<img src="/git-push-force-with-lease/git-flow.webp" alt="git flow diagram" style="margin-left: auto; margin-right: auto; display: block;">

[우아한형제들: 우린 Git-flow를 사용하고 있어요](https://techblog.woowahan.com/2553/)

가장 흔하게 사용되는 경우는 규모가 있는 프로젝트에 여러명이 협업을 하는 경우일겁니다. 많은 회사들에서 사용하는 아래의 git-flow diagram을 보면 여러명이 동시에 작업하는 develop 브랜치 대신, feature브랜치들을 생성해 각각 기능 단위의 개선 작업을 진행함을 볼 수 있습니다. 얼마나 브랜치의 단위를 작게 가져갈 지는 팀의 정책에 따라 다를겁니다. 

만약 특정 브랜치를 혼자 작업하는 경우 어떨까요? 해당 브랜치에 WIP 커밋이나, 나중에 히스토리를 트래킹하는데 불필요한 커밋들(tmp, tmp1, tmp2... ) 이 들어갈 수도 있고 또 어떤 커밋에 들어가있는 코드는 아주 민감한 정보라(커밋 히스토리에 남기면 안되는 비밀번호 등) 해당 내용을 추가 커밋으로 제거하는 것이 아니라 아예 원본 커밋을 수정해야할 경우가 생길겁니다.

또, 위의 git-flow 정책을 사용할 경우 주기적으로 target 브랜치와 컨플릭 해결을 해야할 때가 올겁니다. 이떄 target 브랜치를 매번 `git merge`해서 머지 커밋을 만드는것보다, 나의 브랜치의 자체를 rebase해서 target 브랜치의 최신 데이터와 동기화 하는 작업이 **더 깔끔할 수도 있습니다**


# Rebase한 뒤 Push가 안돼요!

자, 그렇다면 위의 장점들을 누리기 위해서 리베이스를 해보겠습니다. 리베이스 한 뒤에는 해당 내용이 remote server에 반영을 해야할텐데요. 로컬에서 제가 특정 브랜치의 커밋들의 rebase를 한 뒤에 push를 시도한다면, remote server의 입장에선, 기존에 있던 커밋들과 아예 다른 커밋들을 받는것이기에 "Push가 안돼요"와 같은 오류가 발생합니다. 게다가, 여기서 `git pull`을 시도한다면 "Remote server에 있는 제거하거나 바꿔치려는 코드"가 제가 방금 예쁘게 rebase 한 코드와 병합이 되어 짬뽕이 되어버립니다. 제가 원했던 결과물이 아니죠.

이럴 경우에는 **의도적**으로 remote server의 데이터를 덮어쓰기 하는게 필요합니다. 이럴떄 사용하는게 `git push` 커맨드의 `--force` 옵션입니다

```
 % git push --force
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/nicholaskim94/my-blog.git
 + c9f3a98...7c70831 develop -> develop (forced update)
 ```

# 이거이거 위험..할수도?
`git push --force`의 가장 큰 문제점은 다른 사람의 커밋을 지워버릴 수 있다는 점입니다. 바로 위의 예시에서 브랜치 A에 리베이스 작업을 한 뒤에 `force push`를 했는데 다른 사람이 그 사이에 해당 브랜치에 `git push`를 했을 경우에는 어떻게 될까요? 우린 **의도적**으로 덮어쓰기를 했기 떄문에 해당 커밋은 우리가 알던 모르던 제거됩니다. 따라서 공용으로 사용하는 브랜치에 `force push`를 하려는 경우 매우 유의해서 해야합니다. 일단 팀의 동료들에게 해당 브랜치에 코드를 푸시하지 말라고 요청을 하고... `git pull`을 통해 최신 상태임을 확인하고... 원하는 상태로 `rebase`를 한뒤에... `git push --force`로 브랜치를 덮어쓰고... 동료들에게 다시 브랜치가 사용 가능하다는 것을 알려야합니다! 시스템이 아닌 사람을 통해 [Lock](https://en.wikipedia.org/wiki/Lock_(computer_science))을 구현하게 되는거죠. 


# 더 나은 방법은 없나요?

<img src="/git-push-force-with-lease/git-push-force-with-lease.jpg" alt="a git push meme" style="margin-left: auto; margin-right: auto; display: block;">

[Git 공식: git push](https://git-scm.com/docs/git-push)


**있습니다.**

`rebase`를 통하여 원하는 상태의 코드를 만들되, `git push --force`를 통해서 덮어쓰는것이 아니라 **`git push --force-with-lease` 옵션을 사용하는 방법**입니다. 공식 문서의 해당 옵션에 대한 설명을 살펴보면:

> This option allows you to say that you expect the history you are updating is what you rebased and want to replace. If the remote ref still points at the commit you specified, you can be sure that no other people did anything to the ref. It is like taking a “lease” on the ref without explicitly locking it, and the remote ref is updated only if the “lease” is still valid.

리베이스 하기 전의 해당 브랜치의 최신 시점에 "lease"를 걸어두고(한글로 직역하면 임대.. 정도가 될것 같네요). 해당 시점으로부터 변화가 없을 경우에만 "lease"가 유효하다고 판단해서 `force push`를 허용해줍니다. 다시 말해 제가 `git pull`을 한뒤에 팀원중 (~~어둠의~~) 누군가가 remote server에 다른 커밋을 `push`한 상황에서 제가 리베이스 후 `push`를 시도해도 성공하지 않는다는 뜻입니다.

팀원들이 모두 해당 옵션을 사용한다면 `rebase`를 해도 남의 커밋을 덮어쓰는 일은 없겠죠?


# References
- [https://git-scm.com/docs/git-push](https://git-scm.com/docs/git-push)
- [https://stackoverflow.com/questions/52823692/git-push-force-with-lease-vs-force](https://stackoverflow.com/questions/52823692/git-push-force-with-lease-vs-force)
- [https://stackoverflow.com/questions/65837109/when-should-i-use-git-push-force-if-includes](https://stackoverflow.com/questions/65837109/when-should-i-use-git-push-force-if-includes)
- [https://willi.am/blog/2014/08/12/the-dark-side-of-the-force-push/](https://willi.am/blog/2014/08/12/the-dark-side-of-the-force-push)
- [https://safjan.com/understanding-the-differences-between-git-push-force-and-git-push-force-with-lease/](https://safjan.com/understanding-the-differences-between-git-push-force-and-git-push-force-with-lease/)