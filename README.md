# Concurrency Test Project

測試 GitHub Actions concurrency 機制的行為。

## 測試目的

驗證 `cancel-in-progress: true` 的行為：
- 當新的 workflow run 觸發時，是否會取消進行中的 run
- 同一個 run 內的多個 jobs 是否會依序執行（因為共用 concurrency group）

## Workflow 設計

- **Job 1**: Sleep 60 秒（1 分鐘）
- **Job 2**: Sleep 300 秒（5 分鐘）
- **Concurrency group**: `test-deploy-prod`
- **Cancel in progress**: `true`

## 測試步驟

### 測試 1: 單次觸發 - 觀察 jobs 是否依序執行

1. Push tag `test-v1`:
   ```bash
   git tag test-v1
   git push origin test-v1
   ```

2. 預期行為：
   - Job 1 開始執行（1 分鐘）
   - Job 1 完成後，Job 2 開始（5 分鐘）
   - 總時間：約 6 分鐘

### 測試 2: 快速連續觸發 - 觀察舊 run 是否被取消

1. Push tag `test-v2`:
   ```bash
   git tag test-v2
   git push origin test-v2
   ```

2. 等待 30 秒（確保 Job 1 開始執行）

3. Push tag `test-v3`:
   ```bash
   git tag test-v3
   git push origin test-v3
   ```

4. 預期行為：
   - test-v2 的 run 被取消（顯示為 cancelled）
   - test-v3 的 run 立即開始

### 測試 3: 在 Job 2 執行中觸發 - 觀察長時間 job 被取消

1. Push tag `test-v4`:
   ```bash
   git tag test-v4
   git push origin test-v4
   ```

2. 等待 2 分鐘（確保 Job 1 完成，Job 2 開始執行）

3. Push tag `test-v5`:
   ```bash
   git tag test-v5
   git push origin test-v5
   ```

4. 預期行為：
   - test-v4 的 Job 2 被中斷（約執行 1 分鐘後）
   - test-v5 的 run 立即開始

## 初始化

建立 Git repository：

```bash
cd ~/Projects/test-concurrency
git init
git add .
git commit -m "Initial commit: concurrency test"
```

如果要推送到 GitHub：

```bash
# 建立 GitHub repo 後
git remote add origin <your-repo-url>
git branch -M main
git push -u origin main
```

## 觀察重點

在 GitHub Actions 頁面觀察：
1. 兩個 jobs 是否真的依序執行（不是並行）
2. 新 run 觸發時，舊 run 的狀態是否變為 "Cancelled"
3. 取消發生在哪個時間點（立即？還是等當前 job 完成？）

## 結果記錄

測試完成後記錄：
- [ ] 測試 1 結果：
- [ ] 測試 2 結果：
- [ ] 測試 3 結果：

## 結論

根據測試結果決定 trident-data 的 `cancel-in-progress` 設定。
