以下提醒(記得刪掉):
1.架構都幫你弄好了，照著架構打就好了
2.[]代表自己打的部分
3.功能123下方[]看你是要打一下原理還是遇到的困難都可，要不要放添加"部分"的程式碼就看你了(老師說不要全貼，太多了，我自己是有放添加功能部分的程式碼)
4.後面括弧備註的部分，不用刪，放在你打字的最後即可，那是備註程式碼上船的檔案
5.記得上傳五個程式碼文件檔
6.記得看一下圖片位子對不對，圖片你要自己下載放上去


# 期末報告：踩地雷小遊戲
- 組員：11328204 彭泓昇 / 11328203 邱健峰
## 摘要
經過上課練習的數獨專題，想了一下什麼遊戲與數獨類似，都需要印出版面？並且版面的分布就是解謎關鍵？而踩地雷的靈感就出現在了我們的腦中，所以決定製作踩地雷小遊戲作為期末專題。
## 程式架構
#### 由組員 <u>邱健峰</u> 完成。
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define SIZE 5        // 地圖大小 5x5
#define MINES 5       // 地雷數量

// 顯示地圖內容：0~8 為鄰近地雷數，-1 為地雷
void print_map(int *map, int *revealed) {
    printf("  ");
    for (int x = 0; x < SIZE; x++) printf("%d ", x);
    printf("\n");

    for (int y = 0; y < SIZE; y++) {
        printf("%d ", y);
        for (int x = 0; x < SIZE; x++) {
            int idx = y * SIZE + x;
            if (*(revealed + idx))
                if (*(map + idx) == -1)
                    printf("* ");
                else
                    printf("%d ", *(map + idx));
            else
                printf("# ");
        }
        printf("\n");
    }
}

// 計算地雷鄰近數
int count_adjacent_mines(int *map, int x, int y) {
    int count = 0;
    for (int dy = -1; dy <= 1; dy++) {
        for (int dx = -1; dx <= 1; dx++) {
            int nx = x + dx, ny = y + dy;
            if (nx >= 0 && nx < SIZE && ny >= 0 && ny < SIZE) {
                if (*(map + ny * SIZE + nx) == -1)
                    count++;
            }
        }
    }
    return count;
}

// 初始化地圖：隨機佈雷並計算鄰近地雷數
void init_map(int *map) {
    for (int i = 0; i < SIZE * SIZE; i++) {
        *(map + i) = 0;
    }

    int placed = 0;
    while (placed < MINES) {
        int idx = rand() % (SIZE * SIZE);
        if (*(map + idx) != -1) {
            *(map + idx) = -1;
            placed++;
        }
    }

    for (int y = 0; y < SIZE; y++) {
        for (int x = 0; x < SIZE; x++) {
            if (*(map + y * SIZE + x) != -1)
                *(map + y * SIZE + x) = count_adjacent_mines(map, x, y);
        }
    }
}

