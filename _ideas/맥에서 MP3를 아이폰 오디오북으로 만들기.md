### 설치부터 사용까지 완벽 가이드

> **챕터까지 자동으로! 한 번만 설치하면 다음부터는 폴더 드래그 + 엔터로 끝.**  
> 터미널 초보도 따라할 수 있게, 단계별로 차근차근.

---

## 이게 뭔가요?

MP3 파일 여러 개가 든 폴더를 터미널에 드래그하면,  
자동으로 챕터가 포함된 `.m4b` 오디오북 파일을 만들어주는 스크립트입니다.

아이폰 **Books 앱**에서 챕터별로 이동 가능한 진짜 오디오북으로 인식됩니다.

**설치는 처음 한 번만, 이후엔 폴더 드래그 + 엔터로 끝!**

---

## 준비물

- Mac
- MP3 파일들 (한 폴더에 모아두기)
- 터미널 (기본 내장 앱, 별도 설치 불필요)

---

## PART 1. 설치 (처음 한 번만)

### STEP 1. 터미널 열기

Spotlight 검색 (⌘ + Space) → `터미널` 입력 → 엔터

---

### STEP 2. Homebrew 설치

Homebrew는 맥에서 개발 도구를 쉽게 설치할 수 있게 해주는 패키지 매니저입니다.

터미널에 아래 명령어를 붙여넣고 엔터:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

진행 중 주의사항:
- **비밀번호 입력** 요청 → 맥 로그인 비밀번호 입력 (입력해도 화면에 표시 안 됨, 정상)
- **`Press RETURN/ENTER to continue`** 메시지 → 엔터키
- macOS 버전 경고가 떠도 무시하고 기다리면 됩니다
- 설치에 **5~15분** 걸릴 수 있습니다
- 터미널에 `%` 프롬프트가 다시 나타나면 완료!

---

### STEP 3. FFmpeg 설치

FFmpeg는 오디오/영상 변환의 핵심 도구입니다.

```bash
brew install ffmpeg
```

`%` 프롬프트가 돌아오면 완료.

---

### STEP 4. 오디오북 변환 스크립트 설치

아래 명령어를 **전체 복사해서 한 번에** 터미널에 붙여넣고 엔터:

```bash
cat > ~/audiobook.sh << 'EOF'
#!/bin/zsh
export PATH="/usr/local/bin:/opt/homebrew/bin:$PATH"
cd "$1"
for mp3 in *.mp3; do
  clean="${mp3//\'/}"
  if [ "$mp3" != "$clean" ]; then mv "$mp3" "$clean"; fi
done
ls *.mp3 | sort > filelist.txt
while IFS= read -r file; do echo "file '$file'"; done < filelist.txt > concat.txt
ffmpeg -f concat -safe 0 -i concat.txt -c:a aac -b:a 128k temp_merged.m4a -y
echo ";FFMETADATA1" > chapters.txt
start=0
while IFS= read -r file; do
  duration=$(ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 "$file")
  duration_ms=$(echo "$duration * 1000 / 1" | bc)
  end=$((start + duration_ms))
  title="${file%.mp3}"
  printf "\n[CHAPTER]\nTIMEBASE=1/1000\nSTART=%s\nEND=%s\ntitle=%s\n" "$start" "$end" "$title" >> chapters.txt
  start=$end
done < filelist.txt
dirname=$(basename "$1")
ffmpeg -i temp_merged.m4a -i chapters.txt -map_metadata 1 -codec copy "${dirname}.m4b" -y
rm -f temp_merged.m4a filelist.txt concat.txt chapters.txt
echo "✅ ${dirname}.m4b 완성!"
EOF
chmod +x ~/audiobook.sh
```

`%` 프롬프트가 돌아오면 설치 완료!

---

## PART 2. MP3 파일 준비

변환할 MP3 파일들을 **한 폴더**에 모아둡니다.  
파일 이름 앞에 `01_`, `02_`, `03_` 같은 번호를 붙여두면 챕터 순서가 자동으로 맞춰집니다.

```
📁 DragonMaster-19
  ├ 01_Introduction.mp3
  ├ 02_Chapter1.mp3
  ├ 03_Chapter2.mp3
  └ ...
```

> ✅ 파일명에 `'`(작은따옴표, apostrophe)가 있어도 괜찮아요.  
> 스크립트가 자동으로 처리해줍니다.

---

## PART 3. 사용법 (매번 이렇게만!)

### 1단계. 터미널을 열고 아래를 입력 (스페이스 포함!)

```bash
~/audiobook.sh 
```

> ⚠️ 마지막에 **스페이스 한 칸** 꼭 넣기!

### 2단계. Finder에서 MP3 폴더를 터미널 창으로 드래그

폴더를 터미널 창 위로 끌어다 놓으면 경로가 자동으로 입력됩니다:

```bash
~/audiobook.sh /Users/나의이름/Desktop/DragonMaster-19
```

### 3단계. 엔터!

자동으로 진행되면서 완료되면 이렇게 나옵니다:

```
✅ DragonMaster-19.m4b 완성!
```

같은 폴더 안에 `.m4b` 파일이 생성됩니다. 🎉

---

## PART 4. 아이폰으로 전송

1. USB 케이블로 아이폰과 맥 연결
2. Finder 열기 → 사이드바에서 아이폰 클릭
3. 상단 탭에서 **Books** 선택
4. `.m4b` 파일을 Books 영역으로 **드래그 앤 드롭**
5. 아이폰 Books 앱에서 확인!

Books 앱을 열면 챕터가 분리된 오디오북이 보입니다:

```
📚 DragonMaster-19
  ├ 01_Introduction
  ├ 02_Chapter1
  ├ 03_Chapter2
  └ ...
```

---

## 스크립트가 자동으로 해주는 것

- 파일명에 `'`(apostrophe)가 있으면 **자동으로 제거** (오류 방지)
- MP3 파일을 번호 순서대로 **자동 정렬**
- 각 MP3를 **챕터로 분리**해서 Books 앱에서 챕터 이동 가능
- 폴더 이름이 곧 **오디오북 파일 이름**이 됨
- 완성 후 **임시 파일 자동 삭제**

---

## 자주 묻는 질문

**Q. 매번 터미널을 켜야 하나요?**  
A. 네, 터미널은 켜야 해요. 하지만 `~/audiobook.sh ` + 폴더 드래그 + 엔터, 이게 전부예요.

**Q. 파일명에 번호가 없으면 어떻게 되나요?**  
A. 알파벳 순서로 정렬돼요. 챕터 순서를 맞추려면 `01_`, `02_` 같은 번호를 앞에 붙여두는 걸 권장해요.

**Q. 챕터 이름을 바꾸고 싶어요.**  
A. MP3 파일명이 곧 챕터명이 됩니다. 파일명을 원하는 이름으로 미리 바꿔두면 돼요.

**Q. 오디오북 파일 이름을 바꾸고 싶어요.**  
A. 완성된 `.m4b` 파일을 Finder에서 그냥 이름 바꾸면 돼요.

**Q. MP3가 아닌 다른 포맷도 되나요?**  
A. 스크립트 안의 `*.mp3` 부분을 `*.m4a` 등으로 바꾸면 됩니다.

---

*도구: FFmpeg + macOS 터미널 + zsh / Homebrew 필요*