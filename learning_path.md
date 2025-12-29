# Lộ trình học Reverse Engineering (Kỹ nghệ ngược)

Tài liệu này được tổng hợp và xây dựng dựa trên kho tài nguyên [wtsxDev/reverse-engineering](https://github.com/wtsxDev/reverse-engineering). Đây là lộ trình từng bước giúp bạn tiếp cận kỹ thuật Reverse Engineering một cách bài bản.

> [!NOTE]
> Reverse Engineering (RE) là quá trình phân tích một hệ thống phần mềm để xác định các thành phần và cách thức hoạt động của nó. Đây là kỹ năng quan trọng trong bảo mật, phân tích mã độc và phát triển phần mềm.

---

## Giai đoạn 1: Nền tảng (Fundamentals)

Trước khi đi sâu vào công cụ, bạn cần hiểu về cấu trúc máy tính và file thực thi.

### 1. Kiến thức cần có

* **Assembly Language**: Hiểu cách máy tính thực thi lệnh (x86/x64).
* **Binary Formats**: Hiểu cấu trúc file PE (Windows) hoặc ELF (Linux).

### 2. Tài liệu & Sách (Books)

Bắt đầu với cuốn sách miễn phí và nổi tiếng nhất:

* [Reverse Engineering for Beginners](https://beginners.re/) (Dennis Yurichev) - *Khuyên đọc*.
* [The Art of Assembly Language](http://amzn.to/2jlxTNp) - Để nắm vững nền tảng Assembly.

### 3. Công cụ cơ bản (Basic Tools)

Làm quen với việc nhìn dữ liệu dưới dạng Hex và các thông tin file cơ bản.

* **Hex Editors**: [HxD](https://mh-nexus.de/en/hxd/) (Windows) hoặc [010 Editor](http://www.sweetscape.com/010editor/).
* **Binary Format Tools**: [CFF Explorer](http://www.ntcore.com/exsuite.php) hoặc [Detect It Easy (DIE)](http://ntinfo.biz/) để xem thông tin file PE, packer.

---

## Giai đoạn 2: Phân tích Tĩnh (Static Analysis) & Disassembly

Giai đoạn này bạn sẽ đọc mã máy (assembly) mà không chạy chương trình.

### 1. Công cụ Disassemblers

Đây là "vũ khí" chính của Reverser.

* **[IDA Pro](https://www.hex-rays.com/products/ida/index.shtml)**: Tiêu chuẩn công nghiệp. Bản Free là đủ cho người mới bắt đầu.
* **[Ghidra](https://ghidra-sre.org/)**: Công cụ miễn phí mạnh mẽ từ NSA. **Lựa chọn tốt nhất (Best Choice)** nếu bạn muốn một công cụ miễn phí, đầy đủ tính năng thay thế IDA Pro.
* **[x64dbg](http://x64dbg.com/)**: Debugger hiện đại cho Windows. **Khuyên dùng** thay thế OllyDbg cũ.

### 2. Khóa học (Courses)

* [Lenas Reversing for Newbies (GitHub Mirror)](https://github.com/kosmokato/Lena151): Link dự phòng cho bộ tutorial kinh điển (do link gốc Tuts4You đã cũ).
* [Open Security Training](http://opensecuritytraining.info/Training.html): Các khóa học chuyên sâu miễn phí.

---

## Giai đoạn 3: Phân tích Động (Dynamic Analysis) & Debugging

Kết hợp vừa đọc code vừa chạy chương trình để xem hành vi thực tế.

### 1. Debugging

Sử dụng Debugger để đặt breakpoint, xem thanh ghi (registers) và bộ nhớ (memory) khi chương trình đang chạy.

* Công cụ: **x64dbg** (Windows) hoặc **GDB** (Linux).
* Sách: [Inside Windows Debugging](http://amzn.to/2iqFTxf).

### 2. Hành vi hệ thống (System & Network Monitoring)

Quan sát chương trình tương tác với hệ điều hành như thế nào.

* **[Process Monitor](https://technet.microsoft.com/en-us/sysinternals/processmonitor)**: Theo dõi file system, registry, process activity.
* **[Wireshark](https://www.wireshark.org/)**: Phân tích gói tin mạng.
* **[Process Hacker](http://processhacker.sourceforge.net/)**: Quản lý tiến trình mạnh mẽ.

---

## Giai đoạn 4: Thực hành & Nâng cao

Học đi đôi với hành. Hãy thử sức với các bài tập thực tế.

### 1. Luyện tập (Practice/CTF)

* **[Crackmes.one](https://crackmes.one/)** (Thay thế cho crackmes.de cũ): Nơi tải các bài tập "crackme" để bẻ khóa.
* **[Flare-on Challenges](http://flare-on.com/)**: Các bài thi hàng năm của Mandiant, rất chất lượng.
* **[Reverse Engineering Challenges](http://challenges.re/)**.

### 2. Phân tích mã độc (Malware Analysis)

Nếu bạn muốn theo hướng bảo mật.

* Sách: **[Practical Malware Analysis](http://amzn.to/2jljYqE)** - *Cuốn sách "gối đầu giường" cho mảng này*.
* Lab an toàn: Luôn thực hành trong máy ảo (VMware/VirtualBox) để tránh nhiễm virus.

---

## Tóm tắt công cụ cần cài đặt ngay

1. **HxD** (Xem Hex)
2. **IDA Pro Free** hoặc **Ghidra** (Disassembler)
3. **x64dbg** (Debugger)
4. **Detect It Easy** (Kiểm tra file)

Chúc bạn học tập hiệu quả!
