---
title: Untrack files already added to git repository based on .gitignore
date: 2018-09-10 18:36:27
tags: ["Git", "Github", ".gitignore"]
category: ["Git"]
---

가끔 `.gitignore`를 추가하지 않고 원격 저장소에 `push`를 하는 경우가 있었습니다. 이런 경우 저는 굉장히 무식한 방법을 통해 이를 수정하였는데요. (정도가 심해 따로 언급하지는 않겠습니다.)

보다 간편한 방법을 알게 되어 이렇게 글을 통해 기록하고자 합니다.

1. `.gitignore`를 포함해 모든 변경 사항을 커밋해줍니다. 
2. `git rm -r --cached .` 명령어를 통해 인덱스에 올라와 있는 파일만 제거해줍니다. 
3. `git add . ` 
4. `git commit -m "message"`
5. `git push`

---

출처

- [Untrack files already added to git repository based on gitignore](http://www.codeblocq.com/2016/01/Untrack-files-already-added-to-git-repository-based-on-gitignore/)