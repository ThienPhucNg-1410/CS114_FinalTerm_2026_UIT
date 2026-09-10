# CS114 Final Term - NBA Game Prediction

Đồ án dự đoán kết quả trận đấu NBA bằng feature engineering theo chuỗi thời gian và ensemble model.

## 1. Nội dung project

Project gồm các phần chính:

- `data_eda/`: crawling, khám phá và trực quan hóa dữ liệu NBA.
- `FeatureEngineering/`: tạo feature phiên bản V1 và V2.
- `V3-HuongMoRong/`: feature engineering và modeling phiên bản V3.
- `Chronos-HuongMoRong/`: thử nghiệm Chronos-2 cho dự báo feature theo thời gian.
- `Modeling/`: huấn luyện và đánh giá model.
- `NBA_app/`: ứng dụng Streamlit dự đoán trận NBA.
- `Demo/`: các file demo.
- `code/`: các notebook và dữ liệu tổng hợp từ quá trình phát triển.

Thư mục `CS114-DoAnCK/` và `CS114-FinalTerm/` là các bản lưu/snapshot của quá trình làm đồ án. Các thư mục ở root là phiên bản chính được dùng để chạy.

## 2. Yêu cầu môi trường

- Python 3.10 hoặc mới hơn.
- Git.
- VS Code có Python và Jupyter extension nếu muốn chạy notebook.

## 3. Cài đặt môi trường

Mở PowerShell tại thư mục repository:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r NBA_app\requirements.txt
```

Nếu PowerShell chặn kích hoạt môi trường ảo, chạy lệnh sau một lần trong PowerShell của user:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

## 4. Chạy ứng dụng dự đoán NBA

Ứng dụng đọc model, cấu hình feature và dữ liệu bằng đường dẫn tương đối. Vì vậy cần chạy Streamlit từ thư mục `NBA_app`:

```powershell
cd NBA_app
streamlit run app.py
```

Sau khi khởi động, mở URL được Streamlit hiển thị, thường là `http://localhost:8501`.

Ứng dụng sử dụng các artifact đã huấn luyện:

- `margin_model.pkl`: dự đoán chênh lệch điểm.
- `total_points_model.pkl`: dự đoán tổng điểm.
- `xgb_clf.pkl`: XGBoost classifier.
- `lgb_clf.pkl`: LightGBM classifier.
- `calibrator.pkl`: hiệu chỉnh xác suất bằng Platt scaling.
- `meta_model.pkl`: stacking meta-model.
- `feature_config_v3.json`: danh sách 21 feature đầu vào.
- `nba_model_ready_v3.csv`: dữ liệu dùng cho suy luận và hiển thị thống kê.

Ứng dụng lấy lịch thi đấu từ API balldontlie.io và ESPN. Nếu API không khả dụng, phần lịch có thể không hiển thị nhưng chức năng dự đoán local vẫn dùng được.

## 5. Chạy notebook

Mở repository trong VS Code, chọn Python interpreter là `.venv`, sau đó mở notebook cần chạy trong các thư mục:

- `data_eda/`
- `FeatureEngineering/V1/` và `FeatureEngineering/V2/`
- `Modeling/`
- `V3-HuongMoRong/`
- `Chronos-HuongMoRong/`

Một số notebook được phát triển trên Kaggle và có thể chứa đường dẫn dạng `/kaggle/input/...`. Khi chạy local, cần thay các đường dẫn này bằng đường dẫn tương đối tới dữ liệu trong `data_eda/nba_data/`.

Chronos-2 có thể yêu cầu package và môi trường GPU riêng tùy notebook. Hãy cài dependency theo môi trường đang sử dụng trước khi chạy các cell Chronos.

## 6. Dữ liệu và feature

Pipeline sử dụng các nhóm feature như:

- rolling/EMA statistics của đội nhà và đội khách;
- hiệu suất ghi điểm, FG%, FG3%, FT%, rebound, assist, steal, block và turnover;
- win percentage, win streak, số ngày nghỉ và back-to-back;
- Elo rating và các chỉ số rate-based như eFG%, TO ratio và FT rate.

Các feature đầu vào của app phải khớp với `feature_config_v3.json`. Không đổi tên cột hoặc thứ tự feature nếu chưa cập nhật cả pipeline huấn luyện và app.

## 7. Git workflow

```powershell
git status
git add .
git commit -m "Describe your change"
git push
```

Remote hiện tại:

```text
https://github.com/ThienPhucNg-1410/CS114_FinalTerm_2026_UIT.git
```

## 8. Lưu ý về kết quả

Đây là project nghiên cứu/học tập. Kết quả dự đoán phụ thuộc vào dữ liệu lịch sử, feature được tạo tại thời điểm dự đoán và model artifact hiện có; không nên xem kết quả là đảm bảo cho kết quả thực tế của trận đấu.
