# C 語言建置與驗證流程 - 加強版 (GitHub Actions Demo)

## 專案簡介
本專案 fork 自 [Bensonlllll/build_on_github_test](https://github.com/Bensonlllll/build_on_github_test)，在原本的基礎上新增了以下功能：

### 新增功能
- **使用 curl 下載測試資料**：自動從網路下載文字檔案作為測試輸入
- **檔案 I/O 處理**：改寫 `main.c` 使用命令列參數讀取輸入檔並寫入輸出檔
- **自動化測試驗證**：比對輸入與輸出檔案，確保程式正確性

### GitHub Actions 工作流程
本專案使用 `.github/workflows/c_build.yml` 配置檔案，在每次程式碼推送時自動執行：
1. 取得程式碼 (Checkout)
2. 編譯 C 程式 (GCC)
3. 使用 curl 下載測試資料
4. 執行檔案複製程式
5. 驗證輸出結果 (自動化測試)
6. 上傳建置成品 (Artifact)

## 檔案結構

```text
.
├── .github/
│   └── workflows/
│       └── c_build.yml  # GitHub Actions 工作流程定義
└── main.c               # 核心 C 語言程式（檔案複製功能）
```

## main.c

此程式改寫為檔案複製工具，使用命令列參數指定輸入和輸出檔案：

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]){
  // 檢查命令列參數是否正確
  if(argc != 3){
    printf("使用方式: %s <輸入檔案> <輸出檔案>\n", argv[0]);
    return 1;
  }
  
  // 開啟輸入檔案並讀取
  FILE *input_file = fopen(argv[1], "r");
  if(input_file == NULL){
    printf("錯誤: 無法開啟輸入檔案 '%s'\n", argv[1]);
    return 1;
  }
  
  // 開啟輸出檔案並寫入
  FILE *output_file = fopen(argv[2], "w");
  if(output_file == NULL){
    printf("錯誤: 無法開啟輸出檔案 '%s'\n", argv[2]);
    fclose(input_file);
    return 1;
  }
  
  // 逐字元讀取並寫入
  int ch;
  while((ch = fgetc(input_file)) != EOF){
    fputc(ch, output_file);
  }
  
  // 關閉檔案
  fclose(input_file);
  fclose(output_file);
  
  printf("成功將 '%s' 複製到 '%s'\n", argv[1], argv[2]);
  return 0;
}
```

## GitHub Actions 工作流程 (`c_build.yml`)

這個加強版工作流程名稱為 "**C 語言建置流程**"，它會在任何分支發生 push 事件時被觸發，並在最新的 **Ubuntu** 虛擬機上執行。

### Build Job 步驟詳解

#### 1. Checkout code

指令：`actions/checkout@v4`
功能：取得 repo 中的程式碼。

#### 2. Compile C program (GCC)

指令：`gcc main.c -o by_pass_c`
功能：使用 GCC 編譯器，將 `main.c` 編譯成名為 `by_pass_c` 的可執行檔。

#### 3. Download text file with curl

指令：`curl -o cano.txt https://sherlock-holm.es/stories/plain-text/cano.txt`
功能：使用 curl 從網路下載 "The Adventure of the Yellow Face" 文字檔作為測試資料。

#### 4. Run compiled application

指令：`./by_pass_c cano.txt output.txt`
功能：執行編譯後的程式，讀取 `cano.txt` 並複製到 `output.txt`。

#### 5. Verify output file

指令：`diff cano.txt output.txt`
功能：使用 diff 命令比對輸入和輸出檔案。
- 如果內容相同：顯示 "✓ 測試通過: output.txt 與 cano.txt 內容相同"
- 如果內容不同：顯示 "✗ 測試失敗: output.txt 與 cano.txt 內容不同" 並回傳錯誤，導致 workflow 失敗

#### 6. Upload build artifact

指令：`actions/upload-artifact@v4`
功能：將編譯好的 `by_pass_c` 可執行檔上傳為建置成品 (Artifact)，方便下載驗證。

## 如何運行與查看結果

1. **推送程式碼**：
   將本專案的程式碼推送到 GitHub 倉庫的任何分支。

2. **查看 Actions**：
   導航至 GitHub 倉庫的 **Actions** 頁籤。

3. **選擇工作流程**：
   點擊名為 "**C 語言建置流程**" 的運行紀錄。

4. **檢查日誌**：
   在各個步驟中查看執行結果，特別是 **Verify output file** 步驟可以確認測試是否通過。

5. **下載成品**：
   在工作流程運行成功後，可以在摘要頁面找到並下載名為 `by_pass_c-executable` 的 **Artifact**。

## 改進重點

相較於原始版本，本專案加入了：