int main() {
    srand(time(NULL));
    int *map = malloc(sizeof(int) * SIZE * SIZE);
    int *revealed = calloc(SIZE * SIZE, sizeof(int)); // 0: 未翻開，1: 翻開

    init_map(map);

    int game_over = 0;
    while (!game_over) {
        print_map(map, revealed);

        int x, y;
        printf("輸入座標 (x y): ");
        scanf("%d %d", &x, &y);

        if (x < 0 || x >= SIZE || y < 0 || y >= SIZE) {
            printf("座標超出範圍。\n");
            continue;
        }

        int idx = y * SIZE + x;
        *(revealed + idx) = 1;

        if (*(map + idx) == -1) {
            printf("踩到地雷！遊戲結束。\n");
            game_over = 1;
            // 顯示全部地圖
            for (int i = 0; i < SIZE * SIZE; i++) revealed[i] = 1;
            print_map(map, revealed);
        }
    }

    free(map);
    free(revealed);
    return 0;
}
```
- 架構測試：
![](20250611222101_遊戲架構.png)
- 這個架構已能簡單輸入行列座標，並提醒座標是否有效，也能提醒該格子周遭地雷數，當踩到地雷則遊戲結束。
## 功能添加1
#### 此部分由組員 <u>邱健峰</u> 添加 **插旗、儲存遊戲進度、自動展開空白格**功能。 (備註：附件 功能添加2)
![](20250611233143_功能測試.png)
- 功能測試：

![](20250611232650_功能添加2.png)
- 功能測試：
## 功能添加2
**計時功能：** 即時記錄玩家自開始遊戲以來所花費的秒數，並於每輪操作時更新顯示。

**難度選擇：** 提供不同地圖大小與炸彈數量的選項（後期擴增如初級 9x9、進階 16x16），讓玩家能依照能力挑戰。

**挑戰次數限制：** 限制玩家最多可翻開的格數，若達上限仍存活即視為過關，有別於傳統掃雷的清圖目標。

(備註：附件 功能添加1)
### 功能1:計時
**使用技巧：** 使用 `time(NULL)` 擷取 UNIX 時間點，於 `init_map()` 或讀取存檔時儲存 `start_time`，每次顯示地圖時透過 `time(NULL) - start_time` 計算已用時間。

**遇到的困難：** 初期是以遊戲結束（掃雷勝利或失敗）為條件，提供玩家掃雷的總共嘗試時間，但一般常見遊戲的遊玩時間是以即時更新提醒使用者，並提供因時間累積而帶來的壓力感。若玩家無法在執行一步後得知所花時間，似乎也讓「挑戰」一詞失去意義。

**解決辦法：** 改善為玩家若每在終端機執行下一步(無論是翻格、插旗/取消插旗都算一步)，均會提醒遊玩時間，該時間為自遊戲開始時就開始累計。
### 功能2:難度選擇
**使用技巧：** 透過函式 `choose_difficulty()` 提供選單，改用變數 `SIZE` 與 `MINES` 取代固定常數，並動態配置地圖記憶體，傳遞地圖尺寸與地雷數。

**遇到的困難：** 原始程式以固定常數設計，變更尺寸需全面調整所有涉及地圖的操作與記憶體配置，執行起來相當的複雜。

**解決辦法：** 將地圖尺寸與地雷數從 `#define` 改為全域變數，並更新所有相關函式使用動態變數，使地圖可隨難度調整，讓隨機生成的地圖更加靈活。
### 功能3:挑戰次數
**使用技巧：** 設立常數 `MAX_MOVES` 表示玩家最多可翻格的次數。透過 `count_moves(revealed)` 計算當前已翻格數，於每輪後判斷是否達上限。

**遇到的困難：**  在早期版本中，未排除遞迴翻開時一併翻開的多格，導致挑戰次數瞬間耗盡，與預期不符。挑戰次數也較不人性，給玩家的容錯率偏低。

**解決辦法：** 調整 `reveal_recursive()` 的行為與挑戰次數判斷位置，讓其統一在主迴圈中以實際翻格狀態 `revealed[][]` 來計算次數，確保公平一致。

### 功能測試
![](20250611222748_功能添加1.png)
## 頁面優化
**使用技巧：** 本掃雷遊戲在終端機中使用簡潔清晰的文字介面設計。為了提升可讀性與趣味性，採用了：
- Unicode 符號（如 🚩、💣、■）表示旗子、地雷與未翻開格子。
- 表格式座標軸，橫列顯示欄號、左側標示列號，協助玩家快速定位。
- 分段式訊息（如「操作選項」、「剩餘挑戰次數」、「已用時間」）讓使用者操作直觀明確。

**遇到的困難：** 顯示混亂、資訊太雜亂、座標輸入錯誤
- 早期版本在不同終端機中對Unicode支援程度不同，容易導致排版錯位或亂碼。
- 將地圖與遊戲提示混合顯示時，玩家不容易分辨重點資訊。
- 玩家常誤解輸入格式，如輸入 `(y,x)` 或錯誤座標，導致程式崩潰。

**解決辦法：** 簡化並模組化輸出、使用固定寬度字元、加入輸入檢查、優化提示順序
- 將 `print_map()` 函式中地圖顯示與資訊提示分段設計，保證版面整齊。
- 避免使用寬度不一致的中文字元，所有格子用寬度一致的「空格＋符號＋空格」來保持對齊。
- 在讀取玩家輸入時加入座標合法性檢查，防止程式錯誤終止。
- 先印地圖，再列出遊戲狀態與選項，建立「從遊戲狀態 → 操作」的流程習慣，提升互動直覺性。
(備註：附件 最終優化版本)
![](20250611232849_最終成果.png)
### 優化後的介面
## 成果展示
經過兩人的努力，終於完成了踩地雷小遊戲。
![](20250611233433_start.png)
- 成果1
![](20250611233440_play.png)
- 成果2

## 分工表
|11328203 邱健峰| 11328204 彭泓昇|
| - | - |
|程式架構 |計時功能 |
|插旗功能 |難度選擇功能 |
|自動展開空白格功能 |挑戰次數功能 |
|進度儲存功能 |介面優化 |
## 心得
[]
