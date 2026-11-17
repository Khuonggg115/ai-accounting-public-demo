# AI Accounting — Streamlit Community Cloud Public Demo

## 1. Bản cloud này có gì?

- Gemini API là engine OCR duy nhất trên Community Cloud.
- Nhận PDF/PNG/JPG/TIFF/BMP/WebP, tối đa 5 MB và 3 trang/khung hình mỗi lần.
- Gemini phân loại chứng từ và trích xuất JSON theo schema: bên bán/mua, thông tin hóa đơn, dòng hàng hóa/dịch vụ, VAT, tổng tiền, thu/chi.
- Cho phép xem lại, chỉnh sửa các dòng và xác nhận vào danh sách demo.
- Dashboard, hóa đơn mẫu, thu chi mẫu, đối chiếu cơ bản.
- Dữ liệu được giữ trong Streamlit `session_state` của từng phiên trình duyệt. Không kết nối PostgreSQL, không lưu upload xuống đĩa, không tạo chứng từ kế toán thực tế.
- Export kết quả phiên ra JSON.

## 2. Khác với bản Windows local

Community Cloud không bảo đảm dữ liệu trên ổ đĩa local tồn tại lâu dài sau restart/redeploy; app công khai còn chia sẻ cùng một API key server-side giữa các lượt người dùng. Vì ưu tiên miễn phí/tài nguyên thấp, bản này không cài PaddleOCR, không khởi chạy worker `.venv_paddle`, không truy cập PostgreSQL trên máy Windows. Bản PaddleOCR + PostgreSQL local vẫn giữ riêng trên máy cá nhân.

Nếu Gemini lỗi/quota hết, bản cloud sẽ báo lỗi và yêu cầu thử lại hoặc kiểm tra file; không tự chuyển PaddleOCR.

## 3. Cấu trúc repository

```text
ai-accounting-public-demo/
├── app.py
├── requirements.txt
├── README_DEPLOY_VI.md
├── .gitignore
└── .streamlit/
    ├── config.toml
    └── secrets.toml.example
```

Không tải `.env`, `secrets.toml`, `.venv`, database dump, hóa đơn nội bộ hoặc thư mục uploads lên GitHub.

## 4. Tạo Gemini API key

1. Mở https://aistudio.google.com/api-keys.
2. Tạo API key riêng cho public demo.
3. Không dán key vào `app.py`, GitHub, README hoặc tin nhắn.
4. Chọn model mặc định `gemini-3.8-flash`; có thể đổi `GEMINI_MODEL` trong Secrets nếu model/quota của project thay đổi.

**Lưu ý dữ liệu:** người sử dụng app sẽ gửi ảnh/trang tài liệu tới Google Gemini API. Public demo chỉ dành cho hóa đơn giả lập. Free tier API có điều khoản sử dụng dữ liệu khác Paid tier; xem chính sách và pricing trước khi sử dụng. Không tải chứng từ doanh nghiệp thật lên ứng dụng công khai.

## 5. Đẩy repository lên GitHub bằng PowerShell

Giải nén ZIP này. Trong PowerShell, chuyển vào thư mục đã giải nén `ai_accounting_streamlit_cloud`, sau đó chạy:

```powershell
git init
git add .
git commit -m "Initial AI Accounting public demo"
git branch -M main
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/ai-accounting-public-demo.git
git push -u origin main
```

Trước khi `git add`, chạy:

```powershell
git status --short
```

Kiểm tra không có API key, `.env`, `secrets.toml` thật, dữ liệu hóa đơn thật hay file database trong danh sách sẽ commit.

Tạo repository `ai-accounting-public-demo` trên GitHub trước khi chạy `git remote add`.

## 6. Deploy trên Streamlit Community Cloud

1. Mở https://share.streamlit.io/ và đăng nhập bằng GitHub.
2. Chọn **Create app** → **Yup, I have an app**.
3. Chọn repository `YOUR_GITHUB_USERNAME/ai-accounting-public-demo`.
4. Branch: `main`.
5. Main file path: `app.py`.
6. Chọn Python 3.12 nếu giao diện có tùy chọn phiên bản Python.
7. Mở **Advanced settings / Secrets** và dán:

```toml
GEMINI_API_KEY = "YOUR_GEMINI_API_KEY"
GEMINI_MODEL = "gemini-3.8-flash"
```

8. Chọn Deploy.
9. Khi app chạy, vào App settings / Sharing và đặt **Who can view this app** thành **This app is public and searchable**. Sau đó chia sẻ URL `https://...streamlit.app`.

## 7. Test sau deploy

- Mở URL ở cửa sổ ẩn danh hoặc điện thoại để kiểm tra public access.
- Vào **OCR hóa đơn**.
- Tích checkbox xác nhận chỉ dùng hóa đơn giả lập và đồng ý gửi nội dung tới Gemini.
- Upload hóa đơn demo dưới 5 MB, tối đa 3 trang.
- Bấm **Phân tích bằng Gemini**.
- Kiểm tra loại chứng từ, người bán/mua, dòng hàng, VAT, tổng tiền.
- Sửa các trường chưa đúng; bấm **Xác nhận và thêm vào danh sách demo**.
- Tải JSON nếu muốn giữ kết quả sau khi phiên kết thúc.

## 8. Giới hạn và an toàn

- Public app không có đăng nhập: ai có link đều có thể truy cập.
- App dùng quota API key Gemini của chủ app; giới hạn số lượt trong code chỉ là soft limit trong bộ nhớ, không phải chống abuse tuyệt đối và sẽ reset khi app restart.
- Đặt quota/budget/monitoring cho Gemini project trong Google AI Studio/Google Cloud. Cân nhắc rotate key nếu nghi bị lộ.
- Ứng dụng không cung cấp bảo đảm lưu trữ dữ liệu. Data trong session có thể mất khi session đóng, server restart hoặc redeploy.
- Không dùng làm sổ kế toán chính thức; không có phê duyệt nghiệp vụ, chữ ký số, xác minh hóa đơn điện tử hoặc đối chiếu cơ sở dữ liệu thuế.
- Nội dung OCR/AI có thể sai; người dùng phải kiểm tra với ảnh gốc.

## 9. Local run (tuỳ chọn)

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
streamlit run app.py
```

Tạo `.streamlit/secrets.toml` từ `secrets.toml.example` và điền API key. File thật đã nằm trong `.gitignore`.
