# News Aggregation AI Agent

Một tác nhân (agent) tổng hợp tin tức dùng AI để thu thập, làm sạch, phân loại, tóm tắt và cung cấp nội dung tin tức từ nhiều nguồn. Dự án hướng tới việc xây dựng một pipeline tự động để thu thập tin, xử lý bằng LLM, lưu trữ kết quả và cung cấp API để truy vấn.

## Tính năng chính
- Thu thập tin tức từ RSS/Atom feed, trang web và API.
- Lọc và loại bỏ tin trùng lặp (deduplication).
- Tự động tóm tắt bài báo bằng AI (short/medium/long summaries).
- Phân lớp (categorization) và gắn nhãn (tagging) bài viết.
- Lưu trữ metadata và nội dung (PostgreSQL).
- Cung cấp REST API (FastAPI) để truy vấn tin tóm tắt, filter theo tag, nguồn, thời gian.
- Hỗ trợ chạy theo lịch (cron/scheduler) để thu thập định kỳ.
- Cấu hình nguồn (feeds), bộ lọc và tham số tóm tắt.

## Kiến trúc tổng quan
1. Fetchers — thu thập dữ liệu từ feed / trang web / API.
2. Extractors — trích xuất nội dung, metadata (tiêu đề, tác giả, ngày).
3. Cleaners — loại bỏ HTML thừa, script, tracking.
4. Processors — dedupe, phân loại, tóm tắt bằng LLM.
5. Storage — lưu bài viết và kết quả xử lý (DB).
6. API — expose endpoint để truy xuất kết quả.
7. Scheduler — điều phối công việc thu thập định kỳ.

## Yêu cầu
- Python 3.9+
- Hệ quản trị cơ sở dữ liệu: SQLite (mặc định), PostgreSQL để production.
- API key cho model LLM (nếu sử dụng OpenAI/Anthropic/..., đặt trong biến môi trường).
- Cài đặt `requirements.txt`.

## Cài đặt nhanh
1. Clone repository:
   ```bash
   git clone https://github.com/anphthanh004/news-aggregation-ai-agent.git
   cd news-aggregation-ai-agent
   ```

2. Tạo virtual environment và cài dependencies:
   ```bash
   python -m venv .venv
   source .venv/bin/activate   # macOS / Linux
   .venv\Scripts\activate      # Windows (PowerShell)
   pip install -r requirements.txt
   ```

## Cấu hình
Tạo tệp `.env` (hoặc sử dụng biến môi trường) với các biến mẫu:
```env
# Định danh LLM / provider
OPENAI_API_KEY=sk-...

# Database
DATABASE_URL=sqlite:///./data/news.db
# hoặc ví dụ PostgreSQL:
# DATABASE_URL=postgresql://user:password@localhost:5432/newsdb

# Scheduler
FETCH_CRON="*/15 * * * *"  # mỗi 15 phút (tùy triển khai)

# Các cấu hình khác
DEFAULT_SUMMARY_LENGTH=short  # short | medium | long
```

Cấu hình nguồn tin (ví dụ file `feeds.yaml`):
```yaml
feeds:
  - name: "BBC News"
    type: "rss"
    url: "http://feeds.bbci.co.uk/news/rss.xml"
    tags: ["world", "uk"]
  - name: "Example API"
    type: "api"
    url: "https://example.com/news"
    api_key: "${EXAMPLE_API_KEY}"
```

## Chạy agent
Ví dụ khởi chạy agent thu thập một lần:
```bash
# Chạy script thu thập (tùy cấu trúc repo)
python -m news_agent.run --config feeds.yaml
```

Chạy server API (nếu dùng FastAPI + Uvicorn):
```bash
uvicorn news_agent.api:app --host 0.0.0.0 --port 8000 --reload
```

Một ví dụ endpoint:
- GET /articles — danh sách bài báo (filter theo ngày, tag, nguồn)
- GET /articles/{id} — chi tiết + tóm tắt
- GET /summaries?length=short&type=bullet — tóm tắt theo yêu cầu

## Sử dụng LLM để tóm tắt
Nếu dự án tích hợp với OpenAI hoặc provider khác, hãy đặt khóa API vào biến môi trường tương ứng (ví dụ `OPENAI_API_KEY`) và cấu hình model trong file cấu hình. Ví dụ gọi tóm tắt:
```python
from llm_client import summarize

summary = summarize(text, length="short", model="gpt-4o-mini")
```

Ghi chú bảo mật: không commit khóa API vào git, luôn dùng `.env` / secret manager.

## Lưu trữ & migration
- Mặc định có thể dùng SQLite cho phát triển.
- Đối với production dùng PostgreSQL.

## Logging và giám sát
- Bật logging (INFO/DEBUG) trong cấu hình.
- Tích hợp Sentry / Prometheus nếu cần giám sát production.

## Testing
- Viết unit tests cho fetcher, extractor, processor.
- Chạy test:
  ```bash
  pytest
  ```
