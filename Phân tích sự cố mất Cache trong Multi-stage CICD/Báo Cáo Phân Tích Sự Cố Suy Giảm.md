Báo Cáo Phân Tích Sự Cố: Suy Giảm Hiệu Suất Trích Xuất Cache Trong Multi-stage Build
1. Tổng quan sự cố
Việc chuyển đổi quy trình CI/CD sang kiến trúc Multi-stage Dockerfile đã làm tăng thời gian build của order-service từ 1.5 phút lên 4-5 phút. Nguyên nhân trực tiếp là do công cụ Gradle liên tục tải lại toàn bộ thư viện (dependencies) qua Internet ở mỗi lần chạy, không tận dụng được cache đã tồn tại trên Self-hosted Runner.

2. Phân tích nguyên nhân gốc rễ (Root Cause)
Sự cố này xuất phát từ bản chất không gian lưu trữ và tính cô lập môi trường của tiến trình đóng gói Docker, cụ thể qua các yếu tố kỹ thuật sau:

Tính cô lập của tiến trình Builder: Trong pipeline cũ, lệnh biên dịch mã nguồn được thực thi trực tiếp trên hệ điều hành của máy chủ Runner. Do đó, tiến trình này dễ dàng truy xuất và kế thừa thư mục lưu trữ nội bộ ~/.gradle trên host. Ngược lại, với Multi-stage build, lệnh RUN ./gradlew bootJar ở Stage 1 lại được thực thi hoàn toàn bên trong một container tạm thời (ephemeral container). Hệ thống tệp (filesystem) của container này được khởi tạo hoàn toàn trống rỗng ở mỗi lần build và bị cô lập với máy chủ vật lý bên ngoài.

Cơ chế giới hạn của Build Context: Lệnh docker build tiêu chuẩn chỉ cho phép builder container nhận dữ liệu cục bộ thông qua các lệnh COPY từ thư mục dự án hiện tại. Quá trình này không tự động ánh xạ (mount) các thư mục hệ thống như ~/.gradle từ máy chủ Runner vào bên trong vùng không gian của container Stage 1. Hậu quả là tiến trình Gradle bên trong container luôn nhận diện đây là một môi trường hoàn toàn mới và buộc phải kéo lại toàn bộ thư viện từ đầu.

Giới hạn của Docker-out-of-Docker (DooD): Việc cấu hình DooD (chia sẻ docker.sock) chỉ cung cấp cho khối CI/CD quyền giao tiếp và ra lệnh cho Docker Engine trên máy chủ host khởi tạo các tiến trình. Nó không có chức năng chia sẻ hoặc đồng bộ hóa hệ thống tệp tin giữa máy chủ host và vùng không gian cô lập của các container đang trong quá trình đóng gói.

Việc ứng dụng Multi-stage build mang lại ưu điểm tuyệt đối về bảo mật mã nguồn thô và tối ưu hóa dung lượng image đầu ra ở Stage cuối cùng. Tuy nhiên, sự đánh đổi cấu trúc này là sự phá vỡ liên kết với bộ nhớ đệm (cache) vật lý trên máy chủ. Để khôi phục tốc độ biên dịch mà vẫn giữ nguyên kiến trúc Multi-stage, hệ thống sẽ cần tích hợp các cơ chế bộ nhớ đệm nội sinh của Docker nâng cao (như Docker BuildKit Cache Mounts) để duy trì trạng thái thư viện giữa các phiên bản build độc lập.