# Planet X and SPHEREx: 해커톤 준비 및 구현 플랜

## 개요

NASA Space Apps 2026 챌린지 ["Planet X and SPHEREx"](https://www.spaceappschallenge.org/2026/challenges/planet-x-and-spherex/)(난이도 Advanced)에 2인 팀(둘 다 코딩 입문 수준)으로 참가한다.

요구사항은 RA/Dec를 입력하면 SPHEREx 이미지를 시간순 영상처럼 보여주는 공개 웹 도구이며, 테스트 대상은 Barnard's Star, Epsilon Indi B, WISE 0855, RW Aurigae, R Draconis다.

Stellarium은 코드 기반으로 쓰지 않는다. 계산으로 그린 하늘을 보여주는 C++/Qt 데스크톱 앱(GPL-2.0)이라 실제 관측 이미지를 다루는 웹 도구와 구조가 다르기 때문이다. 별도 앱으로만 활용한다(조작 방식 참고, 위치 검증, 발표 자료).

## 꼭 지켜야 할 규칙 ([참가 FAQ](https://www.spaceappschallenge.org/resources/participant-faqs/) 기준)

- **11월 14일 전에는 챌린지 작업을 시작할 수 없다.** FAQ에 "Teams are not allowed to begin working on the challenges prior to the hackathon"이라고 적혀 있다.
- **AI 도구 사용은 허용되지만, 제출물에 어떻게 사용했는지 설명해야 한다.** 사이트에서는 AI를 "Super Intelligence(SI)"라고 부른다. 해커톤 중 사용 내역을 메모해 둔다.
- **참가 등록과 이벤트 선택이 필요하다.** 지역 이벤트 또는 온라인 Universal Event 중 선택한다. Universal Event는 최소 2명이어야 글로벌 심사 대상이 된다.
- **프로젝트는 소속 이벤트의 종료 시각까지 팀 프로젝트 페이지로 제출한다.**
- 18세 미만 참가자는 보호자 등록과 동행 등 추가 조건이 있다. 해당되면 FAQ와 약관을 확인한다.

**이 문서의 경계:** 이 플랜은 방향 정리 수준이다. 어디까지가 "챌린지 작업"인지 FAQ에 명시되어 있지 않으므로, 궁금하면 Local Lead에게 물어본다. 11월 14일 전에는 이 문서를 더 구체화하거나 프로젝트 코드를 작성하지 않는다.

## 용어 설명

| 용어 | 뜻 |
|---|---|
| **RA/Dec** | 하늘의 위치를 나타내는 좌표. 지구의 경도와 위도에 해당한다. |
| **고유운동** | 별이 실제로 움직여서 하늘에서 위치가 조금씩 바뀌는 현상. |
| **IRSA** | NASA의 적외선 천문 데이터 보관소. SPHEREx 이미지를 여기서 받는다. |
| **FITS** | 천문학에서 쓰는 이미지 파일 형식. 사진과 함께 ZODI, FLAGS 같은 정보가 한 파일에 들어 있다. |
| **TAP** | 데이터 보관소에 "이 위치를 찍은 이미지 목록을 줘"라고 묻는 방식. |
| **컷아웃** | 큰 이미지에서 필요한 부분만 잘라 받은 조각. |
| **ZODI** | 태양계 먼지에 반사된 햇빛(황도광). 사진 전체에 낀 안개 같아서 빼면 천체가 더 잘 보인다. |
| **FLAGS** | 픽셀마다 붙은 "이 값은 믿기 어려움" 같은 표시. 이런 픽셀은 계산에서 뺀다. |
| **재투영** | 찍힌 방향이 조금씩 다른 이미지들을 같은 기준에 맞춰 정렬하는 작업. |
| **Streamlit** | Python 코드만으로 웹페이지를 만들어 주는 도구. |
| **CORS** | 웹페이지가 다른 사이트의 데이터를 가져오려면 그 사이트의 허락이 필요하다는 브라우저 보안 규칙. |
| **가상환경** | 이 프로젝트에 쓸 Python 라이브러리를 따로 모아 두는 공간. |

## 기술 선택

**Streamlit 앱 하나 + Python 천문 라이브러리**로 만든다.

- 서버와 화면을 Python 한 언어로 작성할 수 있어 입문자에게 가장 부담이 적다.
- 주소에 좌표를 넣는 기능은 `st.query_params`로 처리한다.
- 배포는 Streamlit Community Cloud를 1순위로 하고, 실제 배포 가능 여부는 해커톤 초반에 확인한다.
- 데이터는 `astroquery`(`Irsa.query_tap`), FITS 처리는 `astropy`, 이미지 정렬은 `reproject`, 영상 생성은 `imageio`(GIF)를 쓴다.
- Aladin Lite(Gaia/WISE 겹쳐 보기)는 시간이 남을 때만 HTML 임베드로 붙인다.

## 개발 환경

가상환경은 OneDrive 동기화를 피해 `%USERPROFILE%\.venvs\spaceapps`에 둔다. 11월 14일에 만들 저장소도 OneDrive 밖에 두는 것을 권장한다.

| 항목 | 버전 |
|---|---|
| Python | 3.14.0 |
| streamlit | 1.66.1 |
| astropy | 8.0.2 |
| astroquery | 0.4.11 |
| reproject | 0.21.0 |

팀원 PC에서는 [uv](https://docs.astral.sh/uv/)를 설치한 뒤 PowerShell에서 아래 명령을 차례로 실행한다.

```powershell
uv venv "$env:USERPROFILE\.venvs\spaceapps" --python 3.14
uv pip install --python "$env:USERPROFILE\.venvs\spaceapps\Scripts\python.exe" streamlit astropy astroquery reproject imageio pytest
```

Streamlit Community Cloud가 Python 3.14를 지원하는지는 배포할 때 확인한다.

## 데이터 흐름

1. **입력:** RA/Dec 또는 천체 이름, 시야 크기, 검출기(D1~D6)
2. **조회:** IRSA TAP으로 해당 위치의 이미지를 시간순 조회. [공식 튜토리얼](https://caltech-ipac.github.io/irsa-tutorials/spherex-cutouts)의 쿼리를 재사용한다.
   - 테이블: `spherex.artifact`, `spherex.plane`
   - 조건: `CONTAINS(POINT('ICRS', ra, dec), p.poly)`, `p.energy_bandpassname = 'SPHEREx-D2'`
   - 정렬: `ORDER BY p.time_bounds_lower`
3. **다운로드:** `{uri}?center={ra},{dec}&size={deg}` 컷아웃을 재시도와 함께 받고 디스크에 캐시한다. 튜토리얼에서도 병렬 다운로드 실패가 많았으므로 동시 요청 수를 낮게 시작한다.
4. **보정:** IMAGE에서 ZODI를 빼고, FLAGS로 나쁜 픽셀을 제외한다.
5. **정렬:** 이미지마다 방향이 다를 수 있으므로 공통 좌표 격자로 재투영한다.
6. **묶기:** 관측 시기와 검출기별로 합쳐 신호 대 잡음비를 높인다. 파장이 다른 이미지끼리 직접 비교하지 않기 위한 핵심 단계다.
7. **출력:** 밝기 조정 후 프레임을 GIF로 만들고, 시간 슬라이더로도 볼 수 있게 한다.

## 파일 구조 (11월 14일 이후 생성)

- `app.py`: Streamlit 화면과 URL 파라미터
- `spherex/query.py`: TAP 조회 (2단계)
- `spherex/cutouts.py`: 다운로드, 재시도, 캐시 (3단계)
- `spherex/stack.py`: 보정, 재투영, 묶기 (4~6단계)
- `spherex/render.py`: 밝기 조정, GIF 생성 (7단계)
- `tests/`: 시기 묶기, 나쁜 픽셀 제외 등 순수 함수 테스트
- `requirements.txt`, `README.md`

## 일정

### 11월 13일까지: 등록과 기초 학습만
- [ ] 참가 등록, 이벤트 선택, 팀 생성
- [ ] Git 사용자 이름과 이메일 설정, GitHub 계정 준비, `gh auth login`
- [ ] 팀원 PC에 같은 개발 환경 설치
- [ ] C 드라이브 여유 공간 확보 (2026-10-30 기준 약 2.6GB)
- [ ] Python, Git, FITS 기초 개념 익히기 (챌린지 결과물로 이어지는 코드는 만들지 않는다)
- [ ] Stellarium 앱 설치 후 사용법 익히기
- [ ] 11월 3일: Space Apps Connect 열림 (선택)
- [ ] 11월 13일: Resources 탭, Project Submission Guide, Judging & Awards Guide 확인 후 이 플랜 점검

### 1일차 (11월 14일)
- [ ] 저장소 생성, 빈 Streamlit 앱 배포까지 먼저 완료
- [ ] TAP 조회 → 컷아웃 → 단일 이미지 표시
- [ ] 시기별 묶기와 GIF 재생 (**여기까지가 최소 완성본**)
- [ ] 5개 타깃 데이터를 미리 받아 캐시

### 2일차 (11월 15일)
- [ ] URL 좌표 파라미터, 시야 크기 선택
- [ ] 여유가 있으면 순서대로: 시기 간 이미지 빼기 → 오염 플래그 표시 → 위치별 스펙트럼 → Aladin Lite 겹쳐 보기
- [ ] 발표 자료와 AI 도구 사용 설명 작성, 소속 이벤트 종료 시각 전에 제출

## 역할 분담

- **A:** 2~3단계 (데이터 조회와 다운로드), 배포
- **B:** 7단계와 `app.py` (화면과 영상), 발표 자료
- **함께:** 4~6단계 (제일 어려운 처리)와 타깃 검증

## 범위에서 제외

- Stellarium 코드 수정이나 코드 복사
- 자동 이동 천체 탐지, 계정 시스템, 사용자별 저장 기능
- 전체 하늘 처리 (요청한 영역만 처리)

## 검증

**전제:** 1일차 초반에 실제 데이터로 확인할 항목이 있다. 브라우저 없이 서버에서 받는 구조라 CORS는 문제가 되지 않을 것으로 보지만, 배포 환경에서 IRSA 접속과 메모리 한도는 직접 확인해야 한다.

| 타깃 | 확인할 내용 |
|---|---|
| Barnard's Star | 프레임 간 이동이 보이고, 방향과 크기가 SIMBAD 고유운동 및 Stellarium 예상 위치와 맞는지 |
| WISE 0855 | 주어진 좌표(08h55m10.831s, -7d14m42.53s) 근처에서 이동이 보이는지 |
| Epsilon Indi B | 주변에서 이동 천체가 보이는지 |
| RW Aurigae, R Draconis | 같은 검출기, 같은 파장대 조건에서 밝기가 변하는지 |

- 프레임 수가 조회 결과와 맞는지, 실패한 다운로드 수가 화면에 표시되는지 확인한다.
- URL에 좌표를 넣어 열었을 때 같은 영역이 바로 표시되는지 확인한다.
- 네트워크를 끊어도 캐시된 5개 타깃은 재생되는지 확인한다(발표 대비).
- `pytest`로 `tests/`를 실행한다.

## 미확인 사항

- 고유운동(Barnard's Star 약 10.4″/년)과 픽셀 크기(약 6.2″)는 기억에 의존한 값이다. 해커톤 때 SIMBAD와 [Explanatory Supplement](https://irsa.ipac.caltech.edu/data/SPHEREx/docs/SPHEREx_Expsupp_QR.pdf)로 확인한다.
- 움직임이 2~3픽셀 수준이면 GIF만으로는 눈에 잘 띄지 않을 수 있다. 그럴 경우 이미지 빼기를 1일차 범위로 앞당긴다.
- QR2와 QR3를 섞어 쓸 때의 플럭스 차이는 확인이 필요하다. 처음에는 한 릴리스만 사용한다.
