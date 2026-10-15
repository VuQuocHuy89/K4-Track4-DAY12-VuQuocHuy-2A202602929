# REPORT — Lab Day 12

> Điền **bằng lời và số liệu của chính bạn** (lấy từ màn hình và từ `evidence/*.json`). Mỗi câu 4–8 dòng là đủ. Số không có trong bằng chứng của bạn sẽ bị trừ điểm. Báo cáo công khai trên GitHub; giảng viên đối chiếu với `evidence/` và với bài của các bạn khác.

| | |
|---|---|
| Họ và tên | `Vũ Quốc Huy` |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | `2A202602929` |
| GitHub | `VuQuocHuy89` |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | `7466 / 5873` |
| Số seed huấn luyện bạn dùng (`--seeds`) | `3 (5873, 5874, 5875)` |

## Kết quả cuối

Điền từ `evidence/closedloop.json`, `openloop.json`, `gate.json` (cho mỗi mô hình học, lấy **dải qua các seed**).

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---|---|---|
| constant-velocity | 1.255 | 20.2% | BLOCK |
| IDM | 1.531 | 0.0% | baseline |
| EgoMLP | 0.757–0.761 | 17.0–27.0% | BLOCK |
| PercMLP | 0.620–0.621 | 0.7–1.4% | BLOCK |
| PercMLP+ (sau khi sửa) | 0.779–0.817 | 0.2–0.3% | RELEASE |

## Câu 1 · Dữ liệu
Bước 0 bạn phát hiện những vấn đề gì ở dữ liệu? Vì sao không dùng cột `Acc_LV`, và bạn thay bằng gì? Bao nhiêu quỹ đạo bị loại vì trùng, và vì sao chia theo quỹ đạo?

CSV có 992.679 hàng: 9.993 quỹ đạo thô, loại 383 quỹ đạo trùng còn 9.610; mỗi quỹ đạo dài 70–109 bước ở 10 Hz. Tương quan của `Acc_LV` với đạo hàm vận tốc chỉ 0,045 nên cột này không đáng tin; gia tốc xe dẫn được tính lại từ `Speed_LV` đã làm trơn. Các bản trùng có thể nằm ở ID xa nhau, vì vậy chia ngẫu nhiên theo hàng có thể đưa cùng quỹ đạo vào cả train và test, gây rò rỉ. Chia theo quỹ đạo giữ các cửa sổ của một chuyến đi cùng một phía.

## Câu 2 · Phát lại
Bản refactor float32 khác bit ở bao nhiêu đoạn, vượt dung sai ở bao nhiêu? Thay đổi AEB làm lệch bao nhiêu đoạn, và vì sao lệch ở đúng những khung đó (mở một đoạn ra xem)? Phát theo thời gian thật làm lộ ra điều gì mà phát nhanh nhất che đi?

Phát lại cùng mã cho kết quả bit-identical ở cả 600 đoạn. Refactor float32 khác bit ở 600/600 nhưng không đoạn nào vượt dung sai; sai khác cực đại là 9,7×10⁻⁵, nhỏ hơn mức 10⁻³. AEB làm 8 đoạn vượt dung sai (9 đoạn không bit-identical). Tôi mở đoạn 69: tại frame 73, t=7,32 s, gap=3,757 m, tốc độ đóng=2,717 m/s nên TTC=1,383 s < 1,5 s; lệnh đổi từ −5,58 sang −6,0 m/s². Những frame này khác vì điều kiện TTC kích hoạt phanh khẩn cấp. Chạy realtime có 582 frame trễ và 577 frame bị bỏ trong 59.806 frame, làm đổi 383 đoạn; chế độ fastest che mất việc này vì không áp deadline.

## Câu 3 · Kịch bản
Lưới của bạn (tốc độ cruise, giới hạn khoảng cách, số ô): bao nhiêu % ô **không ai tránh được**? Độ trễ phản ứng 0 → 0,6 s làm vùng đâm của IDM tăng bao nhiêu? Log có phủ vùng khó không? (dẫn giới hạn phanh mạnh nhất của người lái trong log, và phần lưới đòi hỏi hơn thế)

Với cruise 10 m/s, lưới gồm 60 gap (2,76–59,31 m) × 45 tốc độ đóng (0–10 m/s), tổng 2.700 ô. Ở trễ phản ứng 0,6 s, 5,15% ô không thể tránh va chạm ngay cả khi phanh hết mức; khi trễ bằng 0 là 0,96%. Va chạm IDM tăng từ 0,96% lên 5,15%, tức +4,19 điểm phần trăm và trùng vùng bất khả thi theo vật lý. Log có 614 sự kiện phanh, nhưng 5,22% lưới đòi hỏi phanh mạnh hơn giới hạn p99,9 của người lái trong log là 3,62 m/s²; vì vậy dữ liệu không phủ hết vùng khó.

