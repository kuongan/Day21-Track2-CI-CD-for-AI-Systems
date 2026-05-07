

# Báo cáo – Huấn luyện & tối ưu mô hình

Tên: Nguyễn Trần Khương An 
MSSV: 2A202600222

## 1. Bộ siêu tham số đã chọn

Sau quá trình thử nghiệm và **grid search toàn bộ không gian tham số khả thi**, bộ siêu tham số tốt nhất cho mô hình `RandomForestClassifier` được lựa chọn như sau:

```yaml
n_estimators: 200
max_depth: 20
min_samples_split: 5
max_features: "sqrt"
min_samples_leaf: 1
```

### Lý do lựa chọn:

* **n_estimators = 200**: tăng số lượng cây giúp mô hình ổn định hơn, giảm phương sai.
* **max_depth = 20**: cân bằng giữa khả năng học và tránh overfitting.
* **min_samples_split = 5**: giúp cây không tách quá chi tiết, giảm overfitting.
* **max_features = "sqrt"**: chiến lược mặc định tốt cho bài toán classification, giúp đa dạng hóa cây.
* **min_samples_leaf = 1**: giữ độ linh hoạt cao cho cây quyết định.

Bộ tham số này cho kết quả tốt nhất trong quá trình grid search và được chọn làm cấu hình chính thức cho pipeline.

---

## 2. Khó khăn gặp phải và cách giải quyết

### 1. Chuyển đổi cloud storage từ GCP sang AWS

**Khó khăn:**

* Ban đầu pipeline được thiết kế theo Google Cloud Platform (GCS).
* Khi chuyển sang AWS S3, các cấu hình DVC và authentication không tương thích.
* Một số lỗi phát sinh do thiếu `dvc-s3` plugin.

**Cách giải quyết:**

* Cập nhật `requirements.txt`:

  ```bash
  dvc[s3]
  boto3
  ```
* Cấu hình lại remote:

  ```bash
  dvc remote add -d myremote s3://<bucket>/dvc
  ```
* Thiết lập AWS credentials thông qua GitHub Secrets.

---

### 2. Không đạt ngưỡng quality gate (accuracy < 0.70)

**Khó khăn:**

* Sau khi thực hiện **grid search toàn bộ tham số**, accuracy cao nhất chỉ đạt ~0.686.
* Không đủ điều kiện deploy theo threshold ban đầu (0.70).

**Cách giải quyết:**

* Tạm thời điều chỉnh quality gate xuống:

  ```python
  EVAL_THRESHOLD = 0.65
  ```
* Tiếp tục tối ưu tham số và cải thiện pipeline.

---

### 3. Cải thiện dataset (Version 2)

**Khó khăn:**

* Dataset ban đầu (Version 1) chưa đủ đa dạng → mô hình bị giới hạn performance.

**Cách giải quyết:**

* Bổ sung thêm dữ liệu (train_phase2.csv).
* Sử dụng DVC để versioning dữ liệu.
* Sau khi retrain:

  * Accuracy tăng lên **0.74**
  * Model ổn định hơn

👉 Do đó, quality gate được nâng lại về:

```python
EVAL_THRESHOLD = 0.70
```

---

## 3. Kết luận

* Grid search giúp xác định bộ tham số tối ưu cho RandomForest.
* Việc mở rộng dataset có ảnh hưởng trực tiếp đến performance mô hình.
* Pipeline CI/CD hoạt động ổn định, tự động hóa từ train → eval → deploy.
* Hệ thống đã chuyển thành công từ GCP sang AWS mà không phá vỡ workflow MLOps.
