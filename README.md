# EE6423 — Ước lượng độ sáng thật của điểm ảnh từ video camera cố định khi có nhiễu cảm biến, nhấp nháy đèn lưới điện

## Nội dung

- `EE6423_M4.ipynb`: notebook tự chứa, lý thuyết Markdown/LaTeX và code Python theo chương; lưu sẵn output.
- `requirements.txt`: phiên bản thư viện đã dùng để kiểm thử.

Dữ liệu **hoàn toàn mô phỏng**, được sinh lại bằng code và seed cố định. Không cần tệp dữ liệu, không dùng Google Drive, GPU hay thư viện Kalman bên ngoài.

## Chạy bằng Google Colab

1. Truy cập https://colab.research.google.com/ và upload `EE6423_M4.ipynb`.
2. Dùng CPU runtime. Các thư viện NumPy, SciPy, Matplotlib, Pandas và IPython thường đã có trong Colab.
3. Chọn **Runtime → Restart session and run all** (hoặc tên tương đương của giao diện hiện tại).
4. Xem bảng và đồ thị trong Notebook. Mục Files có `figures/`, `results/` và `EE6423_outputs.zip` do code sinh; có thể tải ZIP kết quả nếu muốn sử dụng hình và số liệu bên ngoài Notebook.
5. Lưu Notebook kèm output. Không cần upload các thư mục output lên GitHub để chạy lại.

Nếu chạy local: dùng Python 3, môi trường Jupyter và các thư viện trong `requirements.txt`. Cài bằng `python -m pip install -r requirements.txt`, sau đó mở Notebook bằng JupyterLab/Jupyter Notebook và Restart & Run All. Môi trường đã kiểm thử: Python 3.13.5, NumPy 2.3.5, SciPy 1.17.0, Pandas 2.2.3, Matplotlib 3.10.8. Phiên bản runtime khác có thể cho sai khác số nhỏ; kiểm tra kết quả và in phiên bản ở Chương 0.

## Cấu trúc và cách tái tạo hình

| Chương | Chức năng | Hình sinh từ code |
|---|---|---|
| 0–1 | Cấu hình, đề bài | — |
| 2 | Ground truth, nguồn nhiễu, mô hình đo | `02_ground_truth.png`, `02_aliasing.png`, `02_measurements.png` |
| 3 | WSS, mean/variance, ACF, Welch | `03_acf.png`, `03_welch.png` |
| 4 | Hiệu chuẩn tối, PTC, LS Flicker, Allan, AR(1), độ bất định | `04_calibration.png` |
| 5 | Raw, MA, KF3/KF5, validation, kiểm thử nhân quả | — |
| 6 | Monte Carlo, bước sáng, innovation, độ nhạy | `06_rmse.png`, `06_tracking.png`, `06_innovation.png`, `06_sensitivity.png` |
| 7 | Kết luận, ghi cấu hình và đóng gói output | — |

CSV trong `results/` gồm `shot_check`, `simulation_check`, `statistics`, `welch`, `calibration`, `calibration_trials`, `validation`, `monte_carlo`, `monte_carlo_trials`, `paired`, `step_metrics`, `innovation`, `innovation_trials`, `sensitivity`, `clipping`; `configuration.json` ghi tham số, seed, phiên bản và thiết kế đã chọn. Cell cuối đóng gói output riêng, không đưa ZIP lồng vào ZIP code.

## Thiết kế thực nghiệm

- 1.800 mẫu ở 30 FPS cho mỗi test; C0, S0, S1, S2; 100 realization/kịch bản. Seed minh họa 42, hiệu chuẩn thống kê 50–79, hiệu chuẩn chính 11, MC hiệu chuẩn 500–529, validation 100–109, test 200–299, sensitivity 600–619.
- Hai KF tuyến tính tự lập trình, chung tham số nền và nhiễu cảm biến, khác mô hình Flicker. MA nhân quả. Bộ lọc chỉ nhận số đo và cấu hình học từ hiệu chuẩn/validation; ground truth chỉ dùng để sinh dữ liệu, kiểm chứng và đánh giá.
- Nhiễu photon Poisson trực tiếp; lượng tử hóa đúng một lần. Camera Basler chỉ là thiết bị tham chiếu, OB và các biến động ánh sáng là giả thiết mô phỏng; không có dữ liệu camera thực.
- RMSE so sánh sau 150 mẫu burn-in; S1 đánh giá thêm 90 mẫu từ bước tại n=900. CI 95% là CI của trung bình qua các realization, không phải CI từng bản ghi.
- Phương sai trạng thái nền q_x là tham số thiết kế chọn bằng validation, không phải phép đo phương sai Random Walk thực. Nghiệm validation ở biên lưới đã khảo sát. Innovation còn tương quan; không tuyên bố Kalman tối ưu tuyệt đối hay đã đạt whiteness.
