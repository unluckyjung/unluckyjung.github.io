---
title: Github 파일,폴더 이름 대소문자 변경 감지하기
date: 2022-01-18-02:00
categories:
- Github

tags:
- Git
- Github


---

## 깃허브에서 파일, 폴더명에서 대소문자만 변경되었을때 감지하는법을 알아봅니다.
> git의 기본 설정은 대소문자를 구분하지 못합니다.

---

## Goal
- 파일이나 폴더(디렉토리)명의 대소문자 변경사항만 있을때 Github에 적용해봅니다.

---

## 파일 이름 변경시
> FileName -> filename

```console
// git 2.21.0 이전 버전
git mv --force 이전파일이름 새로운파일이름
git mv --force FileName filename

// 이후 버전
git mv FileName filename
```

## 폴더(디렉토리) 이름 변경시
> FolderName -> foldername

```console
-- tmp를 임시 폴더로 사용

git mv FolderName tmp
git mv tmp foldername
```

- 잠시 다른폴더에 옮겼다가 (커밋할 필요는 없습니다), 대소문자가 변경된 폴더로 옮겨주면 됩니다.

![image](https://user-images.githubusercontent.com/43930419/149812711-b5c14e75-7c31-4626-abe6-a1dbac424946.png)

[실제로 적용된 결과](https://github.com/unluckyjung/blog-codes/commit/10ecf620215c1c4a9694bf30cf8ba77c6737c7de)  


---

## Reference
- [git github](https://github.com/git/git/commit/baa37bff9a845471754d3f47957d58a6ccc30058)
- [zansol velog](https://velog.io/@zansol/Git-%EB%8C%80%EC%86%8C%EB%AC%B8%EC%9E%90-%EB%B3%80%EA%B2%BD-%ED%9B%84-github-repo%EB%B0%98%EC%98%81%ED%95%98%EA%B8%B0)