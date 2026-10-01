# DDM501 Individual Assignment 2 - ML Pipeline and MLOps

Pipeline Airflow + MLflow cho bài toán phân loại WDBC, phát triển tiếp từ kiến trúc hệ thống trong Individual Assignment 1.

Pipeline gồm bảy bước:

```text
ingest -> validate -> split -> scale -> train -> register -> report
```

Airflow điều phối workflow và xử lý retry/backfill. MLflow lưu parameters, metrics, model artifacts và các model version. Dữ liệu được chia theo hash của `sample_id`; scaler chỉ được fit trên tập train để tránh data leakage.

## Nội dung đáp ứng Assignment 2

- Sơ đồ pipeline, stage specifications và quality gates
- Ma trận 10 experiment configurations
- Phân tích recall, specificity, precision, F1, ROC-AUC và latency
- MLflow experiment tracking và model registry
- Airflow DAG, scheduling, failure handling và monitoring
- Configuration, type hints, documentation và code-quality strategy
- Git/data/model versioning, random seeds và Docker reproducibility

## Cấu trúc chính

| Đường dẫn | Vai trò |
|---|---|
| `dags/wdbc_pipeline.py` | Airflow DAG với 7 task |
| `scripts/fetch_and_predict.py` | Tải model từ MLflow Registry và dự đoán |
| `scripts/corrupt_extract.py` | Tạo/khôi phục dữ liệu lỗi để kiểm thử validation |
| `data/raw/wdbc.csv` | Dữ liệu đầu vào |
| `docs/screenshots/` | Minh chứng Airflow và MLflow |
| `report/` | PDF Assignment 2 hoàn chỉnh |
| `docker-compose.yml` | Airflow và MLflow services |

## Chạy project

Yêu cầu Docker Desktop đang chạy. Từ thư mục repository:

```powershell
docker compose up -d --build
docker compose ps
docker compose exec airflow cat /opt/airflow/standalone_admin_password.txt
```

Mở:

- Airflow: <http://127.0.0.1:18080>
- MLflow: <http://127.0.0.1:15030>

Chạy DAG và tải model đã đăng ký:

```powershell
docker compose exec airflow airflow dags test wdbc_pipeline 2026-08-25
docker compose exec airflow python scripts/fetch_and_predict.py --ds 2026-08-25
```

Kiểm thử quality gate:

```powershell
python scripts/corrupt_extract.py
docker compose exec airflow airflow dags test wdbc_pipeline 2026-08-26
python scripts/corrupt_extract.py --repair
```

## Kết quả đã xác minh

Run mẫu ngày `2026-08-25` có 564 dòng hợp lệ, 441 dòng train và 123 dòng test. Logistic-regression implementation trong DAG đạt `accuracy=0.9512` và `roc_auc=0.9956`.

Báo cáo chạy thêm 10 cấu hình. SVM RBF có kết quả tốt nhất tại threshold mặc định trong nhóm thử nghiệm (`recall=0.941`, `specificity=0.972`, `roc_auc=0.996`) nhưng chưa đạt recall floor đề xuất là `0.95`; vì vậy không có model nào được đề xuất promotion tự động.

## Tài liệu nộp bài

PDF hoàn chỉnh nằm tại [`report/DDM501_Assignment2_STUDENTID_NAME.pdf`](report/DDM501_Assignment2_STUDENTID_NAME.pdf).

Trước khi nộp, thay tên/MSSV và đổi tên file theo định dạng:

```text
DDM501_Assignment2_[StudentID]_[Name].pdf
```

## Phạm vi sử dụng

Đây là project học tập về MLOps. Dataset và kết quả thử nghiệm không đủ để triển khai trong môi trường chẩn đoán lâm sàng.

