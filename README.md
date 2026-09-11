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

Các thư mục ở root là phiên bản chính của project. Những bản snapshot cũ đã được loại bỏ để tránh trùng lặp và nhầm đường dẫn.

## 2. Mục tiêu và luồng xử lý

Mục tiêu của project là dự đoán khả năng đội nhà thắng, chênh lệch điểm và tổng điểm của một trận NBA dựa trên dữ liệu lịch sử.

Luồng xử lý tổng quát:

```text
Dữ liệu trận đấu
	-> EDA và làm sạch
	-> Feature engineering theo thứ tự thời gian
	-> Elo, EMA/rolling statistics, rest days và back-to-back
	-> Chronos-2 cho một số feature nhạy với phong độ
	-> Huấn luyện và đánh giá model
	-> Lưu model/config/data ready
	-> Streamlit app suy luận trận đấu
```

Các feature thống kê phải được tính từ những trận đã diễn ra trước thời điểm dự đoán. Khi bổ sung feature mới, cần kiểm tra đặc biệt nguy cơ target leakage.

## 3. Yêu cầu môi trường

- Python 3.10 hoặc mới hơn.
- Git.
- VS Code có Python và Jupyter extension nếu muốn chạy notebook.

## 4. Cài đặt môi trường

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

## 5. Chạy ứng dụng dự đoán NBA

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

### Cách sử dụng app

1. Chọn đội nhà trong danh sách `Home team`.
2. Chọn đội khách trong danh sách `Away team`.
3. Nhấn nút dự đoán.
4. Đọc xác suất đội nhà thắng, đội được dự đoán thắng, chênh lệch điểm và tổng điểm dự kiến.

Các thống kê hiển thị trong app được lấy từ dòng dữ liệu gần nhất của mỗi đội trong `nba_model_ready_v3.csv`. Đây là dữ liệu lịch sử đã xử lý, không phải dữ liệu live theo từng phút.

### Cơ chế dự đoán

- `margin_model` dự đoán chênh lệch điểm đội nhà trừ đội khách.
- `total_model` dự đoán tổng điểm cả hai đội.
- `xgb_clf` và `lgb_clf` tạo xác suất thắng độc lập.
- `calibrator` chuyển chênh lệch điểm thành một xác suất đã hiệu chỉnh.
- `meta_model` kết hợp ba xác suất để tạo xác suất cuối cùng.
- Điểm dự kiến được tính như sau:

```text
home_points = (predicted_total + predicted_margin) / 2
away_points = (predicted_total - predicted_margin) / 2
```

## 6. Chạy notebook

Mở repository trong VS Code, chọn Python interpreter là `.venv`, sau đó mở notebook cần chạy trong các thư mục:

- `data_eda/`
- `FeatureEngineering/V1/` và `FeatureEngineering/V2/`
- `Modeling/`
- `V3-HuongMoRong/`
- `Chronos-HuongMoRong/`

Một số notebook được phát triển trên Kaggle và có thể chứa đường dẫn dạng `/kaggle/input/...`. Khi chạy local, cần thay các đường dẫn này bằng đường dẫn tương đối tới dữ liệu trong `data_eda/nba_data/`.

Chronos-2 có thể yêu cầu package và môi trường GPU riêng tùy notebook. Hãy cài dependency theo môi trường đang sử dụng trước khi chạy các cell Chronos.

### Thứ tự chạy đề xuất

Nếu cần tái tạo toàn bộ pipeline từ đầu, nên chạy theo thứ tự:

1. `data_eda/CrawlingData.ipynb` để thu thập hoặc cập nhật dữ liệu.
2. `data_eda/EDA.ipynb` để kiểm tra phân phối, missing values, outlier và tương quan.
3. Notebook trong `FeatureEngineering/V1/` hoặc `FeatureEngineering/V2/` để tạo feature cơ bản và feature mở rộng.
4. `Chronos-HuongMoRong/chronos2-v2.ipynb` nếu cần tạo dự báo feature bằng Chronos-2.
5. Notebook trong `V3-HuongMoRong/` để tạo bộ feature V3 và huấn luyện model.
6. Notebook trong `Modeling/` để so sánh model và đánh giá trên tập test.
7. Sao chép các artifact mới vào `NBA_app/` trước khi chạy app.

