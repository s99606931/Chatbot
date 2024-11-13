# Github 기본 명령어 가이드

## 1. 저장소 시작하기

### 사용자 등록 하기
```bash
git config --global user.email "s99606931@gmail.com"
git config --global user.name "kunkin"
```

### 새 저장소 만들기
```bash
git init
```

### 원격 저장소 복제하기
```bash
git clone <저장소_URL>
```

## 2. 변경사항 관리하기

### 파일 상태 확인
```bash
git status
```

### 변경사항 스테이징하기
```bash
git add <파일명>     # 특정 파일 스테이징
git add .           # 모든 변경사항 스테이징
```

### 변경사항 커밋하기
```bash
git commit -m "커밋 메시지"
```

## 3. 원격 저장소와 동기화

### 원격 저장소 추가하기
```bash
git remote add origin <저장소_URL>
```

### 변경사항 업로드하기
```bash
git push origin <브랜치명>
git push origin main    # main 브랜치로 푸시
```

### 원격 저장소에서 변경사항 가져오기
```bash
git pull origin <브랜치명>
```

## 4. 브랜치 관리

### 브랜치 목록 보기
```bash
git branch            # 로컬 브랜치 목록
git branch -r         # 원격 브랜치 목록
git branch -a         # 모든 브랜치 목록
```

### 새 브랜치 만들기
```bash
git branch <브랜치명>
```

### 브랜치 전환하기
```bash
git checkout <브랜치명>
git switch <브랜치명>    # 최신 버전
```

### 브랜치 생성과 동시에 전환하기
```bash
git checkout -b <브랜치명>
git switch -c <브랜치명>  # 최신 버전
```

## 5. 유용한 명령어

### 변경사항 확인하기
```bash
git diff              # 스테이징되지 않은 변경사항 보기
git diff --staged     # 스테이징된 변경사항 보기
```

### 커밋 히스토리 보기
```bash
git log               # 상세한 커밋 히스토리
git log --oneline     # 한 줄로 간단히 보기
```

### 실수 되돌리기
```bash
git reset HEAD <파일명>     # 스테이징 취소
git checkout -- <파일명>    # 수정사항 되돌리기
git reset --hard HEAD^     # 마지막 커밋 되돌리기
```

## 6. 자주 발생하는 오류와 해결방법

### 충돌(Conflict) 해결하기
1. 충돌이 발생한 파일을 열어서 충돌 부분 확인
2. 충돌 마커(<<<<<<, =======, >>>>>>>)를 찾아 원하는 코드만 남기고 수정
3. 수정 후 add와 commit 진행

### 권한 오류
```bash
git remote set-url origin https://<토큰>@github.com/사용자명/저장소명.git
```

### .gitignore 적용이 안될 때
```bash
git rm -r --cached .
git add .
git commit -m "gitignore 적용"
```


# 1. Visual Studio Code (VSCode) 사용
## VSCode 설치: Visual Studio Code를 설치합니다.
# 2. Markdown Preview Mermaid Support 확장 설치:
## VSCode의 확장(Extensions) 탭에서 "Markdown Preview Mermaid Support"를 검색하여 설치합니다.
# 3. 미리보기 사용:
## .md 파일을 열고, Ctrl + Shift + V (또는 Cmd + Shift + V on Mac)를 눌러 미리보기를 활성화합니다.
Mermaid 그래프가 시각적으로 표시됩니다.