1. **實際的檔案 I/O 操作**：程式從簡單的 printf 改為完整的檔案讀寫功能
2. **網路資料下載**：使用 curl 自動獲取測試資料
3. **自動化測試**：使用 diff 驗證程式輸出的正確性
4. **錯誤處理**：測試失敗時會導致 CI 流程失敗，確保程式品質


## 問與答（many thanks to ChatGPT!)
### 問題1: https://github.com/Bensonlllll/build_on_github_test 這個 repo 是較有關 CI/CD 嗎？
#### 答案1:
是的，這個 repo 的重點就是示範 CI/CD（比較偏 CI）。

從 README 可以看到，它在示範如何用 GitHub Actions 自動做這幾件事： ￼
1. 自動建置 (Continuous Integration) 
    * 每次 push 到任何分支就觸發 workflow。
    * 在 Ubuntu runner 上執行 gcc main.c -o hello_c_app 做編譯。
2. 自動執行與驗證程式
    * 在 workflow 裡跑 ./hello_c_app，確認程式能正常輸出結果（Hello From C Build... 和 160）。
3. 自動上傳建置產物 (Artifact)
    * 使用 actions/upload-artifact@v4，把編好的 hello_c_app 上傳成 artifact，讓人可以從 Actions 頁面下載。

所以這個 repo 比較像是：

「用 GitHub Actions 做 C 程式的自動建置與驗證的教學範例」

屬於 CI/CD 裡面的 CI + 一點點 Delivery（產物輸出），
還沒有做到真正「部署到伺服器 / 使用者端」那種完整 CD。

接下來若要進一步變成「完整 CI/CD」，就可以在 workflow 裡再加：
* 自動打包成壓縮檔 / 安裝檔
* 上傳到 GitHub Releases 或 GitHub Packages
* 或是部署到某個測試伺服器（例如用 scp、ssh、docker push 等）。

---

### 問題2: .github/workflows/c_build.yml 這個是什麼？有沒有比較正式的名稱？
#### 答案2:
它就是 GitHub Actions 的工作流程設定檔（workflow file）。
比較正式一點可以這樣叫：
* GitHub Actions workflow file
* 或中文：GitHub Actions 工作流程設定檔 / CI 工作流程檔

順便拆一下這個檔名與路徑的意思：
* github/workflows/
  * 這是 GitHub 規定放 Actions 設定檔的資料夾。
  * 只要有放在這裡的 .yml / .yaml 檔，GitHub 就會把它當作一個 workflow。
* c_build.yml
  * 只是你自訂的檔名，用來表示「這個 workflow 是拿來 build C 程式的」。
  * 在 GitHub UI 上顯示的工作流程名稱，則是檔案裡 name: 那一行（如果有寫）；
  * 檔名只是在 repo 裡方便你區分不同 workflow。

---

### 問題3: 這些 CI/CD 的 workflows actions 是誰提供的服務和資源？
#### 答案3:
這些東西大致上可以拆成三個「角色」來看，就比較清楚是誰提供什麼：

1. CI/CD 平台與跑程式的機器：GitHub 提供
  * 你在 .github/workflows/*.yml 裡寫的這些 workflows，實際上是交給 GitHub Actions 這個服務來執行。
  * GitHub（背後是 Microsoft 的雲端資源） 會在你 push、開 PR 等時，啟動一台暫時的虛擬機（runner），例如：
  * runs-on: ubuntu-latest → 就是在 GitHub 的雲端開一台 Ubuntu VM
  * 也可以是 windows-latest、macos-latest
  * 這些 VM 的 硬體資源、網路、作業系統 等，都是 GitHub 幫你準備好 的（對 public repo 免費、有使用額度限制）。
  
簡單講：
CI/CD 平台（GitHub Actions） + 執行工作用的機器（GitHub-hosted runners）都是 GitHub 提供的服務和資源。


2. workflow 檔（c_build.yml）：你／repo 作者提供
  * 	.github/workflows/c_build.yml 這個檔案裡寫的是：
  * 要在什麼事件觸發（push、pull_request…）
  * 要跑哪些步驟（checkout、安裝工具、編譯、測試、上傳 artifact…）
  * 這些內容是 repo 的維護者（也就是你或專案作者）自己寫的設定，GitHub 只是照著你寫的流程在雲端跑。


3. 裡面用到的 actions（小積木）：GitHub 或社群提供

在 steps: 裡常看到這種東西：

uses: actions/checkout@v4
uses: actions/upload-artifact@v4

  * actions/checkout、actions/upload-artifact 這些叫做 actions，就像「現成的小模組」。
  * 有些是 GitHub 官方維護 的（像上面那兩個）。
  * 也有很多是 社群 / 公司 / 個人開發者 放到 GitHub Marketplace 上給大家用的。
  
所以：
  * 平台與運算資源：GitHub 提供
  * 流程邏輯（workflow yml）：你或專案作者寫
  * actions 模組：GitHub 官方 + 社群共同提供
  