## Câu 4 · Vòng hở và vòng kín
Dán bảng L2 và tỷ lệ va chạm của bốn bộ điều khiển (kèm hạng ở mỗi cách chấm). Cặp nào đổi thứ tự? Vì sao IDM có L2 như vậy mà vẫn lái tốt? Vì sao EgoMLP có L2 tốt hơn constant-velocity mà vẫn đâm?

| Bộ điều khiển | L2 vòng hở (hạng) | Va chạm vòng kín (hạng) |
|---|---:|---:|
| constant-velocity | 1,255 (3) | 20,2% (3) |
| IDM | 1,531 (4) | 0,0% (1) |
| EgoMLP | 0,757–0,761 (2) | 17,0–27,0% (4) |
| PercMLP | 0,620–0,621 (1) | 0,7–1,4% (2) |

Có bốn cặp đổi thứ tự giữa hai cách chấm, gồm IDM với constant-velocity, EgoMLP và PercMLP. L2 đo độ khớp dự báo với quỹ đạo con người, còn IDM chủ động giữ khoảng cách và phanh theo nguy cơ nên có L2 cao nhưng ít va chạm. EgoMLP không thấy gap hay xe dẫn; lỗi nhỏ tích lũy khi tự điều khiển, dẫn tới va chạm dù L2 tốt hơn constant-velocity.

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

Tỷ lệ va chạm của EgoMLP qua ba seed là 27,0%, 25,0% và 17,0%, dao động gần 10 điểm phần trăm. Ở seed xấu nhất, khoảng Wilson 95% là 25,4–28,6% trên 2.883 quỹ đạo; riêng lát cắt có xe dẫn phanh chỉ có 187 quỹ đạo nên khoảng tin cậy còn rộng hơn. Chạy một seed có thể chỉ báo 17,0% hoặc 27,0%, tùy seed, và bỏ qua độ bất ổn này. Một kết quả đơn lẻ vì vậy dễ khiến người đánh giá kết luận quá lạc quan hoặc quá bi quan.

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

| Bộ điều khiển | Xe sau phát lại: va chạm bất kỳ | Xe sau phản ứng: va chạm bất kỳ |
|---|---:|---:|
| constant-velocity | 73,4% | 73,4% |
| IDM | 70,1% | 5,1% |
| EgoMLP | 73,6% | 73,6% |
| PercMLP | 81,8% | 19,3% |
| PercMLP+ | 69,6% | 5,6% |

Ở trễ phản ứng 0,6 s, IDM được hiệu chỉnh với T=1,4 s (Wasserstein 0,34; xe không phản ứng là 0,90), gần người lái có headway trung vị 1,86 s. Khi xe sau chỉ phát lại quỹ đạo đã ghi, nó không phanh theo ego mới và gây va chạm sau giả: riêng IDM có 64,9% ô va chạm phía sau; với xe sau phản ứng, con số này là 0%. Do đó kết luận dựa trên replay phóng đại rủi ro va chạm phía sau; còn các va chạm còn lại trong cột phản ứng chủ yếu là va chạm phía trước.

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

PercMLP bị chặn bởi R2, R3, R4 và R6: cận trên Wilson của va chạm log là 1,884% so với ngưỡng 1,035%; lát cắt phanh là 6,8% so với 5,5%; vùng cut-in tránh được nhưng vẫn đâm là 14,15% so với 2%; thế giới xe sau phản ứng là 19,3% so với ngưỡng 7,1%. Nguyên nhân là log ít có cut-in và lỗi bắt chước nhỏ tích lũy khi planner tự lái. Bản sửa thêm 20.000 cut-in mô phỏng và hai vòng DAgger; vùng cut-in giảm còn 0,41%, va chạm log còn 0,2–0,3%, và cả 7 quy tắc đều pass. PercMLP+ có L2 tốt hơn IDM (0,779–0,817 m so với 1,531 m), nhưng va chạm log vẫn cao hơn IDM (0,2–0,3% so với 0,035%); nó đạt tiêu chí RELEASE của lab, không chứng minh an toàn tuyệt đối. Trước triển khai thật cần kiểm thử độc lập trên dữ liệu đa dạng, mô phỏng phần cứng và thử nghiệm có giám sát.

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

Thứ nhất, dữ liệu chỉ gồm khoảng 9.610 quỹ đạo xe bám xe từ một tập dữ liệu; nó không đại diện đủ thời tiết, loại đường, xe máy, người đi bộ hay lỗi cảm biến, nên kết quả khó khái quát ra giao thông thật. Thứ hai, cut-in và xe sau được mô phỏng bằng mô hình 1D/IDM đơn giản; động lực học, độ trễ phanh và hành vi người lái thật có thể khác, làm tỷ lệ va chạm trong mô phỏng sai lệch. Thứ ba, chỉ có 187 quỹ đạo kiểm tra có xe dẫn phanh và ba seed; khoảng Wilson còn rộng, nên ước lượng rủi ro hiếm chưa chắc chắn. Tiếp theo cần dữ liệu độc lập đa miền, kiểm thử phần cứng-in-the-loop, kiểm thử đường kín có giám sát và đánh giá lại với kịch bản nguy hiểm chưa dùng khi huấn luyện.
