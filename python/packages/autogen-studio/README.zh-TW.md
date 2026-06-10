# AutoGen Studio

[![PyPI version](https://badge.fury.io/py/autogenstudio.svg)](https://badge.fury.io/py/autogenstudio)
![PyPI - Downloads](https://img.shields.io/pypi/dm/autogenstudio)

![ARA](https://media.githubusercontent.com/media/microsoft/autogen/refs/heads/main/python/packages/autogen-studio/docs/ags_screen.png)

[English](./README.md) | **繁體中文**

AutoGen Studio 是一個由 AutoGen 驅動的 AI 應用程式（使用者介面），旨在幫助您快速設計 AI 智能體的原型、透過技能（Skills）增強其功能、將其編排為工作流（Workflows），並透過與其互動來完成任務。它基於 [AutoGen](https://microsoft.github.io/autogen) 框架構建，該框架是用於構建 AI 智能體的工具包。

AutoGen Studio 的程式碼託管於 GitHub：[microsoft/autogen](https://github.com/microsoft/autogen/tree/main/python/packages/autogen-studio)

> [!CAUTION]
> AutoGen Studio 旨在幫助您快速設計多智能體工作流的原型，並展示使用 AutoGen 構建的終端使用者介面範例。它**並非旨在作為生產就緒的應用程式**。我們鼓勵開發人員使用 [AutoGen 框架](https://microsoft.github.io/autogen) 來構建自己的應用程式，並自行實作部署應用程式所需的驗證、安全性和其他功能。

> [!WARNING]
> AutoGen Studio 目前處於積極開發階段。後續版本中可能會引入破壞性變更（Breaking Changes）。

## 安全性說明

AutoGen Studio 是一個研究原型，**不適用於**生產環境。我們鼓勵使用一些基準安全實踐，例如為您的智能體使用 Docker 程式碼執行環境。

然而，其他安全性考量，如與越獄（Jailbreaking）相關的嚴格測試、確保 LLM 僅能在獲得終端使用者授權的情況下訪問正確的資料金鑰，以及其他安全性功能，皆尚未在 AutoGen Studio 中實作。

如果您正在構建生產級應用程式，請使用 [AutoGen 框架](https://microsoft.github.io/autogen) 並自行實作必要的安全性功能。

## 更新日誌

- **2024-11-14:** AutoGen Studio 正在重寫，以使用更新的 AutoGen 0.4.0 API (AgentChat API)。
- **2024-04-17:** AutoGen Studio 資料庫層現已重寫以使用 [SQLModel](https://sqlmodel.tiangolo.com/)（Pydantic + SQLAlchemy）。這提供了實體連結（技能、模型、智能體和工作流透過關聯表連結），並支援 SQLAlchemy 支援的多種[資料庫後端方言](https://docs.sqlalchemy.org/en/20/dialects/)（SQLite, PostgreSQL, MySQL, Oracle, Microsoft SQL Server）。運行應用程式時，可使用 `--database-uri` 參數來指定後端資料庫。例如，SQLite 可使用 `autogenstudio ui --database-uri sqlite:///database.sqlite`，PostgreSQL 可使用 `autogenstudio ui --database-uri postgresql+psycopg://user:password@localhost/dbname`。
- **2024-03-12:** AutoGen Studio 的預設目錄現已更改為 `/home/<USER>/.autogenstudio`。您也可以在運行應用程式時使用 `--appdir` 參數來指定此目錄。例如：`autogenstudio ui --appdir /path/to/folder`。這會將資料庫和其他檔案儲存在指定的目錄中，例如 `/path/to/folder/database.sqlite`。該目錄中的 `.env` 檔案將用於設定應用程式的環境變數。

## 專案結構：

- `autogenstudio/` 包含後端類別與 Web API (FastAPI) 的程式碼。
- `frontend/` 包含 Web UI 的程式碼，使用 Gatsby 和 TailwindCSS 構建。

## 安裝說明

安裝 AutoGen Studio 有兩種方式：從 PyPi 安裝或從原始碼安裝。除非您計劃修改原始碼，否則我們**推薦從 PyPi 安裝**。

### 1. 從 PyPi 安裝（推薦）

我們建議使用虛擬環境（例如 venv）以避免與現有的 Python 套件衝突。在虛擬環境中啟用 Python 3.10 或更新版本後，使用 pip 安裝 AutoGen Studio：

```bash
pip install -U autogenstudio
```

### 2. 從原始碼安裝

*注意：此方法需要對 React 介面構建有一定了解。*

#### 重要提示：Git LFS 需求

AutoGen Studio 使用 Git 大檔案儲存 (Git LFS) 來管理圖片和其他大型檔案。如果您在沒有安裝 git-lfs 的情況下 Clone 此倉庫，將會遇到與圖片格式相關的建置錯誤。

**在 Clone 倉庫之前：**

1. 安裝 git-lfs：

   ```bash
   # 在 Debian/Ubuntu 上
   apt-get install git-lfs

   # 在 macOS 上使用 Homebrew
   brew install git-lfs

   # 在 Windows 上使用 Chocolatey
   choco install git-lfs
   ```

2. 設定 git-lfs：
   ```bash
   git lfs install
   ```

**如果您已經 Clone 了倉庫：**

```bash
git lfs install
git lfs fetch --all
git lfs checkout  # 下載所有缺失的圖片檔案至工作目錄
```

如果您使用開發容器（Dev Container）方式安裝，這些設定將會自動處理。

您可以選擇以下兩種方式之一來進行原始碼安裝：手動安裝或使用開發容器。

#### A) 手動從原始碼安裝

1. 確保您已安裝 Python 3.10+ 和 Node.js（版本高於 14.15.0）。
2. Clone AutoGen Studio 倉庫，並使用 `pip install -e .` 安裝 Python 依賴項。
3. 導航至 `python/packages/autogen-studio/frontend` 目錄，安裝依賴項並建置 UI：

   ```bash
   npm install -g gatsby-cli
   npm install --global yarn
   cd frontend
   yarn install
   yarn build
   # Windows 使用者可能需要替代命令來建置前端：
   gatsby clean && rmdir /s /q ..\\autogenstudio\\web\\ui 2>nul & (set "PREFIX_PATH_VALUE=" || ver>nul) && gatsby build --prefix-paths && xcopy /E /I /Y public ..\\autogenstudio\\web\\ui
   ```

#### B) 使用開發容器 (Dev Container) 安裝

1. 參考 [Dev Containers 教學](https://code.visualstudio.com/docs/devcontainers/tutorial) 安裝 VS Code、Docker 與相關擴充功能。
2. Clone AutoGen Studio 倉庫。
3. 在 VS Code 中開啟 `python/packages/autogen-studio/`。點擊左下角的藍色按鈕或按下 F1，並選擇 *"Dev Containers: Reopen in Container"*。
4. 建置 UI：

   ```bash
   cd frontend
   yarn build
   ```

### 執行應用程式

安裝完成後，在終端機輸入以下指令來啟動 Web UI：

```bash
autogenstudio ui --port 8081
```

此指令將在指定的連接埠啟動應用程式。開啟您的網頁瀏覽器並前往 <http://localhost:8081/> 即可開始使用 AutoGen Studio。

AutoGen Studio 還支援多個自訂參數：

- `--host <host>`：指定主機位址。預設為 `localhost`。
- `--appdir <appdir>`：指定儲存應用程式檔案（例如資料庫和產生的使用者檔案）的目錄。預設為使用者家目錄下的 `.autogenstudio` 目錄。
- `--port <port>`：指定連接埠號。預設為 `8080`。
- `--reload`：程式碼變更時自動重載伺服器。預設為 `False`。
- `--database-uri`：指定資料庫 URI。例如 SQLite 的 `sqlite:///database.sqlite` 或 PostgreSQL 的 `postgresql+psycopg://user:password@localhost/dbname`。若未指定，資料庫 URL 預設為 `--appdir` 目錄下的 `database.sqlite` 檔案。
- `--upgrade-database`：將資料庫結構升級至最新版本。預設為 `False`。

現在您已成功安裝並啟動 AutoGen Studio，可以開始探索其功能，包括定義和修改智能體工作流、與智能體和會話進行互動，以及擴展智能體的技能。

## AutoGen Studio Lite

AutoGen Studio Lite 提供了一種輕量級的方式來快速設計 AI 智能體團隊的原型並進行實驗。它專為快速實驗而設計，無需設定完整的資料庫。

### 命令列使用 (CLI Usage)

從命令列啟動 Studio Lite：

```bash
# 使用預設團隊快速啟動
autogenstudio lite

# 使用自訂團隊檔案
autogenstudio lite --team ./my_team.json --port 8080

# 自訂會話名稱並自動開啟瀏覽器
autogenstudio lite --session-name "My Experiment" --auto-open
```

### 程式碼中呼叫 (Programmatic Usage)

直接在您的 Python 程式碼中使用 Studio Lite：

```python
from autogenstudio.lite import LiteStudio

# 使用預設團隊快速啟動
studio = LiteStudio()

# 使用 AutoGen 團隊物件
from autogen_agentchat.teams import RoundRobinGroupChat
team = RoundRobinGroupChat([agent1, agent2], termination_condition=...)

# 使用上下文管理器 (Context Manager)
with LiteStudio(team=team) as studio:
    # Studio 會在背景運行
    # 在此處執行其他工作
    pass
```

#### 本地前端開發伺服器

請參閱 `./frontend/README.md`

## 貢獻指南

我們非常歡迎對 AutoGen Studio 的貢獻！建議以下列通用步驟參與專案：

- 閱讀 AutoGen 專案的 [貢獻指南](https://github.com/microsoft/autogen?tab=readme-ov-file#contributing)
- 請查看 AutoGen Studio [開發藍圖 (Roadmap)](https://github.com/microsoft/autogen/issues/4006) 以了解當前的優先任務。非常歡迎協助處理標記為 `help-wanted` 的 Studio 問題。
- 請在開發藍圖 Issue 中發起討論，或建立一個新的 Issue 來討論您計劃貢獻的內容。
- 提交 Pull Request (PR) 貢獻您的程式碼！
- 如果您要修改 AutoGen Studio，它有自己的開發容器。請參閱 `.devcontainer/README.md` 中的說明來使用。
- 與 Studio 相關的所有 Issue、問題和 PR，請使用 `proj-studio` 標籤。

## 常見問題解答 (FAQ)

請參考 AutoGen Studio [FAQs](https://microsoft.github.io/autogen/docs/autogen-studio/faqs) 頁面以獲取更多資訊。

## 鳴謝

AutoGen Studio 基於 [AutoGen](https://microsoft.github.io/autogen) 專案。它改編自 2023 年 10 月建立的研究原型（原創成員：Gagan Bansal, Adam Fourney, Victor Dibia, Piali Choudhury, Saleema Amershi, Ahmed Awadallah, Chi Wang）。
