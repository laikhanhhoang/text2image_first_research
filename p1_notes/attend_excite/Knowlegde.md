THÔNG SỐ HÌNH HỌC (SHAPE) CÁC KHỐI TRONG UNET (STABLE DIFFUSION) - CẬP NHẬT PHÉP CHIẾU
Kích thước Latent chuẩn: 64 x 64

1. TRẠM RESNET BLOCK (NHÚNG THỜI GIAN)
- Input 1 (Feature Map): [B, C, H, W] (Ví dụ: [1, 320, 64, 64])
- Input 2 (Time Embedding): [B, D] (Ví dụ: [1, 1280])
- Tính toán: Vector thời gian được Project (qua lớp Linear) lên cùng số kênh C, cộng vào Feature Map.
- Output: [B, C, H, W] (Kích thước giữ nguyên, giá trị bị biến đổi bởi timestep t).

2. TRẠM CROSS-ATTENTION (NHÚNG VĂN BẢN) - CẬP NHẬT CHI TIẾT
- Input Hình ảnh (Q_in): [B, H*W, C] (Ví dụ: [1, 4096, 320])
- Input Văn bản (K_in, V_in): [B, L, D_clip] (Ví dụ: [1, 77, 768] từ CLIP)

- PHÉP CHIẾU (PROJECTION):
  + Q = Q_in x W_q  => Shape: [B, H*W, d] (với d là dimension của Head, ví dụ 320)
  + K = K_in x W_k  => Shape: [B, L, d]   (Ép từ 768 về 320 để khớp với Q)
  + V = V_in x W_v  => Shape: [B, L, d]   (Ép từ 768 về 320)

- TÍNH TOÁN: 
  + Attention Map = Softmax(Q @ K.T / sqrt(d)) => Shape: [B, H*W, L] (Ma trận 4096 x 77)
  + Ý nghĩa: Đây là Cross-attention map, liên kết từng Pixel với từng Token chữ.

- OUTPUT: (Attention Map @ V) sau đó qua lớp Linear cuối => Shape: [B, C, H, W].

3. TRẠM SELF-ATTENTION (TƯƠNG TÁC NỘI BỘ ẢNH)
- Input: [B, H*W, C] (Dùng chung cho cả Q, K, V qua các lớp chiếu W_q, W_k, W_v nội bộ).
- Tính toán: Attention Map = Softmax(Q @ K.T / sqrt(d)) => Shape: [B, H*W, H*W] (Ma trận 4096 x 4096).
- Ý nghĩa: Mỗi pixel tự so sánh với toàn bộ các pixel khác để giữ tính nhất quán vật thể.
- Output: [B, C, H, W].

GHI CHÚ NGHIÊN CỨU (ATTRIBUTE BINDING):
- Lỗi Binding thường nằm ở ma trận [H*W, L] của Trạm 2.
- Nếu Token "Red" có trọng số cao tại vị trí Pixel của "Car", xe sẽ có màu đỏ.
- Nếu trọng số "Red" bị tràn sang vùng Pixel của "Tree", lỗi Binding xảy ra (cây bị đỏ).