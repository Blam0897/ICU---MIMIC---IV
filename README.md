# [Tên đề tài] — Dự đoán suy tạng ICU đa nhiệm vụ (SOFA-2, MIMIC-IV & eICU-CRD)

Repo chung của nhóm 5 người: **Blam · Kong · Bin · Hari · Johnny**

Dự án xây dựng pipeline dự đoán suy giảm chức năng 6 hệ cơ quan (theo SOFA-2) cho bệnh nhân ICU, huấn luyện trên MIMIC-IV và đánh giá ngoại kiểm trên eICU-CRD (3 vùng: South/Midwest/West).

Dựa trên phương pháp của: Zeng Z, Liu Y, Yao S, et al. *Inter-organ correlation based multi-task deep learning model for dynamically predicting functional deterioration in multiple organ systems of ICU patients.* BioData Mining. 2025;18:31.

---

## Cấu trúc thư mục

```
.
├── data/           # KHÔNG commit dữ liệu bệnh nhân thật lên đây (xem mục "Dữ liệu" bên dưới)
│   ├── raw/        # Dữ liệu thô tải từ PhysioNet (gitignore)
│   ├── processed/  # Dữ liệu đã qua tiền xử lý (gitignore)
│   └── config/     # File mô tả schema, data dictionary (được phép commit — không chứa dữ liệu bệnh nhân)
│
├── src/            # Code chính của pipeline
│   ├── cohort/         # Trích xuất & lọc cohort (H1, H3, B4)
│   ├── harmonize/      # Ánh xạ biến MIMIC ↔ eICU, physiologic bounds (H2)
│   ├── sofa/           # Tính điểm SOFA-2 / SOFA-1, 6 organ subscore (B3, K5)
│   ├── coverage/        # Kiểm tra coverage dữ liệu từng hệ cơ quan (J1, J2)
│   ├── windowing/       # Observation window, chống rò rỉ dữ liệu (BI1)
│   ├── labeling/        # Quy tắc gán nhãn động, masked loss (BI2, BI3, BI4)
│   ├── mts/             # Dựng bảng multivariate time series theo giờ (H4)
│   ├── sample_gen/      # Sinh sample input/output cho model (BI5)
│   └── utils/            # Hàm dùng chung
│
├── config/          # File cấu hình trung tâm (B5)
│   └── config.yaml   # Bảng SOFA-2, horizon đã chọn, ngưỡng đã duyệt — DÙNG CHUNG CHO CẢ NHÓM
│
├── notebooks/       # Notebook thử nghiệm, phân tích, vẽ hình — mỗi người 1 thư mục con
│   ├── blam/
│   ├── kong/
│   ├── bin/
│   ├── hari/
│   └── johnny/
│
├── docs/            # Tài liệu: bảng công thức SOFA-2, outcome definition, báo cáo, checklist
│   ├── sofa2_formula.md         # B3 — bảng công thức 6 cơ quan
│   ├── outcome_definition.md    # BI3 — quy tắc gán nhãn
│   ├── exclusion_criteria.md    # B1 — tiêu chí loại trừ
│   ├── data_dictionary.md       # H2 — bảng ánh xạ biến
│   └── open_questions.md        # Câu hỏi mở cần thầy xác nhận
│
├── .gitignore
├── requirements.txt  # hoặc environment.yml nếu dùng conda
└── README.md          # File này
```

## Quy ước nhánh (branching)

- `main` — nhánh ổn định, chỉ merge qua Pull Request đã được ít nhất 1 người khác review.
- Mỗi người làm việc trên nhánh riêng, đặt tên theo mẫu:
  ```
  <ten-nguoi>/<ma-task>-<mo-ta-ngan>
  ```
  Ví dụ:
  - `blam/b1-38-bien-tien-xu-ly`
  - `kong/k5-sofa2-subscore`
  - `bin/bi1-windowing`
  - `hari/h1-cohort-extraction`
  - `johnny/j1-coverage-respiratory-cv`

- Commit message ngắn gọn, có mã task ở đầu:
  ```
  [B1] Thêm bảng 38 biến và tiêu chí loại trừ
  [K5] Fix lỗi tính subscore CNS khi thiếu GCS
  ```

- Trước khi mở Pull Request: `git pull origin main` và giải quyết conflict trước, không đẩy conflict cho người review.

## Dữ liệu — KHÔNG commit dữ liệu bệnh nhân

- **Tuyệt đối không đẩy dữ liệu MIMIC-IV/eICU-CRD thật (kể cả đã xử lý) lên GitHub** — đây là dữ liệu y tế có credential theo PhysioNet DUA (Data Use Agreement).
- Thư mục `data/raw/` và `data/processed/` đã được thêm vào `.gitignore`.
- Mỗi người tự tải dữ liệu về máy/server riêng theo hướng dẫn PhysioNet, đặt đúng cấu trúc thư mục `data/raw/` để code chạy nhất quán.

## Cách bắt đầu

```bash
git clone <repo-url>
cd <ten-repo>
git checkout -b <ten-nguoi>/<ma-task>-<mo-ta-ngan>

# cài môi trường
pip install -r requirements.txt
# hoặc: conda env create -f environment.yml

# đặt dữ liệu vào data/raw/ theo hướng dẫn trong docs/
```

## Liên hệ / phân công

Xem chi tiết phân việc từng người theo ngày trong `docs/` (kế hoạch sprint 4 ngày) hoặc hỏi trực tiếp trong nhóm chat.

| Người | Phụ trách chính |
|---|---|
| Blam | 38 biến, repo, công thức SOFA-2, flowchart cohort, config trung tâm |
| Kong | Tra cứu y tế (PaO2/FiO2, delirium), đối chiếu ngưỡng, code subscore SOFA |
| Bin | Windowing, missing label, outcome definition, horizon, sample generation |
| Hari | Cohort extraction, ánh xạ biến, MTS, prevalence |
| Johnny | Coverage dữ liệu 6 organ, QA subscore, tổng hợp sample + schema |
