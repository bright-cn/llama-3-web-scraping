# 使用 LLaMA 3 进行网页爬虫

[![Bright Data 推广](https://github.com/bright-cn/Awesome-Web-Scraping/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner%20CN.png)](https://www.bright.cn/)

本指南介绍如何使用 LLaMA 3 将大型 HTML 转换为结构化、整洁且可用的 JSON：

- [为什么选择 LLaMA 3 进行网页爬虫](#为什么选择-llama-3-进行网页爬虫)
- [系统要求](#系统要求)
- [设置 Ollama](#设置-ollama)
- [选择合适的 LLaMA 模型](#选择合适的-llama-模型)
- [下载并运行模型](#下载并运行模型)
- [构建由 LLM 驱动的 Amazon 爬虫工具](#构建由-llm-驱动的-amazon-爬虫工具)
- [处理反机器人保护](#处理反机器人保护)
- [增强并扩展你的抓取工具](#增强并扩展你的抓取工具)

## 为什么选择 LLaMA 3 进行网页爬虫

[Meta 的 LLaMA 3](https://ai.meta.com/blog/meta-llama-3/)（于 2024 年 4 月首次发布）是一个开放权重 LLM 系列，参数规模从 8B 到 405B，适用于各种任务和硬件配置。3.1 至 3.3 版本的更新进一步增强了其能力。

使用 [XPath 或 CSS](https://www.bright.cn/blog/web-data/xpath-vs-css-selectors) 的传统抓取技术很容易受到网站布局变化的影响。而 LLaMA 3 能以类似人类的方式理解内容，从而实现智能、灵活的抓取，即使网站更新也能保持可靠。

因此，它非常适合：

- Amazon 等零售巨头
- 复杂的数据解析
- 稳定耐用的爬虫工具
- 确保敏感数据保留在内部

你可以通过我们[之前的指南](https://www.bright.cn/blog/web-data/ai-web-scraping)详细了解基于 AI 的网页爬虫。

## 系统要求

开始这个[由 LLM 驱动的抓取](https://www.bright.cn/blog/web-data/web-scraping-with-scrapegraphai)项目之前，请确保你具备：

- [Python 3](https://www.python.org/downloads/)
- 基本的 Python 知识
- 以下任一操作系统环境：
  - macOS 11 Big Sur 或更高版本
  - Linux
  - Windows 10 或更高版本
- 充足的计算机资源（下文将详细介绍）

## 设置 Ollama

Ollama 简化了在本地安装、运行和管理大语言模型的流程。

![Ollama 安装页面](https://github.com/bright-cn/llama-3-web-scraping/blob/main/images/ollama-llm-download-installation-page.png)

开始使用：

1. 前往 [Ollama 官方网站](https://ollama.com/)
2. 下载与你的操作系统匹配的版本
3. **重要提示**：设置过程中，系统会要求你运行一条命令，请先不要运行，等选好模型后再执行。

## 选择合适的 LLaMA 模型

浏览 [Ollama 的模型库](https://ollama.com/library)，找到合适的版本。

如果你使用的是普通配置的计算机，`llama3.1:8b` 是一个绝佳选择：体积小、效率高，仅需要约 4.9 GB 磁盘空间和 6–8 GB 内存。它可以在大多数现代笔记本电脑上顺畅运行。

如果你的硬件性能更强，`70B` 或 `405B` 等更大的版本可以提供更出色的推理能力和更长的上下文窗口，但它们对硬件资源的要求也更高。

## 下载并运行模型

使用以下命令拉取 LLaMA 3.1（8B）模型：

```sh
ollama run llama3.1:8b
```

你将看到一个交互式提示：

```sh
>>> Send a message (/? for help)
```

测试模型：

```sh
>>> who are you?
I am LLaMA, *an AI assistant developed by Meta AI...*
```

确认无误后，启动 Ollama 服务器：

```sh
ollama serve
```

这将在 `http://127.0.0.1:11434/` 启动一个本地服务器。请保持此窗口开启。

在浏览器中访问该地址，你应该会看到消息 **“Ollama 正在运行。”**

## 构建由 LLM 驱动的 Amazon 爬虫工具

让我们构建一个从 Amazon 提取商品详情的抓取工具。由于其[动态内容](https://www.bright.cn/blog/how-tos/scrape-dynamic-websites-python)和强大的反机器人保护，Amazon 是最具挑战性的抓取目标之一。

![Amazon 商品页面](https://github.com/bright-cn/llama-3-web-scraping/blob/main/images/amazon-office-chair-product-page-1.png)

我们将提取：

- 标题
- 价格
- 折扣
- 评分和评论数量
- 描述和特性
- 库存状态和 [ASIN](https://www.bright.cn/blog/web-data/how-to-scrape-amazon-asin)

### 智能多阶段方法

由 LLaMA 驱动的爬虫工具采用智能的多阶段工作流程：

1. 使用 Selenium 进行**浏览器自动化**
2. 从目标部分进行 **HTML 提取**
3. 通过 **Markdown 转换**精简输入
4. 使用 **LLM 处理**实现数据结构化
5. 通过**结果存储**支持进一步分析

以下是该工作流程的可视化说明：

![工作流程图](https://github.com/bright-cn/llama-3-web-scraping/blob/main/images/llama-web-scraping-workflow-diagram.png)

我们将使用 **Python**，但也可以调整为使用 [JavaScript](https://www.bright.cn/blog/web-data/best-languages-web-scraping) 等其他语言。

### 第 1 步：安装所需库

首先，安装所需的 Python 库：

```sh
pip install requests selenium webdriver-manager markdownify
```

- `requests`：[最佳 Python HTTP 客户端](https://www.bright.cn/blog/web-data/best-python-http-clients)，用于向 LLM 服务发送 API 调用
- `selenium`：实现浏览器自动化，非常适合大量使用 JavaScript 的网站
- `webdriver-manager`：自动下载并管理正确版本的 ChromeDriver
- `markdownify`：将 HTML 转换为 Markdown

### 第 2 步：初始化无头浏览器

使用 Selenium 设置一个[无头浏览器](https://www.bright.cn/blog/proxy-101/what-is-a-headless-browser)：

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager

options = Options()
options.add_argument("--headless")

driver = webdriver.Chrome(
    service=Service(ChromeDriverManager().install()),
    options=options
)
```

### 第 3 步：提取商品 HTML

Amazon 商品详情采用动态渲染，并封装在 `<div id="ppd">` 容器中。脚本将等待此部分加载，然后提取其 HTML：

```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

wait = WebDriverWait(driver, 15)
product_container = wait.until(
    EC.presence_of_element_located((By.ID, "ppd"))
)

# Extract the full HTML of the product container
page_html = product_container.get_attribute("outerHTML")
```

此方法会：

- 等待由 JavaScript 渲染的内容，例如价格和评分
- 仅定位相关的商品部分，忽略页眉、页脚和侧边栏

_查看关于[如何使用 Python 抓取 Amazon 商品数据](https://www.bright.cn/blog/how-tos/how-to-scrape-amazon)的完整指南。_

### 第 4 步：将 HTML 转换为 Markdown

Amazon 页面包含深层嵌套的 HTML，LLM 处理起来效率很低。最好将这些 HTML 转换为整洁的 Markdown，以移除多余数据，从而减少 token 数量并提高理解能力。

运行完整脚本后，将生成两个文件：`amazon_page.html` 和 `amazon_page.md`。尝试将两者粘贴到 [Token 计算器工具](https://token-calculator.net/)中，比较它们的 token 数量。

HTML 包含约 **270,000 个 token**：

![token-calculator-html-tokens](https://github.com/bright-cn/llama-3-web-scraping/blob/main/images/token-calculator-html-tokens.png)

Markdown 版本仅包含 **约 11,000 个 token**：

![token-calculator-markdown-tokens](https://github.com/bright-cn/llama-3-web-scraping/blob/main/images/token-calculator-markdown-tokens.png)

这一 **96% 的缩减**带来了以下优势：

- **提高成本效益**：token 越少，API 或计算成本越低
- **加快处理速度**：输入数据越少，LLM 响应越快
- **提高准确性**：更整洁、更扁平的文本有助于模型更准确地提取结构化数据

_详细了解[为什么 AI 智能体更偏好 Markdown 而非 HTML](https://hackernoon.com/why-are-the-new-ai-agents-choosing-markdown-over-html)。_

以下是使用 Python 进行转换的方法：

```python
from markdownify import markdownify as md

clean_text = md(page_html, heading_style="ATX")
```

### 第 5 步：创建数据提取提示词

结构良好的提示词对于让 LLM 持续输出整洁一致的 JSON 至关重要。以下提示词会要求模型**仅**以预定义格式返回有效的 JSON：

```python
PROMPT = (
    "You are an expert Amazon product data extractor. Your task is to extract product data from the provided content. "
    "Return ONLY valid JSON with EXACTLY the following fields and formats:\n\n"
    "{\n"
    '  "title": "string – the product title",\n'
    '  "price": number – the current price (numerical value only)",\n'
    '  "original_price": number or null – the original price if available,\n'
    '  "discount": number or null – the discount percentage if available,\n'
    '  "rating": number or null – the average rating (0–5 scale),\n'
    '  "review_count": number or null – total number of reviews,\n'
    '  "description": "string – main product description",\n'
    '  "features": ["string"] – list of bullet point features,\n'
    '  "availability": "string – stock status",\n'
    '  "asin": "string – 10-character Amazon ID"\n'
    "}\n\n"
    "Return ONLY the JSON without any additional text."
)
```

### 第 6 步：调用 LLM API

通过 HTTP API 将 Markdown 文本发送到你的 LLaMA 实例：

```python
import requests
import json

response = requests.post(
    "<http://localhost:11434/api/generate>",
    json={
        "model": "llama3.1:8b",
        "prompt": f"{PROMPT}\n\n{clean_text}",
        "stream": False,
        "format": "json",
        "options": {
            "temperature": 0.1,
            "num_ctx": 12000,
        },
    },
    timeout=250,
)

raw_output = response.json()["response"].strip()
product_data = json.loads(raw_output)
```

以下是每个选项的作用：

- `temperature`：设置为 0.1，以获得确定性输出（非常适合 JSON 格式）
- `num_ctx`：定义最大上下文长度。对于大多数 Amazon 商品页面，12,000 个 token 已经足够
- `stream`：当设置为 `False` 时，API 会在处理完成后返回完整响应
- `format`：指定输出格式（JSON）
- `model`：指定要使用的 LLaMA 版本

当然！以下是更简洁的改写：

转换后的 Markdown 通常包含约 11,000 个 token，因此应相应地设置上下文窗口（`num_ctx`）。增大上下文窗口可以支持更长的输入，但会占用更多内存并降低处理速度。仅在必要或资源充足时增大该值。

### 第 7 步：保存结果

最后，将结构化商品数据保存到 JSON 文件：

```python
with open("product_data.json", "w", encoding="utf-8") as f:
    json.dump(product_data, f, indent=2, ensure_ascii=False)
```

### 第 8 步：执行脚本

要运行你的爬虫工具，请提供一个 Amazon 商品 URL 并调用抓取函数：

```python
if __name__ == "__main__":
    url = "<https://www.amazon.com/Black-Office-Chair-Computer-Adjustable/dp/B00FS3VJAO>"

    # Call your function to scrape and extract product data
    scrape_amazon_product(url)
```

### 第 9 步：完整代码示例

以下是完整的 Python 脚本：

```python
import json
import logging
import time
from typing import Final, Optional, Dict, Any

import requests
from markdownify import markdownify as html_to_md
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait
from webdriver_manager.chrome import ChromeDriverManager

# Configuration constants
LLM_API_CONFIG: Final[Dict[str, Any]] = {
    "endpoint": "<http://localhost:11434/api/generate>",
    "model": "llama3.1:8b",
    "temperature": 0.1,
    "context_window": 12000,
    "stream": False,
    "timeout_seconds": 220,
}

DEFAULT_PRODUCT_DATA: Final[Dict[str, Any]] = {
    "title": "",
    "price": 0.0,
    "original_price": None,
    "discount": None,
    "rating": None,
    "review_count": None,
    "description": "",
    "features": [],
    "availability": "",
    "asin": "",
}

PRODUCT_DATA_EXTRACTION_PROMPT: Final[str] = (
    "You are an expert Amazon product data extractor. Your task is to extract product data from the provided content. "
    "Return ONLY valid JSON with EXACTLY the following fields and formats:\n\n"
    "{\n"
    '  "title": "string - the product title",\n'
    '  "price": number - the current price (numerical value only),\n'
    '  "original_price": number or null - the original price if available,\n'
    '  "discount": number or null - the discount percentage if available,\n'
    '  "rating": number or null - the average rating (0-5 scale),\n'
    '  "review_count": number or null - total number of reviews,\n'
    '  "description": "string - main product description",\n'
    '  "features": ["string"] - list of bullet point features,\n'
    '  "availability": "string - stock status",\n'
    '  "asin": "string - 10-character Amazon ID"\n'
    "}\n\n"
    "Return ONLY the JSON without any additional text."
)

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
    handlers=[logging.StreamHandler()],
)

def initialize_web_driver(headless: bool = True) -> webdriver.Chrome:
    """Initialize and return a configured Chrome WebDriver instance."""
    options = Options()
    if headless:
        options.add_argument("--headless=new")

    service = Service(ChromeDriverManager().install())
    return webdriver.Chrome(service=service, options=options)

def fetch_product_container_html(product_url: str) -> Optional[str]:
    """Retrieve the HTML content of the Amazon product details container."""
    driver = initialize_web_driver()
    try:
        logging.info(f"Accessing product page: {product_url}")
        driver.set_page_load_timeout(15)
        driver.get(product_url)

        # Wait for the product container to appear
        container = WebDriverWait(driver, 5).until(
            EC.presence_of_element_located((By.ID, "ppd"))
        )
        return container.get_attribute("outerHTML")
    except Exception as e:
        logging.error(f"Error retrieving product details: {str(e)}")
        return None
    finally:
        driver.quit()

def extract_product_data_via_llm(markdown_content: str) -> Optional[Dict[str, Any]]:
    """Extract structured product data from markdown text using LLM API."""
    try:
        logging.info("Extracting product data via LLM API...")
        response = requests.post(
            LLM_API_CONFIG["endpoint"],
            json={
                "model": LLM_API_CONFIG["model"],
                "prompt": f"{PRODUCT_DATA_EXTRACTION_PROMPT}\n\n{markdown_content}",
                "format": "json",
                "stream": LLM_API_CONFIG["stream"],
                "options": {
                    "temperature": LLM_API_CONFIG["temperature"],
                    "num_ctx": LLM_API_CONFIG["context_window"],
                },
            },
            timeout=LLM_API_CONFIG["timeout_seconds"],
        )
        response.raise_for_status()

        raw_output = response.json()["response"].strip()
        # Clean JSON output if it's wrapped in markdown code blocks
        if raw_output.startswith(("```json", "```")):
            raw_output = raw_output.split("```")[1].strip()
            if raw_output.startswith("json"):
                raw_output = raw_output[4:].strip()

        return json.loads(raw_output)

    except requests.exceptions.RequestException as e:
        logging.error(f"LLM API request failed: {str(e)}")
        return None
    except json.JSONDecodeError as e:
        logging.error(f"Failed to parse LLM response: {str(e)}")
        return None
    except Exception as e:
        logging.error(f"Unexpected error during data extraction: {str(e)}")
        return None

def scrape_amazon_product(
    product_url: str, output_file: str = "product_data.json"
) -> None:
    """Scrape an Amazon product page and save extracted data along with HTML and Markdown to files."""
    start_time = time.time()
    logging.info(f"Starting scrape for: {product_url}")

    # Step 1: Fetch product page HTML
    product_html = fetch_product_container_html(product_url)
    if not product_html:
        logging.error("Failed to retrieve product page content")
        return

    # Optional: save HTML for debugging
    with open("amazon_product.html", "w", encoding="utf-8") as f:
        f.write(product_html)

    # Step 2: Convert HTML to Markdown
    product_markdown = html_to_md(product_html)

    # Optional: save Markdown for debugging
    with open("amazon_product.md", "w", encoding="utf-8") as f:
        f.write(product_markdown)

    # Step 3: Extract structured data via LLM
    product_data = (
        extract_product_data_via_llm(product_markdown) or DEFAULT_PRODUCT_DATA.copy()
    )

    # Step 4: Save JSON results
    try:
        with open(output_file, "w", encoding="utf-8") as json_file:
            json.dump(product_data, json_file, indent=2, ensure_ascii=False)
        logging.info(f"Successfully saved product data to {output_file}")
    except IOError as e:
        logging.error(f"Failed to save JSON results: {str(e)}")

    elapsed_time = time.time() - start_time
    logging.info(f"Completed in {elapsed_time:.2f} seconds")

if __name__ == "__main__":
    # Example usage
    test_url = (
        "<https://www.amazon.com/Black-Office-Chair-Computer-Adjustable/dp/B00FS3VJAO>"
    )
    scrape_amazon_product(test_url)
```

该脚本会将提取的商品数据保存到名为 `product_data.json` 的文件中。输出将类似于：

```json
{
    "title": "Home Office Chair Ergonomic Desk Chair Mesh Computer Chair with Lumbar Support Armrest Executive Rolling Swivel Adjustable Mid Back Task Chair for Women Adults, Black",
    "price": 36.98,
    "original_price": 41.46,
    "discount": 11,
    "rating": 4.3,
    "review_count": 58112,
    "description": 'Office chair comes with all hardware and tools, and is easy to assemble in about 10–15 minutes. The high-density sponge cushion offers flexibility and comfort, while the mid-back design and rectangular lumbar support enhance ergonomics. All components are BIFMA certified, supporting up to 250 lbs. The chair includes armrests and an adjustable seat height (17.1"–20.3"). Its ergonomic design ensures a perfect fit for long-term use.',
    "features": [
        "100% mesh material",
        "Quick and easy assembly",
        "High-density comfort seat",
        "BIFMA certified quality",
        "Includes armrests",
        "Ergonomic patented design",
    ],
    "availability": "In Stock",
    "asin": "B00FS3VJAO",
}
```

## 处理反机器人保护

运行上述[网页爬虫机器人](https://www.bright.cn/blog/how-tos/what-is-a-scraping-bot)时，你很可能会遇到 Amazon 的反机器人措施，例如 CAPTCHA 验证：

![amazon-captcha-anti-bot-challenge](https://github.com/bright-cn/llama-3-web-scraping/blob/main/images/amazon-captcha-anti-bot-challenge.png)

虽然 LLaMA 3 能出色地处理解析任务，但绕过网站保护仍然很棘手。[Bright Data 抓取浏览器](https://www.bright.cn/products/scraping-browser)提供了强大的解决方案。

### 为什么使用 Bright Data 抓取浏览器

[Bright Data 抓取浏览器](https://www.bright.cn/products/scraping-browser)是一款基于云端的无头浏览器，专为扩展现代网页爬虫项目而设计。它内置代理基础设施和高级解锁功能，是 [Bright Data Unlocker 抓取套件](https://docs.brightdata.com/scraping-automation/introduction)的一部分。

选择它的部分原因包括：

- 可靠的 TLS 指纹和隐蔽规避能力
- 通过[拥有超过 1.5 亿个住宅 IP 的代理网络](https://www.bright.cn/proxy-types/residential-proxies)实现内置 IP 轮换
- 自动破解 CAPTCHA
- 降低基础设施成本：无需配置或维护云环境
- 原生支持 Playwright、Puppeteer 和 Selenium
- 可无限扩展，满足大规模数据提取需求

最重要的是，只需几行代码即可将其集成到你的工作流程中。

### 设置抓取浏览器

要开始使用抓取浏览器：

[创建 Bright Data 账户](https://www.bright.cn/)（新用户添加付款方式后可获得 5 美元赠金），然后在控制面板中前往**代理与抓取**，并点击**开始使用**。

![Bright Data 抓取解决方案控制面板](https://github.com/bright-cn/llama-3-web-scraping/blob/main/images/brightdata-scraping-solutions-dashboard.png)

创建一个新区域（例如 _test\_browser_），并启用_高级域名_和 [CAPTCHA 破解](https://www.bright.cn/products/web-unlocker/captcha-solver)等功能。

![Bright Data 创建抓取浏览器区域](https://github.com/bright-cn/llama-3-web-scraping/blob/main/images/brightdata-create-scraping-browser-zone.png)

接下来，从控制面板复制 Selenium URL。

![Bright Data Selenium 连接凭据](https://github.com/bright-cn/llama-3-web-scraping/blob/main/images/brightdata-selenium-connection-credentials.png)

### 修改代码以使用抓取浏览器

更新你的 `initialize_web_driver` 函数，以通过抓取浏览器进行连接：

```python
from selenium.webdriver import Remote
from selenium.webdriver.chrome.options import Options as ChromeOptions
from selenium.webdriver.chromium.remote_connection import ChromiumRemoteConnection

SBR_WEBDRIVER = "<https://username:password@host>:port"

def initialize_web_driver():
    options = ChromeOptions()
    sbr_connection = ChromiumRemoteConnection(SBR_WEBDRIVER, "goog", "chrome")
    driver = Remote(sbr_connection, options=options)
    return driver
```

你的爬虫工具现在会通过 Bright Data 的基础设施路由，并可轻松应对 Amazon 及其他反机器人系统。

## 增强并扩展你的抓取工具

以下是你将来可以添加的一些改进：

- 让 URL 和提示词参数可配置
- 从 `.env` 文件安全加载凭据
- 支持多页面抓取和[分页处理](https://www.bright.cn/blog/web-data/pagination-web-scraping)
- 将抓取扩展到[其他电商平台](https://www.bright.cn/blog/how-tos/ecommerce-web-scraping-guide)
- 从 Google 服务中提取数据：
  - [Google 航班](https://github.com/bright-cn/google-flights-api)
  - [Google 搜索](https://github.com/bright-cn/google-search-api)
  - [Google 趋势](https://github.com/bright-cn/google-trends-api)
- 探索不同的 LLM 集成，例如：
  - [Gemini](https://www.bright.cn/blog/web-data/web-scraping-with-gemini)
  - [Perplexity](https://www.bright.cn/blog/web-data/web-scraping-with-perplexity)
  - [Crawl4AI 和 DeepSeek](https://www.bright.cn/blog/web-data/crawl4ai-and-deepseek-web-scraping)

## 总结

本指南可帮助你使用 LLaMA 3 创建可靠、智能的爬虫工具。

要取得最佳抓取效果，请将 LLaMA 的推理能力与 [Bright Data 抓取浏览器](https://www.bright.cn/products/scraping-browser)等工具的基础设施相结合。

准备好更进一步了吗？免费[试用 Bright Data 的完整抓取套件](https://www.bright.cn/)！
