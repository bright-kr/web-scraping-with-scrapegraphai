# ScrapeGraphAI를 활용한 LLM 기반 Webスクレイピング

[![Promo](https://github.com/bright-kr/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/)

이 가이드는 ScrapeGraphAI와 대규모 언어 모델(LLM)을 사용하여 Webスクレイピング을 단순화하고 데이터 추출을 자동화하는 방법을 설명합니다.

- [왜 ScrapeGraphAI를 사용해야 하나요?](#why-use-scrapegraphai)
- [사전 준비 사항](#prerequisites)
- [환경 설정](#setting-up-your-environment)
- [ScrapeGraphAI로 데이터 스クレイピング하기](#scraping-data-with-scrapegraphai)
  - [스크레이퍼 코드 작성](#writing-the-scraper-code)
  - [ScrapeGraphAI에서 プロキシ 사용하기](#using-proxies-with-scrapegraphai)
- [데이터 정리 및 준비](#cleaning-and-preparing-data)

## Why use ScrapeGraphAI?

기존 Webスクレイピング은 각 웹사이트 레이아웃에 맞춘 복잡하고 시간이 많이 드는 코드를 작성해야 하며, 사이트가 변경되면 종종 동작이 깨지곤 합니다.

[ScrapeGraphAI](https://scrapegraphai.com/)는 대규모 언어 모델(LLM)을 활용하여 사람처럼 데이터를 해석하고 추출하므로, 레이아웃이 아니라 데이터 자체에 집중할 수 있습니다. LLM을 통합함으로써 ScrapeGraphAI는 데이터 추출을 개선하고, 콘텐츠 집계를 자동화하며, 실시간 분석을 가능하게 합니다.

## Prerequisites

다음 사전 준비 사항이 필요합니다:

* [Python 3.x](https://www.python.org/downloads/).
* [OpenAI 계정](https://platform.openai.com/signup) — [GPT-4](https://openai.com/index/gpt-4/)에 접근하기 위해 필요합니다.

## Setting Up Your Environment

가상 환경을 생성합니다:

```bash
python -m venv venv
```

그다음 가상 환경을 활성화합니다. macOS 및 Linux에서는 다음을 실행합니다:

```bash
source venv/bin/activate
```

Windows에서는 다음 명령을 사용할 수 있습니다:

```powershell
venv\Scripts\activate
```

ScrapeGraphAI와 의존성을 설치합니다:

```bash
pip install scrapegraphai
playwright install
```

`playwright install` 명령은 Chromium, Firefox, WebKit에 필요한 브라우저를 설정합니다.

환경 변수를 안전하게 관리하기 위해 `python-dotenv`를 설치합니다:

```bash
pip install python-dotenv
```

API key와 같은 민감한 정보는 보호하는 것이 중요합니다. 이를 위해 환경 변수를 `.env` 파일에 저장하여 코드 파일과 분리해 두어야 합니다.

프로젝트 디렉터리에 `.env`라는 새 파일을 만들고, OpenAI key를 지정하는 다음 줄을 추가합니다:

```python
OPENAI_API_KEY="your-openai-api-key"
```

이 파일은 Git과 같은 버전 관리 시스템에 커밋하면 안 됩니다. 이를 방지하려면 `.gitignore` 파일에 `.env`를 추가합니다.

## Scraping Data with ScrapeGraphAI

Webスクレイピング 기법 연습을 위해 특별히 만들어진 데모 웹사이트인 [Books to Scrape](http://books.toscrape.com/)에서 제품 데이터를 스クレイピング하는 것부터 시작합니다. 이 웹사이트는 온라인 서점을 모방하고 있으며, 다양한 장르의 책을 가격, 평점, 재고 상태와 함께 제공합니다:

![Books to Scrape website](https://github.com/bright-kr/web-scraping-with-scrapegraphai/blob/main/images/Books-to-Scrape-website-1024x772.png)

기존 HTML 스クレイピング에서는 데이터를 추출하기 위해 요소를 수동으로 검사합니다. ScrapeGraphAI에서는 프롬프트로 원하는 데이터를 지정하기만 하면 LLM이 대신 추출해 줍니다.

ScrapeGraphAI는 다양한 스クレイピング 요구에 맞춘 여러 graph를 제공합니다:

* **[SmartScraperGraph](https://scrapegraph-ai.readthedocs.io/en/latest/scrapers/types.html#smartscrapermultigraph)**: 프롬프트와 URL 또는 로컬 파일을 사용하는 단일 페이지 스크레이퍼입니다.
* **[SearchGraph](https://scrapegraph-ai.readthedocs.io/en/latest/scrapers/types.html#searchgraph)**: 검색 엔진 결과에서 데이터를 추출하는 다중 페이지 스크레이퍼입니다.
* **[SpeechGraph](https://scrapegraph-ai.readthedocs.io/en/latest/scrapers/types.html#speechgraph)**: 텍스트-투-스피치를 추가하여 오디오 파일을 생성하는 SmartScraperGraph 확장입니다.
* **[ScriptCreatorGraph](https://scrapegraph-ai.readthedocs.io/en/latest/scrapers/types.html#scriptcreatorgraph-scriptcreatormultigraph)**: 지정한 URL을 스크레이핑하기 위한 Python 스크립트를 출력합니다.

또한 특정 요구에 맞도록 노드를 조합하여 커스텀 graph를 만들 수도 있습니다.

정확한 스크레이핑을 보장하려면 명확한 프롬프트, 모델 선택, 지오로케이션 제한 콘텐츠를 위한 プロキシ, 효율성을 위한 headless 모드 등을 포함하여 스크레이퍼를 적절히 구성해야 합니다. 올바른 설정은 추출된 데이터의 정밀도에 영향을 미칩니다.

### Writing the Scraper Code

`app.py`라는 새 파일을 만들고 다음 코드를 삽입합니다:

```python
from dotenv import load_dotenv
import os
from scrapegraphai.graphs import SmartScraperGraph

# Load environment variables from .env file
load_dotenv()

# Access the OpenAI API key
OPENAI_API_KEY = os.getenv('OPENAI_API_KEY')

# Configuration for ScrapeGraphAI
graph_config = {
    "llm": {
        "api_key": OPENAI_API_KEY,
        "model": "openai/gpt-4o-mini",
 }
}

# Define the prompt and source
prompt = "Extract the title, price and availability of all books on this page."
source = "http://books.toscrape.com/"

# Create the scraper graph
smart_scraper_graph = SmartScraperGraph(
 prompt=prompt,
 source=source,
 config=graph_config
)

# Run the scraper
result = smart_scraper_graph.run()

# Output the results
print(result)
```

이 코드는 환경 변수를 관리하기 위한 `os` 및 `dotenv` 같은 필수 모듈과, 스크레이핑을 위한 ScrapeGraphAI의 `SmartScraperGraph` 클래스를 가져옵니다. 또한 `dotenv`를 통해 환경 변수를 로드하여 민감한 데이터(예: API key)를 안전하게 유지합니다. 이후 스크레이핑에 사용할 LLM을 구성하며, 모델과 API key를 지정합니다. 이 구성과 사이트 URL, 스크레이핑 프롬프트를 바탕으로 `SmartScraperGraph`를 정의하고, `run()` 메서드로 실행하여 지정한 데이터를 수집합니다.

코드를 실행하려면 터미널에서 `python app.py` 명령을 사용합니다. 출력은 다음과 같이 표시됩니다:

```
{
    "books": [
        {
            "title": "A Light in the Attic",
            "price": "£51.77",
            "availability": "In stock"
        },
        {
            "title": "Tipping the Velvet",
            "price": "£53.74",
            "availability": "In stock"
        }, ...
       ]
}
```

> **Note**:
> 
> 잠재적인 오류를 방지하려면 [`grpcio`](https://pypi.org/project/grpcio/) 패키지가 설치되어 있는지 확인하십시오.

ScrapeGraphAI는 Webスクレイピング의 데이터 추출 부분을 쉽게 만들어 주지만, CAPTCHA와 IP 차단 같은 일반적인 과제는 여전히 존재합니다.

브라우징 동작을 모방하기 위해 코드에 시간 지연을 구현할 수 있습니다. 또한 탐지를 피하기 위해 ローテーティングプロキシ를 활용할 수도 있습니다. 추가로 Bright Data의 CAPTCHA solver 또는 Anti Captcha 같은 CAPTCHA 해결 서비스를 스크레이퍼에 통합하면 CAPTCHA를 자동으로 해결할 수 있습니다.

> **Important**:
> 
> 항상 웹사이트의 이용 약관을 준수하고 있는지 확인하십시오. 개인적 사용을 위한 스크레이핑은 종종 허용되지만, 데이터를 재배포하는 경우 법적 문제가 발생할 수 있습니다.

## Using Proxies with ScrapeGraphAI

ScrapeGraphAI에서는 IP 차단을 피하고 지오로케이션 제한 페이지에 접근하기 위해 プロキシ 서비스를 설정할 수 있습니다. 이를 위해 `graph_config`에 다음을 추가합니다:

```python
graph_config = {
    "llm": {
        "api_key": OPENAI_API_KEY,
        "model": "openai/gpt-4o-mini",
 },
    "loader_kwargs": {
        "proxy": {
            "server": "broker",
            "criteria": {
                "anonymous": True,
                "secure": True,
                "countryset": {"US"},
                "timeout": 10.0,
                "max_tries": 3
 },
 },
 }
}
```

이 구성은 ScrapeGraphAI가 지정한 criteria에 맞는 무료 プロキシ 서비스를 사용하도록 지시합니다.

Bright Data 같은 제공업체의 커스텀 プロキシ 서버를 사용하려면, server URL, username, password를 넣어 다음과 같이 `graph_config`를 변경합니다:

```python
graph_config = {
    "llm": {
        "api_key": OPENAI_API_KEY,
        "model": "openai/gpt-4o-mini",
 },
    "loader_kwargs": {
        "proxy": {
            "server": "http://your_proxy_server:port",
            "username": "your_username",
            "password": "your_password",
 },
 }
}
```

커스텀 プロキシ 서버 사용은 특히 대규모 Webスクレイピング에 여러 이점을 제공합니다. プロキシ 위치를 제어할 수 있으므로 지오로케이션 제한 콘텐츠에 접근할 수 있습니다. 또한 커스텀 プロキシ는 무료 プロキ시보다 더 안정적이고 안전하여, IP 차단이나 レート制限 위험을 줄입니다.

## Cleaning and Preparing Data

스크레이핑 이후에는 데이터를 정리하고 전처리하는 것이 중요하며, 특히 AI 모델에 사용할 경우 더욱 그렇습니다. 깨끗한 데이터는 모델이 정확하고 일관된 정보로부터 학습하도록 보장하여 성능과 신뢰성을 향상시킵니다. 데이터 정리는 일반적으로 결측값 처리, 데이터 타입 교정, 텍스트 정규화, 중복 제거를 포함합니다.

다음은 [pandas](https://pypi.org/project/pandas/)를 사용하여 스크레이핑한 데이터를 정리하는 예시입니다:

```python
import pandas as pd

# Convert the result to a DataFrame
df = pd.DataFrame(result["books"])

# Remove currency symbols and convert prices to float
df['price'] = df['price'].str.replace('£', '').astype(float)

# Standardize availability text
df['availability'] = df['availability'].str.strip().str.lower()

# Handle missing values if any
df.dropna(inplace=True)

# Preview the cleaned data
print(df.head())
```

이 코드는 책 가격에서 통화 기호를 제거하고, availability 상태를 소문자로 변환해 표준화하며, 결측값이 있을 경우 이를 처리함으로써 데이터를 정리합니다.

해당 코드를 실행하기 전에 데이터 조작을 위한 `pandas` 라이브러리를 설치합니다:

```bash
pip install pandas
```

터미널을 열고 `python app.py`를 실행합니다. 출력은 다음과 같이 표시되어야 합니다:

```
                                   title  price availability
0                   A Light in the Attic  51.77     in stock
1                     Tipping the Velvet  53.74     in stock
2                             Soumission  50.10     in stock
3                          Sharp Objects  47.82     in stock
4  Sapiens: A Brief History of Humankind  54.23     in stock
```

이는 스크레이핑된 데이터를 정리하는 예시일 뿐이며, 실제 과정은 데이터와 LLM 사용 사례에 따라 달라집니다. 정리는 언어 모델이 구조화되고 의미 있는 입력을 받도록 보장합니다. 가장 인기 있는 [AI 사용 사례](https://brightdata.co.kr/ai)에 대해 알아보십시오.

이 튜토리얼의 전체 코드는 [이 GitHub repo](https://github.com/mikeyny/scrapegraphai-demo)에서 확인할 수 있습니다.

## Conclusion

ScrapeGraphAI는 LLM을 사용하여 적응형 Webスクレイピング을 수행하며, 웹사이트 변경에 맞춰 조정하고 데이터를 지능적으로 추출합니다. 그러나 스크레이핑을 확장하는 과정에서는 IP 차단, CAPTCHA, 법적 준수와 같은 과제가 발생합니다.

Bright Data는 [Web Scraper APIs](https://brightdata.co.kr/products/web-scraper), [プロキシ 서비스](https://brightdata.co.kr/proxy-types), [Serverless Scraping](https://brightdata.co.kr/products/web-scraper/functions) 등 이러한 과제를 해결하기 위한 솔루션을 제공합니다. 또한 100개가 넘는 인기 웹사이트에서 수집한 [바로 사용할 수 있는 データセット](https://brightdata.co.kr/products/datasets)도 제공합니다.

지금 무료 체험을 시작해 보십시오!