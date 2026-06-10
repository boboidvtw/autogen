<a name="readme-top"></a>

<div align="center">
<img src="https://microsoft.github.io/autogen/0.2/img/ag.svg" alt="AutoGen Logo" width="100">

[![Twitter](https://img.shields.io/twitter/url/https/twitter.com/cloudposse.svg?style=social&label=Follow%20%40pyautogen)](https://twitter.com/pyautogen)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Company?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/company/105812540)
[![Discord](https://img.shields.io/badge/discord-chat-green?logo=discord)](https://aka.ms/autogen-discord)
[![Documentation](https://img.shields.io/badge/Documentation-AutoGen-blue?logo=read-the-docs)](https://microsoft.github.io/autogen/)
[![Blog](https://img.shields.io/badge/Blog-AutoGen-blue?logo=blogger)](https://devblogs.microsoft.com/autogen/)

[English](./README.md) | **繁體中文**

</div>

# AutoGen [![維護模式](https://img.shields.io/badge/status-maintenance%20mode-orange)](https://github.com/microsoft/agent-framework)

**AutoGen** 是一個用於建立多智能體 (Multi-Agent) AI 應用程式的框架，這些智能體可以自主運行，也可以與人類協同工作。

> [!CAUTION]
> **⚠️ 維護模式**
>
> AutoGen 目前處於維護模式。它將不再接收新的功能或增強，未來將由社群共同管理。
>
> 新使用者應從 [微軟智能體框架 (Microsoft Agent Framework)](https://github.com/microsoft/agent-framework) 開始。建議現有使用者參考 [AutoGen → Microsoft Agent Framework 遷移指南](https://learn.microsoft.com/zh-tw/agent-framework/migration-guide/from-autogen/) 進行遷移。
>
> Microsoft Agent Framework (MAF) 是 AutoGen 的企業級後繼者。MAF 現已發布生產就緒版本：提供穩定的 API，並承諾長期支援。無論您是建立單個助理還是協調一組專門的智能體，Microsoft Agent Framework 1.0 都能為您提供企業級的多智能體編排、多供應商模型支援，以及透過 A2A 和 MCP 實現的跨執行環境互操作性。

## 安裝說明

AutoGen 需要 **Python 3.10 或更高版本**。

```bash
# 從擴充功能安裝 AgentChat 和 OpenAI 用戶端
pip install -U "autogen-agentchat" "autogen-ext[openai]"
```

目前的穩定版本可以在 [releases](https://github.com/microsoft/autogen/releases) 中找到。如果您是從 AutoGen v0.2 升級，請參考 [遷移指南](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/migration-guide.html) 以取得有關如何更新程式碼與設定的詳細說明。

```bash
# 安裝 AutoGen Studio 以使用無程式碼的圖形化介面 (GUI)
pip install -U "autogenstudio"
```

## 快速上手

以下範例呼叫了 OpenAI API，因此您需要先建立帳戶並將您的 API 金鑰匯出為 `export OPENAI_API_KEY="sk-..."`。

### Hello World

使用 OpenAI 的 GPT-4o 模型建立一個助理智能體 (AssistantAgent)。請參閱[其他支援的模型](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/models.html)。

```python
import asyncio
from autogen_agentchat.agents import AssistantAgent
from autogen_ext.models.openai import OpenAIChatCompletionClient

async def main() -> None:
    model_client = OpenAIChatCompletionClient(model="gpt-4o")
    agent = AssistantAgent("assistant", model_client=model_client)
    print(await agent.run(task="請說 'Hello World!'"))
    await model_client.close()

asyncio.run(main())
```

### MCP 伺服器 (MCP Server)

建立一個使用 Playwright MCP 伺服器的網頁瀏覽助理智能體。

```python
# 首先運行 `npm install -g @playwright/mcp@latest` 來安裝 MCP 伺服器。
import asyncio
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import OpenAIChatCompletionClient
from autogen_ext.tools.mcp import McpWorkbench, StdioServerParams


async def main() -> None:
    model_client = OpenAIChatCompletionClient(model="gpt-4o")
    server_params = StdioServerParams(
        command="npx",
        args=[
            "@playwright/mcp@latest",
            "--headless",
        ],
    )
    async with McpWorkbench(server_params) as mcp:
        agent = AssistantAgent(
            "web_browsing_assistant",
            model_client=model_client,
            workbench=mcp, # 若有多個 MCP 伺服器，請將它們放在一個 list 中。
            model_client_stream=True,
            max_tool_iterations=10,
        )
        await Console(agent.run_stream(task="找出 microsoft/autogen 倉庫有多少位貢獻者"))


asyncio.run(main())
```

> **警告**: 請僅連接到您信任的 MCP 伺服器，因為它們可能會在您的本地環境中執行指令或洩露敏感資訊。

### 多智能體編排 (Multi-Agent Orchestration)

您可以使用 `AgentTool` 來建立一個基礎的多智能體編排設定。

```python
import asyncio

from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.tools import AgentTool
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import OpenAIChatCompletionClient


async def main() -> None:
    model_client = OpenAIChatCompletionClient(model="gpt-4o")

    math_agent = AssistantAgent(
        "math_expert",
        model_client=model_client,
        system_message="你是一位數學專家。",
        description="數學專家助理。",
        model_client_stream=True,
    )
    math_agent_tool = AgentTool(math_agent, return_value_as_last_message=True)

    chemistry_agent = AssistantAgent(
        "chemistry_expert",
        model_client=model_client,
        system_message="你是一位化學專家。",
        description="化學專家助理。",
        model_client_stream=True,
    )
    chemistry_agent_tool = AgentTool(chemistry_agent, return_value_as_last_message=True)

    agent = AssistantAgent(
        "assistant",
        system_message="你是一位通用助理。需要時請使用專家工具。",
        model_client=model_client,
        model_client_stream=True,
        tools=[math_agent_tool, chemistry_agent_tool],
        max_tool_iterations=10,
    )
    await Console(agent.run_stream(task="x^2 的積分是多少？"))
    await Console(agent.run_stream(task="水的分子量是多少？"))


asyncio.run(main())
```

若要深入了解更進階的多智能體編排與工作流，請閱讀
[AgentChat 文件](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/index.html)。

### AutoGen Studio

使用 AutoGen Studio 來原型設計並運行多智能體工作流，而無需編寫程式碼。

> **注意**：AutoGen Studio 旨在幫助您快速設計多智能體工作流的原型，並展示使用 AutoGen 構建的終端使用者介面範例。它**並非旨在作為生產就緒的應用程式**。我們鼓勵開發人員使用 AutoGen 框架來構建自己的應用程式，並自行實作部署應用程式所需的驗證、安全性和其他功能。詳情請參閱[安全性說明](https://microsoft.github.io/autogen/dev/user-guide/autogenstudio-user-guide/index.html#a-note-on-security)。

```bash
# 在 http://localhost:8080 運行 AutoGen Studio
autogenstudio ui --port 8080 --appdir ./my-app
```

## 為什麼選擇 AutoGen？

<div align="center">
  <img src="autogen-landing.jpg" alt="AutoGen Landing" width="500">
</div>

AutoGen 由微軟研究院（Microsoft Research）首創，開啟了實驗性多智能體編排模式的大門，並激發了社群靈感。雖然 AutoGen 目前處於維護模式，但現有使用者仍可繼續使用基於下方所述架構的框架。**對於新專案，我們推薦使用 [微軟智能體框架 (Microsoft Agent Framework)](https://github.com/microsoft/agent-framework)**，它總結了 AutoGen 的經驗教訓，並提供了企業級的支援。

AutoGen 框架採用了分層且可擴充的設計。各層具有清晰的職責劃分，並構建在底層之上。這種設計使您能夠在不同的抽象層次上使用該框架，從高階 API 到低階組件。

- [Core API](./python/packages/autogen-core/) 實作了訊息傳遞、事件驅動的智能體，以及本地和分散式的執行期，以提供靈活性與效能。它還支援 .NET 和 Python 的跨語言互操作。
- [AgentChat API](./python/packages/autogen-agentchat/) 實作了一個更簡單但有特定設計模式的 API，用於快速原型設計。此 API 構建在 Core API 之上，最接近 v0.2 使用者所熟悉的體驗，並支援常見的多智能體模式，如雙智能體對話或群組對話。
- [Extensions API](./python/packages/autogen-ext/) 支援第一方和第三方擴充功能，以持續擴展框架功能。它支援 LLM 用戶端（如 OpenAI、AzureOpenAI）的具體實作，以及程式碼執行等功能。

該生態系統還支援兩個不可或缺的 *開發者工具*：

<div align="center">
  <img src="https://media.githubusercontent.com/media/microsoft/autogen/refs/heads/main/python/packages/autogen-studio/docs/ags_screen.png" alt="AutoGen Studio 螢幕截圖" width="500">
</div>

- [AutoGen Studio](./python/packages/autogen-studio/) 提供了一個無程式碼的圖形介面 (GUI)，用於構建多智能體應用程式。
- [AutoGen Bench](./python/packages/agbench/) 提供了一個基準測試套件，用於評估智能體的效能。

您可以使用 AutoGen 框架和開發者工具為您的領域建立應用程式。例如，[Magentic-One](./python/packages/magentic-one-cli/) 是一個頂尖的多智能體團隊，使用 AgentChat API 和 Extensions API 構建，可以處理各種需要網頁瀏覽、程式碼執行和檔案處理的任務。

若要取得社群支援，請訪問我們的 [Discord 伺服器](https://aka.ms/autogen-discord) 或 [GitHub Discussions](https://github.com/microsoft/autogen/discussions)。請注意，AutoGen 目前由社群管理，回覆可能會有所延遲。

## 下一步去哪裡？

> **正在開始新專案？** 請前往 [微軟智能體框架 (Microsoft Agent Framework)](https://github.com/microsoft/agent-framework) 取得最新的多智能體功能與長期支援。
>
> **現有的 AutoGen 使用者？** 使用 [遷移指南](https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-autogen/) 進行過渡，或參考下方資源以取得目前的 AutoGen 文件。

<div align="center">

|               | [![Python](https://img.shields.io/badge/AutoGen-Python-blue?logo=python&logoColor=white)](./python)                                                                                                                                                                                                                                                                                                                | [![.NET](https://img.shields.io/badge/AutoGen-.NET-green?logo=.net&logoColor=white)](./dotnet)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | [![Studio](https://img.shields.io/badge/AutoGen-Studio-purple?logo=visual-studio&logoColor=white)](./python/packages/autogen-studio)                        |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 安裝說明  | [![Installation](https://img.shields.io/badge/Install-blue)](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/installation.html)                                                                                                                                                                                                                                                         | [![Install](https://img.shields.io/badge/Install-green)](https://microsoft.github.io/autogen/dotnet/dev/core/installation.html)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | [![Install](https://img.shields.io/badge/Install-purple)](https://microsoft.github.io/autogen/stable/user-guide/autogenstudio-user-guide/installation.html) |
| 快速上手    | [![Quickstart](https://img.shields.io/badge/Quickstart-blue)](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/quickstart.html#)                                                                                                                                                                                                                                                         | [![Quickstart](https://img.shields.io/badge/Quickstart-green)](https://microsoft.github.io/autogen/dotnet/dev/core/index.html)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | [![Usage](https://img.shields.io/badge/Quickstart-purple)](https://microsoft.github.io/autogen/stable/user-guide/autogenstudio-user-guide/usage.html#)      |
| 學習教學      | [![Tutorial](https://img.shields.io/badge/Tutorial-blue)](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/index.html)                                                                                                                                                                                                                                                          | [![Tutorial](https://img.shields.io/badge/Tutorial-green)](https://microsoft.github.io/autogen/dotnet/dev/core/tutorial.html)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | [![Usage](https://img.shields.io/badge/Tutorial-purple)](https://microsoft.github.io/autogen/stable/user-guide/autogenstudio-user-guide/usage.html#)        |
| API 參考文件 | [![API](https://img.shields.io/badge/Docs-blue)](https://microsoft.github.io/autogen/stable/reference/index.html#)                                                                                                                                                                                                                                                                                                 | [![API](https://img.shields.io/badge/Docs-green)](https://microsoft.github.io/autogen/dotnet/dev/api/Microsoft.AutoGen.Contracts.html)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | [![API](https://img.shields.io/badge/Docs-purple)](https://microsoft.github.io/autogen/stable/user-guide/autogenstudio-user-guide/usage.html)               |
| 套件列表      | [![PyPi autogen-core](https://img.shields.io/badge/PyPi-autogen--core-blue?logo=pypi)](https://pypi.org/project/autogen-core/) <br> [![PyPi autogen-agentchat](https://img.shields.io/badge/PyPi-autogen--agentchat-blue?logo=pypi)](https://pypi.org/project/autogen-agentchat/) <br> [![PyPi autogen-ext](https://img.shields.io/badge/PyPi-autogen--ext-blue?logo=pypi)](https://pypi.org/project/autogen-ext/) | [![NuGet Contracts](https://img.shields.io/badge/NuGet-Contracts-green?logo=nuget)](https://www.nuget.org/packages/Microsoft.AutoGen.Contracts/) <br> [![NuGet Core](https://img.shields.io/badge/NuGet-Core-green?logo=nuget)](https://www.nuget.org/packages/Microsoft.AutoGen.Core/) <br> [![NuGet Core.Grpc](https://img.shields.io/badge/NuGet-Core.Grpc-green?logo=nuget)](https://www.nuget.org/packages/Microsoft.AutoGen.Core.Grpc/) <br> [![NuGet RuntimeGateway.Grpc](https://img.shields.io/badge/NuGet-RuntimeGateway.Grpc-green?logo=nuget)](https://www.nuget.org/packages/Microsoft.AutoGen.RuntimeGateway.Grpc/) | [![PyPi autogenstudio](https://img.shields.io/badge/PyPi-autogenstudio-purple?logo=pypi)](https://pypi.org/project/autogenstudio/)                          |

</div>

有興趣貢獻嗎？請參閱 [貢獻指南 (CONTRIBUTING.md)](./CONTRIBUTING.md)。由於 AutoGen 目前處於維護模式，貢獻僅限於錯誤修復、安全修補和文件改進。若要進行新功能開發，請考慮貢獻至 [微軟智能體框架 (Microsoft Agent Framework)](https://github.com/microsoft/agent-framework)。

有任何問題嗎？請查看我們的 [常見問題解答 (FAQ)](./FAQ.md) 以獲取解答。社群支援可透過 [GitHub Discussions](https://github.com/microsoft/autogen/discussions) 和 [Discord 伺服器](https://aka.ms/autogen-discord) 取得，但由於 AutoGen 目前由社群管理，回覆時間可能會有所不同。有關目前積極支援的工具，請參閱 [Microsoft Agent Framework](https://github.com/microsoft/agent-framework)。

## 法律聲明

微軟和任何貢獻者根據 [Creative Commons Attribution 4.0 International Public License](https://creativecommons.org/licenses/by/4.0/legalcode) 授權條款授予您本倉庫中微軟文件和其他內容的許可，請參閱 [LICENSE](LICENSE) 檔案；並根據 [MIT 授權條款](https://opensource.org/licenses/MIT) 授予您本倉庫中任何程式碼的許可，請參閱 [LICENSE-CODE](LICENSE-CODE) 檔案。

文件中引用的微軟、Windows、Microsoft Azure 和/或其他微軟產品及服務可能是微軟在美國和/或其他國家的商標或註冊商標。此專案的許可並不授予您使用任何微軟名稱、標誌或商標的權利。微軟的一般商標指引可在 <http://go.microsoft.com/fwlink/?LinkID=254653> 找到。

隱私權資訊可在 <https://go.microsoft.com/fwlink/?LinkId=521839> 找到。

微軟和任何貢獻者保留所有其他權利，無論是版權、專利還是商標，無論是透過暗示、禁反言還是其他方式。

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
  <a href="#readme-top" style="text-decoration: none; color: blue; font-weight: bold;">
    ↑ 回到頂部 ↑
  </a>
</p>
