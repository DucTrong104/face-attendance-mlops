# 🧑‍💻 Hệ Thống Chấm Công Bằng Nhận Diện Khuôn Mặt (Face Attendance MLOps)

Dự án xây dựng hệ thống điểm danh tự động sử dụng AI nhận diện khuôn mặt, kết hợp quy trình MLOps để lưu trữ, đóng gói và triển khai. 

## 🌟 Tính Năng Nổi Bật
* **Nhận diện khuôn mặt:** Phát hiện và trích xuất đặc điểm khuôn mặt tự động qua camera.
* **Tìm kiếm tốc độ cao:** So sánh khuôn mặt nhân viên với cơ sở dữ liệu cực nhanh bằng Vector Database.
* **Giao diện thân thiện:** Web App trực quan để chụp ảnh và hiển thị kết quả chấm công.
* **Kiến trúc MLOps:** API độc lập, dễ dàng đóng gói và triển khai bằng Docker.

## 🛠️ Công Nghệ Sử Dụng
* **AI & Computer Vision:** OpenCV, DeepFace (FaceNet/ArcFace)
* **Backend API:** FastAPI, Uvicorn
* **Cơ sở dữ liệu Vector:** FAISS (Facebook AI Similarity Search)
* **Giao diện (Frontend):** Streamlit
* **MLOps & Triển khai:** Docker, GitHub Actions

## 📂 Cấu Trúc Thư Mục
```text
face-attendance-mlops/
│
├── api/                  # Code FastAPI xử lý logic và tìm kiếm FAISS
├── model/                # Code xử lý ảnh OpenCV và model DeepFace
├── ui/                   # Giao diện web bằng Streamlit
├── requirements.txt      # Danh sách các thư viện Python cần cài đặt
├── Dockerfile            # Cấu hình đóng gói hệ thống
├── .gitignore            # Các file/thư mục không đẩy lên GitHub
└── README.md             # Tài liệu hướng dẫn dự án
