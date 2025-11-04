# C 語言建置與驗證流程 (GitHub Actions Demo)
這是一個簡單的示範專案，展示如何使用 **GitHub Actions** 自動化建置 (Compile)、運行 (Run) 和驗證 (Validate) 一個基本的 C 語言程式。

## 專案簡介
本專案包含一個簡單的 C 語言程式 main.c，以及一個 GitHub Actions 工作流程配置檔案 .github/workflows/c_build.yml，用於在每次程式碼推送 (Push) 時自動執行以下任務：
1. 取得程式碼 (Checkout)
2. 編譯程式 (GCC)
3. 運行並驗證程式 (Execution)
4. 上傳建置成品 (Artifact)

## 檔案結構
```.
├── .github/
│   └── workflows/
│       └── c_build.yml  # GitHub Actions 工作流程定義
└── main.c               # 核心 C 語言程式
```

## main.c
此程式的功能是簡單地印出一串問候語，並計算並印出 $5 \times 32$ 的結果。
```
#include <stdio.h>

int main(){
  printf("Hello From C Build on Github Actions!\n");
  int a=5;
  int b=32;
  int mul = a * b;
  printf("%d\n", mul); // 預期輸出 160
  return 0;
}
```

## GitHub Actions 工作流程 (`c_build.yml`)

這個範例的工作流程名稱為 "**C 語言建置流程**"，它會在任何分支發生 push 事件時被觸發，並在最新的 **Ubuntu** 虛擬機上執行。

### Build Job 步驟詳解

#### 1. Checkout code
指令：`actions/checkout@v4`
功能：取得 repo 中的程式碼。

#### 2. Compile C program (GCC)
指令：`gcc main.c -o hello_c_app`
功能：使用 GCC 編譯器，將 `main.c` 編譯成名為 `hello_c_app` 的可執行檔。

#### 3. Run compiled application
指令：`./hello_c_app`
功能：執行編譯後的可執行檔。此步驟會輸出程式的執行結果 (例如：`Hello...` 和 `160`)，並作為工作流程的驗證日誌。

#### 4. Upload build artifact
指令：`actions/upload-artifact@v4`
功能：將編譯好的 `hello_c_app` 可執行檔上傳為名為 `hello-c-executable` 的建置成品 (Artifact)，方便使用者下載驗證。

## 如何運行與查看結果
1. **推送程式碼**： 
將本專案的程式碼 (特別是 `.github/workflows/c_build.yml` 和 `main.c`) `push` 到 GitHub 倉庫的任何分支。

2. **查看 Actions**： 
導航至 GitHub 倉庫的 **Actions** 頁籤。

3. **選擇工作流程**： 
點擊名為 "**C 語言建置流程**" 的運行紀錄。

4. **檢查日誌**：
在 **Run compiled application** 步驟中，可以查看到程式的實際輸出 (`Hello From C Build...` 和 `160`)，以驗證程式是否正確運行。

5. **下載成品**：
在工作流程運行成功後，可以在摘要頁面找到並下載名為 `hello-c-executable` 的 **Artifact**，這就是編譯好的程式。
