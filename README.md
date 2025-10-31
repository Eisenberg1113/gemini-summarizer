## 📝 프로젝트 소개

이 프로젝트는 Google의 최신 대규모 언어 모델인 **Gemini Flash (gemini-flash-latest)**를 활용하여, 사용자가 입력한 긴 텍스트를 즉시 분석하고 세 가지 형식으로 변환하는 간단한 웹 데모 애플리케이션입니다.

**주요 기능:**
1.  **핵심 3줄 요약:** 긴 글의 핵심 내용을 간결하게 3줄로 정리
2.  **주요 키워드 추출:** 내용의 중심이 되는 핵심 키워드 5개 추출
3.  **쉬운 말 버전 변환:** 내용을 초등학생도 이해할 수 있도록 쉽게 풀어서 설명

## 🛠️ 사용된 주요 기술 및 학습 과정 

### 1. Google Gemini API 및 모델 선택
* **모델:** `gemini-flash-latest` 사용
    * **선택 이유:** 이 프로젝트는 실시간으로 텍스트를 처리하고 빠른 응답 속도를 요구합니다. `Gemini Flash`는 대량의 텍스트를 높은 속도와 효율성으로 처리하는 데 최적화되어 있어, 데모 환경에 가장 적합하다고 판단했습니다.
* **SDK:** `google-generativeai` Python 라이브러리 사용
* **API 연동 경험:**
    * 초기에는 지원이 종료된 모델 이름(`gemini-1.5-flash`)으로 인해 `404 Not Found` 오류가 발생했습니다.
    * `list_models()` 기능을 통해 현재 API 키로 접근 가능한 모델 목록을 확인하고, **`gemini-flash-latest`**라는 올바른 모델 별칭(Alias)을 사용하여 문제를 해결했습니다. (이 경험을 통해 LLM API는 모델의 수명 주기와 버전 관리가 중요하다는 것을 학습했습니다.)

### 2. 프롬프트 엔지니어링 (Prompt Engineering)
* **역할 부여:** 모델에게 '텍스트를 분석하고 요약하는 전문 에디터'라는 역할을 부여하여 출력 품질을 높였습니다.
* **구조화된 출력 요청:** 결과를 단순히 텍스트로 받는 대신, 제목(##)과 목록(-)을 포함하는 **마크다운(Markdown) 형식**을 명시적으로 요청하여 일관되고 시각적으로 깔끔한 결과물을 얻도록 설계했습니다.
* **멀티-태스킹 프롬프트:** 하나의 API 호출 내에서 요약, 키워드 추출, 스타일 변환이라는 세 가지 다른 작업을 동시에 처리하도록 지시했습니다.

### 3. 개발 환경 및 UI 구성
* **환경:** Jupyter Notebook (Google Colab 환경에서도 완벽 호환)
* **UI:** `ipywidgets` 라이브러리를 사용하여 텍스트 입력창, 버튼, 결과 출력 영역 등의 간단한 웹 UI를 구축했습니다. 이를 통해 터미널이나 일반 Python 스크립트보다 시각적이고 직관적인 데모 환경을 제공했습니다.

## ⚙️ 실행 방법 (Jupyter Notebook / Colab)

### 1단계: 환경 준비

1.  GitHub에서 이 저장소의 코드를 다운로드하거나, Google Colab에서 새 Notebook을 엽니다.
2.  다음 명령어를 실행하여 필요한 라이브러리를 설치합니다.

    ```bash
    !pip install google-generativeai ipywidgets
    ```

### 2단계: 코드 실행

아래 코드를 복사하여 Notebook 셀에 붙여넣고 실행합니다.

**주의:** 코드 내 `GOOGLE_API_KEY = 'YOUR_API_KEY_HERE'` 부분을 **반드시 본인의 Gemini API 키**로 교체해야 합니다.

```python
import google.generativeai as genai
import ipywidgets as widgets
from IPython.display import display, Markdown
import os

# --- 1. API 키 설정 (여기를 본인의 키로 교체하세요!) ---
GOOGLE_API_KEY = 'YOUR_API_KEY_HERE' 

try:
    genai.configure(api_key=GOOGLE_API_KEY)
except ValueError as e:
    print("API 키 설정에 실패했습니다. 올바른 키로 교체했는지 확인하세요.")
    print(f"오류: {e}")

# --- 2. Gemini 핵심 기능 함수 (프롬프트 엔지니어링) ---
# 최신 Flash 모델 Alias 사용
model = genai.GenerativeModel('gemini-flash-latest')

def generate_summary(original_text):
    """Gemini를 호출하여 3가지 형식의 요약을 생성하는 함수"""
    
    # 핵심 프롬프트
    prompt = f"""
    당신은 텍스트를 분석하고 요약하는 전문 에디터입니다.
    아래 제공되는 <원본 텍스트>를 분석하여, 다음 3가지 항목을 **반드시 마크다운(Markdown) 형식**으로 정확하게 생성해 주세요.

    <원본 텍스트>
    {original_text}
    </원본 텍스트>

    ---
    
    ## 🎯 핵심 3줄 요약
    - [여기에 원본 텍스트의 핵심 내용을 3줄로 요약]
    - [두 번째 요약]
    - [세 번째 요약]

    ## 🔑 주요 키워드 (5개)
    - [키워드 1]
    - [키워드 2]
    - [키워드 3]
    - [키워드 4]
    - [키워드 5]

    ## 👶 쉬운 말 버전 (초등학생 수준)
    [여기에 원본 텍스트의 내용을 초등학생도 이해할 수 있도록 아주 쉽게 풀어서 설명]
    """
    
    try:
        response = model.generate_content(prompt)
        return response.text
    except Exception as e:
        return f"AI 모델 호출 중 오류가 발생했습니다: {e}"

# --- 3. Jupyter UI 위젯 생성 및 이벤트 연결 ---

# 텍스트 입력창
input_text = widgets.Textarea(
    value='(샘플) AI 기술이 빠르게 발전하면서, 인간의 일자리에 대한 우려도 커지고 있습니다. 하지만 전문가는 AI가 인간을 대체하는 것이 아니라, 강력한 조력자 역할을 할 것이라고 말합니다. 특히, AI에게 정확한 질문을 하고 원하는 결과를 얻어내는 \'프롬프트 엔지니어링\' 능력은 미래 사회의 핵심 경쟁력이 될 것입니다.',
    placeholder='여기에 긴 뉴스 기사나 텍스트를 붙여넣으세요...',
    description='원본 텍스트:',
    layout={'width': '90%', 'height': '200px'}
)

# 실행 버튼
generate_button = widgets.Button(
    description='✨ 변환하기',
    button_style='success',
    tooltip='클릭하면 AI가 텍스트를 요약합니다.',
    icon='magic'
)

# 결과 출력 영역
output_area = widgets.Output(
    layout={'border': '1px solid #ccc', 'padding': '10px', 'width': '90%'}
)

def on_button_clicked(b):
    with output_area:
        output_area.clear_output()
        print("🤖 AI가 텍스트를 분석 중입니다... 잠시만 기다려주세요...")
        
        text = input_text.value
        
        if not text.strip() or text.startswith('(샘플)'):
            output_area.clear_output()
            print("❗ 요약할 텍스트를 입력창에 붙여넣어 주세요.")
            return

        result = generate_summary(text)
        
        output_area.clear_output()
        display(Markdown(result))

# 버튼 클릭 이벤트 핸들러 연결
generate_button.on_click(on_button_clicked)

# 화면에 UI 표시
print("--- 🚀 3줄 요약 & 쉬운 말 변환기 (Gemini Flash 기반) ---")
display(input_text, generate_button, output_area)