Không chạy lại notebook tạo model trên dữ liệu mới rồi dùng artifact cũ trong app. Model, config và dữ liệu ready phải thuộc cùng một phiên bản feature.

## 7. Dữ liệu và feature

Pipeline sử dụng các nhóm feature như:

- rolling/EMA statistics của đội nhà và đội khách;
- hiệu suất ghi điểm, FG%, FG3%, FT%, rebound, assist, steal, block và turnover;
- win percentage, win streak, số ngày nghỉ và back-to-back;
- Elo rating và các chỉ số rate-based như eFG%, TO ratio và FT rate.

Các feature đầu vào của app phải khớp với `feature_config_v3.json`. Không đổi tên cột hoặc thứ tự feature nếu chưa cập nhật cả pipeline huấn luyện và app.

### Các file dữ liệu quan trọng

| File | Vai trò |
| --- | --- |
| `data_eda/nba_data/nba_full_dataset.csv` | Dataset trận đấu gốc đã tổng hợp. |
| `data_eda/nba_data/group1_2_games_basic_stats.csv` | Các thống kê cơ bản của trận đấu. |
| `data_eda/nba_data/group4_contextual_features.csv` | Feature bối cảnh như rest days và standings. |
| `NBA_app/nba_model_ready_v3.csv` | Dữ liệu đã sẵn sàng cho app V3. |
| `NBA_app/feature_config_v3.json` | Cấu hình và danh sách feature model V3. |

### Các nguyên tắc dữ liệu

- `GAME_DATE` cần được parse thành datetime và dùng để sắp xếp trước khi tạo rolling/EMA feature.
- Mỗi `GAME_ID` hợp lệ thường có một dòng đội nhà và một dòng đội khách trong dataset gốc.
- Các trận neutral site cần được xử lý nhất quán vì pipeline dùng thông tin home/away.
- Các giá trị thiếu phải được xử lý trước khi huấn luyện và phải xử lý giống cách app kỳ vọng.
- Tập train, validation và test nên được chia theo thời gian thay vì random split.

## 8. Git workflow

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

## 9. Xử lý lỗi thường gặp

### `FileNotFoundError` khi chạy Streamlit

Kiểm tra terminal đang ở đúng thư mục `NBA_app`:

```powershell
Get-Location
Get-ChildItem margin_model.pkl, total_points_model.pkl, nba_model_ready_v3.csv, feature_config_v3.json
```

Nếu không thấy các file trên, chạy lại bằng `cd NBA_app` trước lệnh `streamlit run app.py`.

### Lỗi thiếu package

Đảm bảo virtual environment đang được kích hoạt rồi chạy:

```powershell
python -m pip install -r NBA_app\requirements.txt
```

Kiểm tra interpreter đang dùng:

```powershell
python --version
python -c "import pandas, sklearn, xgboost, lightgbm, streamlit; print('Dependencies OK')"
```

### Lỗi không khớp số lượng feature

Kiểm tra `feature_config_v3.json`, các cột được tạo trong `build_features()` của app và input shape của các model. Không tự ý thêm/bớt cột chỉ trong app; cần cập nhật cả notebook huấn luyện và artifact tương ứng.

### Lịch thi đấu không hiển thị

Lịch được gọi từ các API bên ngoài và có thể bị giới hạn mạng, timeout hoặc thay đổi response. Chức năng dự đoán từ dữ liệu local vẫn có thể hoạt động độc lập.

## 10. Phát triển và cập nhật model

Khi thay đổi pipeline:

1. Tạo một thư mục output hoặc version config mới, không ghi đè artifact đang được app dùng.
2. Ghi lại dataset, thời gian chạy, feature set và metric validation.
3. Kiểm tra dữ liệu đầu vào không có future leakage.
4. Kiểm tra app với ít nhất một cặp đội hợp lệ.
5. Chỉ thay artifact trong `NBA_app/` sau khi model và config đã được xác nhận cùng version.
6. Commit notebook, config, metric và artifact cần thiết cùng một thay đổi có mô tả rõ ràng.

## 11. Lưu ý về kết quả

Đây là project nghiên cứu/học tập. Kết quả dự đoán phụ thuộc vào dữ liệu lịch sử, feature được tạo tại thời điểm dự đoán và model artifact hiện có; không nên xem kết quả là đảm bảo cho kết quả thực tế của trận đấu.
