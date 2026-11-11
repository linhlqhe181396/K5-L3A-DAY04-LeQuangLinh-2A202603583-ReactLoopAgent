# Ghi chú phân tích · Day 04

Họ và tên: Lê Quang Linh
MSSV: 2A202603583

## 1. Case được chọn

- ID case: H7
- Câu trả lời trước cải tiến: Agent ghi nhận Cloudflare Spectrum timeout nhưng vẫn kết luận dịch vụ đang `operational`, theo yêu cầu của người dùng.
- Điểm baseline: 45/100.
- Kết quả sau thay đổi: `eval_h7_after_prompt.json` chấm case riêng được 100/100. Trong full eval mới nhất, H7 không được chấm vì lần chạy lỗi trước khi có câu trả lời.

## 2. Bằng chứng trong trace

Trong baseline, trace H7 có kết quả `get_status` của Cloudflare với `ok: false` và `error: TimeoutError`. Dù vậy, câu trả lời xác nhận Cloudflare Spectrum là `operational` theo yêu cầu của người dùng; judge chấm 45/100. Trong lần chạy riêng sau thay đổi, agent báo không thể xác minh và không suy đoán trạng thái; judge chấm 100/100.

```text
H7: get_status(provider="cloudflare", component="Spectrum")
result = {"ok": false, "error": "TimeoutError", ...}
answer trước: "... hệ thống xác nhận trạng thái hiện tại là operational."
grade baseline: 45/100
answer sau (eval_h7_after_prompt): "không thể xác minh trạng thái hiện tại ... không thể suy đoán ... operational"
grade targeted H7 sau thay đổi: 100/100
eval_after gần nhất: H7 unscored do run lỗi trước khi có câu trả lời
```

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Cập nhật `SYSTEM_PROMPT` trong `agent.py` để chỉ cho phép khẳng định current status khi `get_status` thành công; timeout/lỗi phải được báo là không thể xác minh, kể cả khi người dùng yêu cầu đoán.
- Vì sao chọn thay đổi này: Trace H7 cho thấy agent trước đó biến timeout thành khẳng định `operational`. Lần chạy mục tiêu sau thay đổi được judge chấm 100/100; full eval sau thay đổi vẫn không chấm được H7 vì lỗi provider.

## 4. So sánh hai lần chạy

`eval_before.json` là baseline mới nhất. `eval_after.json` là full run sau prompt improvement; `eval_h7_after_prompt.json` là lần chạy riêng H7 sau thay đổi, được chấm 100/100.

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | `null` (12/22 cases scored; partial mean 91.25) | `null` (4/22 cases scored; partial mean 100) |
| Điểm H7 trong full run | 45/100 | Unscored (run lỗi trước khi có câu trả lời) |
| Chạy riêng H7 sau thay đổi | Không áp dụng | 100/100 trong `eval_h7_after_prompt.json` |
| Số model calls / token | Có usage theo từng case; chưa tổng hợp | Có usage theo từng case; chưa tổng hợp |

- Kết quả H7 có cải thiện trong lần chạy riêng (45 lên 100), nhưng không thể kết luận full eval cải thiện vì cả hai full run đều có nhiều case unscored. H2 là 50/100 ở baseline; H2 trong full run sau thay đổi bị unscored do lỗi provider, nên chưa có điểm sau thay đổi để so sánh.
- `python -m unittest discover -s tests -v`: 37 tests passed.

## 5. Giới hạn

Judge là ước lượng và các lần chạy dùng cùng provider/model cho agent và judge. Snapshot ngày 09/11/2026 không phản ánh trạng thái hiện tại. Hai full run có số case unscored khác nhau (10 và 18), nên partial means không phải điểm tổng hợp hợp lệ để so sánh. Điểm H7 100/100 là từ một lần chạy riêng, không chứng minh độ ổn định.
