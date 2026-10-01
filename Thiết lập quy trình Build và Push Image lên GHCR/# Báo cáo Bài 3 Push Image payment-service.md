# Báo cáo Bài 3: Push Image payment-service lên GHCR

## 1. Các lệnh CLI đã sử dụng trong quá trình thực hành

# Chuyển vào thư mục chứa code của payment-service
cd ~/Rikkei_Devops/Session7/payment-service

# Đăng nhập vào GitHub Container Registry (đã ẩn token bảo mật)
echo "***" | sudo docker login ghcr.io -u lehoangviet9 --password-stdin

# Tạo và chỉnh sửa nội dung file Dockerfile (do ban đầu thiếu file)
nano Dockerfile

# Build image với tag được viết thường toàn bộ (chữ lehoangviet9 viết thường để tránh lỗi invalid reference format của Docker)
sudo docker build -t ghcr.io/lehoangviet9/payment-service:1.0.0 .

# Đẩy image lên GitHub Package Registry
sudo docker push ghcr.io/lehoangviet9/payment-service:1.0.0


## 2. Kết quả đạt được
- Image đã được push thành công lên GHCR.
- Link kiểm tra package: https://github.com/lehoangviet9?tab=packages
- (Xem thêm ảnh đính kèm hiển thị version 1.0.0 trên GitHub cá nhân).