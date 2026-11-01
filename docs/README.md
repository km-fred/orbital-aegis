# site/ — Orbital Aegis 지원 사이트

App Store 제출에 필요한 **지원 URL**과 **개인정보처리방침 URL**을 채우기 위한 정적 사이트.
빌드 도구 없음. HTML + CSS 3장이 전부다.

| 파일 | App Store Connect 에 넣는 곳 |
|------|------------------------------|
| `support.html` | **Support URL** |
| `privacy.html` | **Privacy Policy URL** |
| `index.html` | 선택 (Marketing URL). 링크 허브 |
| `app-ads.txt` | AdMob 수익 정산용 — 도메인 **루트**에 있어야 한다 |
| `styles.css` | 세 페이지 공용 |

---

## 0. 먼저 할 일 — 연락 이메일 채우기

`privacy.html` 과 `support.html` 의 `CONTACT_EMAIL_PLACEHOLDER` 를 실제 주소로 바꾼다.
Apple 심사관이 이 주소로 연락할 수 있어야 한다.

```bash
cd site
sed -i '' 's/CONTACT_EMAIL_PLACEHOLDER/you@example.com/g' privacy.html support.html
grep -c "CONTACT_EMAIL_PLACEHOLDER" *.html   # 0 이 나와야 한다
```

---

## 1. 배포 — GitHub Pages (권장)

도메인을 사지 않고 `https://<계정>.github.io/orbital-aegis/` 로 쓰는 방법.

```bash
# 저장소 안에서
git init
git add .
git commit -m "Orbital Aegis site"
git branch -M main
git remote add origin https://github.com/<계정>/orbital-aegis.git
git push -u origin main
```

GitHub → **Settings → Pages** → Source: *Deploy from a branch* → Branch: `main`, Folder: **`/docs`**.

> ⚠️ Pages 는 `/docs` 또는 루트만 고를 수 있다. `site/` 폴더를 그대로 쓰려면 `site` 를 `docs` 로
> 이름을 바꾸거나, 아래 Cloudflare 쪽을 쓴다. 이름을 바꾸는 편이 가장 간단하다:
> `mv site docs`

그다음 URL:

| 용도 | URL |
|------|-----|
| Support URL | `https://<계정>.github.io/orbital-aegis/support.html` |
| Privacy URL | `https://<계정>.github.io/orbital-aegis/privacy.html` |

`.nojekyll` 파일이 이미 들어 있다 — 없으면 Jekyll 이 파일명 처리를 건드린다.

### app-ads.txt 함정

AdMob 은 **도메인 루트**의 `app-ads.txt` 만 읽는다(`/orbital-aegis/app-ads.txt` 는 무시).
프로젝트 페이지(`/orbital-aegis/`)로 배포하면 루트에 둘 수 없으므로, 다음 중 하나가 필요하다:

- `<계정>.github.io` 라는 **user 페이지 저장소**를 따로 만들어 그 루트에 `app-ads.txt` 만 올린다
- 또는 커스텀 도메인을 붙인다 (그때는 프로젝트 페이지여도 루트가 된다)

`app-ads.txt` 는 **심사와 무관**하고 수익 정산 정확도에만 영향을 준다. 급하지 않으면 나중에 처리한다.

## 2. 배포 — Cloudflare Pages / Netlify (도메인 루트가 필요할 때)

폴더를 통째로 드래그 업로드하면 끝. 빌드 명령 없음, 출력 디렉터리 `site`.

- Cloudflare Pages → *Upload assets* → `site/` 폴더 선택
- Netlify → *Deploy manually* → `site/` 폴더를 드래그

이 경우 `https://<프로젝트>.pages.dev/app-ads.txt` 가 루트에 놓여 AdMob 조건도 함께 만족한다.

---

## 3. 페이지 내용의 근거

문구는 지어낸 것이 아니라 코드에서 확인한 사실만 쓴다. 바꿀 때 원본을 다시 확인할 것.

| 주장 | 근거 |
|------|------|
| 계정·서버·분석 없음 | `Sources/` 전체에 분석/크래시 SDK 없음, 네트워크 호출은 AdMob 뿐 |
| 수집 3항목(기기 ID·광고 데이터·이용 기록), 추적 안 함 | `Resources/PrivacyInfo.xcprivacy` |
| 저장 항목(별점·지갑·사운드·광고 실행 시각) | `ScoreStore` · `StarWallet` · `SoundLibrary` · `AppOpenAdManager` 의 UserDefaults 키 |
| 설치 후 첫 실행은 광고 없음 | `AdConfig.launchesBeforeFirstAd = 1` |
| 타워 3 + 특화 6, 맵 70, 적 8, SFX 25 | `TowerType` · `TowerSpec` · `LevelID` · `EnemyType` · `Resources/Sounds` |
| 판매 60%, 파괴 회수 25%, 웨이브 클리어 시 에너지 50% 회복 | `GameConfig+Variety` · `GameConfig+TowerCombat` |
| iPad 레터박스 138pt(11") / 199pt(13") | `CLAUDE.md` §1 실측값, `GameViewController.fillScaleMode` |
| 가로 전용, Split View 불가 | `Info.plist` 의 `UIRequiresFullScreen`, 방향 설정 |

`privacy.html` 의 문구를 바꾸면 **App Store Connect 의 App Privacy 설문과 어긋나지 않는지** 반드시 대조한다.
둘이 다르면 심사에서 Guideline 5.1.1 로 반려된다.

---

## 4. 아직 없는 것

- **UMP 동의 흐름** — EU/UK 배포 시 필요. 지금 앱은 동의 화면 없이 광고를 띄우므로,
  EU/UK 까지 배포하면 GDPR 문제가 될 수 있다. `_workspace/05_appstore_metadata.md` §7 참조.
- 사이트는 영문 단일. 앱 UI 와 언어를 맞춘 것이다.
