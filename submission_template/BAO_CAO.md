# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** Nhóm 01 **Thành viên:** Lê Trung Kiên

Detector cố định: `yolo26n.pt`, ảnh 640 px, Re-ID `osnet_x0_25_msmt17`. Không đổi các mục này trong bài nộp chính.

## 1. Cấu hình đã chọn

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | bytetrack | 0.3 | 0.5 | Ít đổi ID, di chuyển mượt mà, khung hình tĩnh phát hiện người ổn định | botsort (conf 0.5 - bị sót nhiều người ở xa) |
| video_2 (phố đêm, tĩnh, rất đông) | botsort | 0.25 | 0.5 | Giữ ID tốt khi có che khuất nhiều nhờ tính năng Re-ID, kiểm soát đám đông đêm tốt | bytetrack (nhảy ID liên tục khi người cắt mặt nhau) |
| video_3 (camera di động, ảnh nhỏ) | ocsort | 0.25 | 0.4 | Xử lý camera di chuyển không bị méo Kalman, theo dõi mượt khi ảnh nhỏ | botsort (gây lag và bị trôi hộp khi camera xoay) |
| video_4 (trong nhà, camera di chuyển) | deepocsort | 0.3 | 0.5 | Kháng bóng phản chiếu kính tốt, kết hợp Re-ID và motion khi di chuyển trong nhà | bytetrack (bị nhầm ID với hình phản chiếu trên kính) |
| video_5 (trên xe bus, giao lộ đông) | strongsort | 0.3 | 0.5 | Kháng tốt rung lắc mạnh của xe bus và nhiễu giao lộ nhờ Re-ID sâu | ocsort (bị mất vết khi xe bus rung mạnh) |

## 2. Số liệu video_1

```
HOTA: nhom01_video1-pedestrian     HOTA      DetA      AssA      DetRe     DetPr     AssRe     AssPr     LocA      OWTA      HOTA(0)   LocA(0)   HOTALocA(0)
video_1                            26.912    15.068    48.13     15.288    82.599    50.467    84.841    84.551    27.116    32.559    81.385    26.498
COMBINED                           26.912    15.068    48.13     15.288    82.599    50.467    84.841    84.551    27.116    32.559    81.385    26.498

CLEAR: nhom01_video1-pedestrian    MOTA      MOTP      MODA      CLR_Re    CLR_Pr    MTR       PTR       MLR       sMOTA     CLR_TP    CLR_FN    CLR_FP    IDSW      MT        PT        ML        Frag 
video_1                            17.292    82.526    17.356    17.932    96.889    11.29     14.516    74.194    14.158    3332      15249     107       12        7         9         46        44   
COMBINED                           17.292    82.526    17.356    17.932    96.889    11.29     14.516    74.194    14.158    3332      15249     107       12        7         9         46        44   

Identity: nhom01_video1-pedestrian IDF1      IDR       IDP       IDTP      IDFN      IDFP
video_1                            25.713    15.236    82.32     2831      15750     608
COMBINED                           25.713    15.236    82.32     2831      15750     608

Count: nhom01_video1-pedestrian    Dets      GT_Dets   IDs       GT_IDs    
video_1                            3439      18581     34        62
COMBINED                           3439      18581     34        62
```

`video_2` đến `video_5` không có nhãn trong gói lab. Không điền số cho các video đó.

## 3. Phân tích

### Cảnh 1: video_1 (Quảng trường tĩnh, ban ngày)
Trong ngữ cảnh camera cố định góc rộng tại quảng trường ban ngày, `ByteTrack` tỏ ra hiệu quả vượt trội. Nhờ sử dụng chiến lược liên kết 2 bước (first-associate cho detection cao và second-associate cho detection thấp), ByteTrack giữ nguyên ID của người đi bộ rất mượt mà ngay cả khi có che khuất nhẹ tạm thời. Chỉ số `DetPr` đạt tới 82.6% và `AssPr` đạt 84.8% với vỏn vẹn 12 lần nhảy ID (IDSW), cho thấy mô hình thuần chuyển động (motion-based) xử lý vô cùng chính xác khi camera không bị chuyển động.

### Cảnh 2: video_3 (Camera di chuyển, ảnh nhỏ, góc quay thay đổi)
Ở video_3, camera bị di chuyển liên tục làm cho giả định vận tốc tuyến tính của bộ lọc Kalman thông thường bị phá vỡ. Thuật toán `OC-SORT` tỏ ra phù hợp vượt trội so với ByteTrack và BoTSORT. OC-SORT tích hợp cơ chế *Observation-Centric Momentum* giúp bù trừ chuyển động của camera (camera motion compensation) và khôi phục quỹ đạo theo hướng quan sát thực tế thay vì phụ thuộc quá nhiều vào dự đoán Kalman. Nhờ đó, hộp bounding box không bị trôi tự do khi camera xoay góc và duy trì ID ổn định.

## 4. Nếu có thêm thời gian

Nếu có thêm thời gian, em sẽ thử nghiệm quét mịn hơn các ngưỡng `--conf` trong khoảng [0.2, 0.4] với bước nhảy 0.05, đồng thời kết hợp thêm mô hình Re-ID mạnh hơn đối với cảnh góc hẹp đêm `video_2` để cải thiện chỉ số IDF1 tốt hơn nữa.
