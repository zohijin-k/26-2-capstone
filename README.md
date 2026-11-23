# 전북 청년정책 신청서류 검토 시스템

**PRODUCTION_NOT_READY.** 두배적금과 청년 취업지원패키지의 신청·서식·서류 확인·담당자 심사를 돕는 데모입니다. 자동 판독은 담당자의 판단을 지원하며 행정 승인이나 실운영 준비 완료를 뜻하지 않습니다.

이 저장소는 **전체 안내와 분리 전 코드·이력 보관**을 담당합니다. `archive/main-before-split` 등 기존 태그를 유지합니다. 최신 세 앱의 코드를 한 런타임으로 합친 저장소가 아니며, 개발·연결 실행은 각 저장소 README를 기준으로 합니다.

## 저장소와 최종 연결 구조

| 저장소 | 현재 역할 |
|---|---|
| [capstone](https://github.com/zohijin-k/26-2-capstone) | 전체 안내·분리 전 코드와 이력 보관 |
| [officer](https://github.com/zohijin-k/26-2-savings-officer) | 두배적금 공용 백엔드 + 담당자 프런트 |
| [applicant](https://github.com/zohijin-k/26-2-savings-applicant) | 두배적금 신청자 프런트 |
| [youth](https://github.com/zohijin-k/26-2-youth) | 취업지원패키지·범용 청년사업의 독립 백엔드 + 신청자·담당자 프런트 |

```mermaid
flowchart LR
    A[두배적금 신청자 프런트 5173] --> O[officer FastAPI 8000]
    B[두배적금 담당자 프런트 5174] --> O
    O --> D[두배적금 DB·원본 저장소]
    Y[youth 신청자·담당자 프런트 5183] --> U[youth FastAPI 8010]
    U --> E[청년사업 DB·원본 저장소]
```

두배적금 연결 실행은 officer 백엔드 하나를 사용합니다. applicant 보존 백엔드와 capstone 보존 백엔드를 같은 인증 서비스로 취급하지 않습니다. youth는 독립 백엔드·DB입니다. 위 포트는 README 개발 예시이며 실운영 구성 승인이 아닙니다.

## OCR·Document AI·정책 경계

최신 개발 저장소의 `demo/backend`가 업로드·보존·체크리스트·제출·심사를 담당합니다. `engine_v2`는 OCR·분류·문서별 필드/좌표/신뢰도 추출, `engine`과 백엔드 규칙은 판정에 사용됩니다. 일반 OCR 체인은 fixture → 조건부 JBAI → LocalOcr → PDF text → 상용 스텁입니다. Upstage는 완성된 상용 연동이 아닙니다. own은 로컬 기본 선택이며 모델은 별도 설치·고정 해시 검증 대상입니다. 기본 선택에서 모델이 없으면 easyocr를 선택하지만 기본 requirements에 easyocr가 설치되는 것은 아닙니다. 명시한 own 오류는 다른 엔진으로 대체하지 않습니다.

두배적금 신청자에게는 `UPLOADED`/`REUPLOAD_REQUIRED`와 허용된 재업로드 사유만 투영합니다. 내부 `PASS`/`FAIL`/`NEEDS_REVIEW`, 점수·OCR 신뢰도·담당자 이유는 별개입니다. 서버가 submit에서도 기존 차단 조건을 재검사합니다. officer는 신청 소유자 쿠키와 담당자 로그인 세션을 구분하며 원본과 담당자 작성 PDF 접근을 보호합니다.

**youth canonical `PAYMENT_RECEIPT`는 수동 검토**입니다. 원본 envelope·열람 가능 여부만 확인하고 OCR·Document AI·필드 추출·신뢰도·화질 평가를 우회해 `NEEDS_REVIEW / RECEIPT_MANUAL_REVIEW`로 보존합니다. 담당자가 원본·신청 금액을 보고 기존 메모·결정에 기록합니다. 합본 하나로 다른 칸을 자동 충족시키지 않고 과거 저장 OCR·판정을 다시 쓰지 않습니다. [영수증 안내](https://github.com/zohijin-k/26-2-youth/blob/main/docs/receipt-manual-review.md)를 보세요.

담당자 결정·메모는 `expected_revision`, 일괄 처리는 `expected_revisions`를 보내는 Review CAS(compare-and-swap)를 사용합니다. 오래된 revision은 충돌로 처리하며 전체 심사 내용을 덮어쓰지 않습니다. 제출 후 문서 변경 잠금과 트랜잭션 회귀도 유지합니다. 세션은 단일 프로세스 가정을 포함하므로 다중 worker 안전성을 보장하지 않습니다.

[youth 정책표](https://github.com/zohijin-k/26-2-youth/blob/main/docs/phase7-policy-decisions.md)의 **P1–P19는 미결이며 현재 youth 판정을 유지**합니다. 낮은 화질·불확실한 항목 부재·발급일·납부금액·주소이력·사업장·동의 mark·읍면동·thread·preload 정책을 일괄 확정하지 않습니다. P11–P13의 추출/검토 사유 결합도 유지합니다. 확정된 P20 영수증 수동 검토만 별도 범위입니다.

## 실행 안내

- [officer 설치·담당자·공용 API](https://github.com/zohijin-k/26-2-savings-officer#readme): Python venv·hash lock·`.env`를 준비해 8000에서 실행합니다.
- [applicant 프런트](https://github.com/zohijin-k/26-2-savings-applicant#readme): `demo/frontend`의 `npm ci`, `npm run dev -- --port 5173 --strictPort`; officer 프런트는 5174입니다.
- [youth 독립 실행](https://github.com/zohijin-k/26-2-youth#readme): API 8010, `VITE_API_TARGET=http://127.0.0.1:8010`의 프런트 5183입니다.

실제 API 저장소의 `.env`에 `OFFICER_PASSWORD`·`OFFICER_SESSION_SECRET`을 설정하고 uvicorn `--env-file .env`로 읽습니다. youth 예제에 빠진 두 키는 로컬 `.env`에 추가합니다. 키가 없으면 담당자 API가 503으로 차단됩니다. 프런트에 비밀값을 넣지 않습니다. 합성 로컬 실행은 `JBAI_API_KEY`를 비우고 `JBAI_ALLOW_REAL_DOCS=false`, `PROGRAM_LLM_ENABLED=false`, youth `LLM_PROVIDER=none`을 유지합니다. JBAI OCR은 키·허용 플래그를 켜면 이미지 외부 전송이 가능하며 예제의 기존 문구가 기관 승인을 대신하지 않습니다. 담당자 요약과 공고문 제안은 판정과 별개인 선택 기능입니다.

### capstone 보존 코드의 로컬 재현

최신 세 앱 연결과 별개로 보존 코드 자체를 연구할 때만 새 개발 사본에서 실행합니다.

검증 환경은 **CPython 3.14.6 / macOS 26.6.2 / arm64, Node 26.13.1 / npm 11.20.0**입니다. 최신 코드의 Python 3.12·Linux·Windows 전체 실행은 미검증입니다. 과거 WSL 결과를 최신 main의 보장으로 사용하지 않습니다. Node의 직접 TypeScript 시험은 22.18 이상이 필요하며, 여기서는 위 실측 버전을 기준으로 합니다.

[`package-lock.json`](demo/frontend/package-lock.json)의 실제 잠금 버전은 React **19.3.0**, Vite **8.3.0**, TypeScript **6.0.3**, pdfjs-dist **5.7.284**입니다. Python 잠금은 FastAPI **0.162.2**, uvicorn **0.54.0**, SQLModel **0.0.44**, ONNX Runtime **1.31.1**, pytest **9.2.0** 등 55개 패키지를 포함합니다. [hash lock](demo/backend/locks/macos-arm64-py314.txt)과 [재현 범위](demo/backend/locks/README.md)를 확인하세요. 이 macOS 전용 lock을 다른 OS에 적용하거나 기존 requirements의 하한을 지원 보장으로 해석하지 않습니다. 프런트 package `0.0.0`·API 선언 `0.1.0`은 제품 릴리스 번호가 아닙니다.
```bash
# 새 로컬 개발 사본의 루트; 위 macOS/arm64·CPython 3.14.6 환경
python3.14 -m venv .venv
source .venv/bin/activate
python -m pip install --require-hashes --only-binary=:all: -r demo/backend/locks/macos-arm64-py314.txt
python -m pip check
cp .env.example .env
cd demo/frontend
npm ci
cd ../..
```

기존 [`demo/backend/requirements.txt`](demo/backend/requirements.txt)·[`engine_v2/requirements.txt`](engine_v2/requirements.txt)는 일반 의존성 선언이며 위 재현용 lock과 목적이 다릅니다. 모델·합성 시험용 글꼴은 lock에 포함되지 않습니다.

```bash
# capstone 루트, 합성 데이터만 사용
JBAI_API_KEY= JBAI_ALLOW_REAL_DOCS=false PROGRAM_LLM_ENABLED=false python -m uvicorn demo.backend.main:app --env-file .env --reload --host 127.0.0.1 --port 8000
```

다른 터미널의 `demo/frontend`에서 `npm run dev -- --port 5173 --strictPort`를 실행합니다. <http://localhost:5173>과 <http://localhost:8000/docs>를 사용합니다. 이 보존 백엔드에 officer/youth 최신 인증·수동 영수증 정책이 모두 존재한다고 가정하지 않습니다.

모델은 `python -m engine_v2.tools.fetch_own_models`로 별도 다운로드하고 `python -c "from engine_v2.tools.fetch_own_models import verify_models; verify_models()"`로 검증합니다. [`fetch_own_models.py`](engine_v2/tools/fetch_own_models.py)의 고정 SHA-256은 다음과 같습니다.

| 모델 | SHA-256 |
|---|---|
| `det.onnx` | `4d97c44a20d30a81aad087d6a396b08f786c4635742afc391f6621f5c6ae78ae` |
| `rec.onnx` | `cd6e2ea50f6943ca7271eb8c56a877a5a90720b7047fe9c41a2e541a25773c9b` |
| `korean_dict.txt` | `a88071c68c01707489baa79ebe0405b7beb5cca229f4fc94cc3ef992328802d7` |

## 검증 기록과 명령

```bash
# 저장소 루트, 위 venv 활성화, 외부 공급자 비활성화
JBAI_API_KEY= JBAI_ALLOW_REAL_DOCS=false PROGRAM_LLM_ENABLED=false python -m pytest -q demo/backend/tests engine_v2/tests
```

전체 suite에는 별도 모델·합성 글꼴 등 자산이 필요합니다. 합성 글꼴 도구는 `python -m engine_v2.tools.fetch_synth_fonts`이며 출처·사용 조건을 확인한 별도 로컬 자산입니다. 자산 없는 사본의 skip/failure를 숨기거나 hash/시험 규칙을 완화하지 않습니다.
```bash
# demo/frontend에서; tests와 함께 배치된 .test.ts도 포함
npm run lint
npm run build
python3 - <<'NODE_TESTS'
from pathlib import Path
import subprocess
roots = ['tests', 'src/lib', 'src/forms', 'src/pages/officer']
files = sorted(str(f) for root in roots for f in Path(root).glob('*.test.ts'))
assert files, 'No Node tests found'
subprocess.run(['node', '--test', *files], check=True)
NODE_TESTS
```

다음은 **이전 기술 PR HEAD의 전체 시험**과 **해당 코드 main의 선택된 핵심 시험** 기록입니다. 이번 README 작업에서 전체 suite를 새로 실행했다고 주장하지 않습니다. 검증 HEAD와 코드 main의 실행 코드·기존 시험·프런트 tree 동일성(추가 lock 제외)을 확인해 연결했습니다.

| 저장소 | 기술 PR | 기존 전체 Python 통과 | 기존 프런트 통과 | 코드 main 핵심 통과 |
|---|---|---:|---:|---:|
| officer | [#43](https://github.com/zohijin-k/26-2-savings-officer/pull/43), [#44](https://github.com/zohijin-k/26-2-savings-officer/pull/44) | 5,124 | 70 | 171 |
| youth | [#39](https://github.com/zohijin-k/26-2-youth/pull/39) | 3,601 | 138 | 43 |
| applicant | [#17](https://github.com/zohijin-k/26-2-savings-applicant/pull/17) | 2,606 | 98 | 41 |
| capstone | [#37](https://github.com/zohijin-k/26-2-capstone/pull/37), [#38](https://github.com/zohijin-k/26-2-capstone/pull/38) | 2,494 | 1 | 21 |

기존 Python 전체 **13,825**개, 핵심 **276**개는 실패·오류·skip 0입니다. 자산이 없는 필터 사본에서도 이 수치가 그대로 나온다는 보장은 아닙니다. officer/youth에는 GitHub Actions의 event/pr-head/latest-main backend/frontend 검증이 있습니다. applicant/capstone에는 hosted CI가 없으므로 로컬 결과만 주장합니다. 기존 bundle·deprecation·Fast Refresh 경고는 유지합니다.

## release-manifest

아래는 README 정리 직전 확인한 **최신 코드 main SHA**와 검증 범위의 공개 요약입니다. README-only merge로 main SHA는 바뀌지만 실행 코드·정책은 바뀌지 않습니다. 현재 main SHA와 문서 병합 이력은 각 저장소의 GitHub main/PR에서 조회하며, 자기 자신의 미래 merge SHA를 이 파일에 하드코딩하지 않습니다. 전체 내부 `release-manifest.final.json`·소스 스냅샷·검증 증거는 비공개 보존 대상이며 이 요약은 소스 재배포 허가가 아닙니다.

```json
{
  "schema_version": 1,
  "production": "PRODUCTION_NOT_READY",
  "observation_date": "2026-11-23 KST",
  "scope": "code main before README-only merges; resolve current main via GitHub or git ls-remote",
  "source_redistribution_authorized": false,
  "repositories": {
    "officer": {
      "repository": "zohijin-k/26-2-savings-officer",
      "code_main_sha": "006f151b67801340bc50302ec2d66dd900024b0b",
      "prior_full_python_passed": 5124,
      "prior_frontend_passed": 70,
      "final_code_core_passed": 171
    },
    "applicant": {
      "repository": "zohijin-k/26-2-savings-applicant",
      "code_main_sha": "a3baa5efe948800568c645c7736f2839f2af3e07",
      "prior_full_python_passed": 2606,
      "prior_frontend_passed": 98,
      "final_code_core_passed": 41
    },
    "youth": {
      "repository": "zohijin-k/26-2-youth",
      "code_main_sha": "2fcfeffe9b3645e99ccb6ec4addc1c3f2e596dbf",
      "prior_full_python_passed": 3601,
      "prior_frontend_passed": 138,
      "final_code_core_passed": 43
    },
    "capstone": {
      "repository": "zohijin-k/26-2-capstone",
      "code_main_sha": "657c926be7ae7edf9c062728d5991e172a889772",
      "prior_full_python_passed": 2494,
      "prior_frontend_passed": 1,
      "final_code_core_passed": 21
    }
  }
}
```

최신 ref는 저장소마다 아래 명령으로 재확인합니다.

```bash
git rev-parse HEAD
git ls-remote origin refs/heads/main
```

미결 README 항목은 검증 사실과 정책 결정을 구분합니다. Python 지원 하한은 승인하지 않고 실측 환경만 적었습니다. OFFICER 키·연결 구조·외부 전송 조건·easyocr 미설치 가능성은 현재 코드 사실이며 정책 변경이 아닙니다. Pretendard GOV 등의 권리 검토와 하위 `TRELLIS_SETUP`의 과거 clone 안내 정리는 별도 미결입니다. 원본 Trellis를 수정하지 않았습니다.

## 개인정보·권리·검증 한계

시험에는 합성 정보와 임시 DB·저장소만 사용합니다. 실서류·주민번호·OCR 원문·환경변수·쿠키·DB·업로드·모델·비공개 증거와 소스 스냅샷을 Git/PR에 올리지 않습니다. 마스킹이 원본 파일을 익명화한다는 뜻은 아닙니다. 실제 외부 OCR/LLM 호출과 실서류 정확도·신뢰도 보정, 최종 SHA 전체 브라우저 흐름, 다중 worker 세션·잠금은 이번 문서 검증으로 승인하지 않습니다.

프로젝트 LICENSE와 기여자·기관의 재배포 권한은 **미확정**입니다. 기존 공개 저장소도 신규 소스 공개 권한의 근거가 아닙니다. Noto subset 출처·조건, Pretendard GOV 고지, 기관 브랜드, PDF/HWP 서식 재사용, PyMuPDF의 AGPL/상용 조건 및 의존성 고지는 별도 확인이 필요합니다. [제3자 고지](THIRD_PARTY_NOTICES.md)를 확인하되 전체 프로젝트의 재배포 허가로 해석하지 마세요. 비공개 소스 스냅샷은 공개하지 않습니다.

**PRODUCTION_NOT_READY.** 운영 DB migration·배포·인프라 변경은 이 안내의 범위가 아닙니다.

기존 [개발 안내](docs/DEVELOPMENT.md) · [사용 안내](docs/USER_GUIDE.md) · [엔진 안내](engine_v2/README.md)는 보존 코드 문서입니다. 최신 앱의 동작·검증 수치는 각 개발 저장소 README와 위 고정 SHA를 우선 대조하세요.
