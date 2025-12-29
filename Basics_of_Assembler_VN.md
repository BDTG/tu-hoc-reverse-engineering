# Hợp Ngữ (Assembler): Những Kiến Thức Cơ Bản Trong Reversing

*(Dịch từ tài liệu Basics of Assembler - Lena151)*

Quả thực: đây là những điều cơ bản!! Tài liệu này còn lâu mới đầy đủ nhưng bao gồm hầu hết mọi thứ bạn cần biết về assembler để bắt đầu hành trình reversing của mình! Assembler là điểm khởi đầu và kết thúc của mọi ngôn ngữ lập trình. Suy cho cùng, tất cả các ngôn ngữ (máy tính) đều được dịch sang assembler. Trong hầu hết các ngôn ngữ, chúng ta làm việc với các cú pháp tương đối rõ ràng. Tuy nhiên, trong assembler lại là một câu chuyện hoàn toàn khác, nơi chúng ta sử dụng các từ viết tắt, các con số và mọi thứ có vẻ rất kỳ quặc...

## I. Các mảnh, Bit và Byte (Pieces, bits and bytes)

* **BIT**: Mảnh dữ liệu nhỏ nhất có thể. Nó có thể là 0 hoặc 1. Nếu bạn ghép một nhóm các bit lại với nhau, bạn sẽ có "hệ số nhị phân" (binary number system).
  * Ví dụ: `00000001 = 1`, `00000010 = 2`, `00000011 = 3`, v.v.
* **BYTE**: Một byte bao gồm 8 bit. Nó có thể có giá trị tối đa là 255 (0-255). Để đọc các số nhị phân dễ dàng hơn, chúng ta sử dụng "hệ số thập lục phân" (hexadecimal). Nó là "hệ cơ số 16", trong khi nhị phân là "hệ cơ số 2".
* **WORD**: Một word chỉ đơn giản là 2 byte ghép lại với nhau, hay 16 bit. Một word có thể có giá trị tối đa là `0FFFFh` (hoặc `65535d`).
* **DOUBLE WORD** (DWORD): Một double word là 2 word ghép lại, hay 32 bit. Giá trị tối đa = `0FFFFFFFF` (hoặc `4294967295d`).
* **KILOBYTE**: 1000 byte ư? Không, một kilobyte KHÔNG bằng 1000 byte! Thực tế, có 1024 byte (32*32).
* **MEGABYTE**: Một lần nữa, không chỉ là 1 triệu byte, mà là 1024*1024 hay 1,048,576 byte.

## II. Các Thanh Ghi (Registers)

Các thanh ghi là những "nơi đặc biệt" trong bộ nhớ máy tính của bạn, nơi chúng ta có thể lưu trữ dữ liệu. Bạn có thể hình dung một thanh ghi giống như một chiếc hộp nhỏ, bên trong chiếc hộp đó chúng ta có thể chứa một thứ gì đó: một cái tên, một con số, hay một câu. Bạn có thể coi thanh ghi như một vật chứa tạm thời (placeholder).

Trên các CPU chuẩn WinTel (Windows + Intel) ngày nay, bạn có các thanh ghi 32-bit (không tính các thanh ghi cờ). Tên của chúng là:

* **EAX**: Thanh ghi Tích lũy Mở rộng (Extended Accumulator Register). Thường dùng để chứa kết quả tính toán.
* **EBX**: Thanh ghi Cơ sở Mở rộng (Extended Base Register).
* **ECX**: Thanh ghi Bộ đếm Mở rộng (Extended Counter Register). Thường dùng làm biến đếm vòng lặp.
* **EDX**: Thanh ghi Dữ liệu Mở rộng (Extended Data Register).
* **ESI**: Chỉ số Nguồn Mở rộng (Extended Source Index).
* **EDI**: Chỉ số Đích Mở rộng (Extended Destination Index).
* **EBP**: Con trỏ Cơ sở Mở rộng (Extended Base Pointer). Dùng để quản lý Stack Frame.
* **ESP**: Con trỏ Ngăn xếp Mở rộng (Extended Stack Pointer). Trỏ tới đỉnh Stack.

*Lưu ý: Chữ "Extended" (Mở rộng) nghĩa là phiên bản 32-bit.*

---
Các thanh ghi đặc biệt khác:

* **EIP**: Con trỏ Lệnh Mở rộng (Extended Instruction Pointer).
  * **Cực kỳ quan trọng**: Nó luôn chỉ vào dòng lệnh tiếp theo sắp được chạy.
  * Đây là thanh ghi quan trọng nhất khi Debug. Bạn thay đổi EIP nghĩa là bạn thay đổi dòng lệnh mà CPU sắp chạy.

---

### Phân loại theo kích thước

Mặc dù các thanh ghi đều lớn 32-bit, nhưng một số phần của chúng (16-bit hoặc 8-bit) có thể truy cập được:

**i. Thanh ghi kích thước byte (8-bit):**

* `AL`, `AH` (Phần thấp/cao của AX)
* `BL`, `BH`
* `CL`, `CH`
* `DL`, `DH`

**ii. Thanh ghi kích thước word (16-bit):**

* `AX` = `AH` + `AL`. (Lưu ý: Thay đổi AH/AL sẽ làm thay đổi AX).
* `BX`, `CX`, `DX`, `SI`, `DI`, `SP`, `BP`.

**iii. Thanh ghi kích thước Doubleword (32-bit):**

* `EAX`, `EBX`, `ECX`... (Thêm chữ 'E' vào trước thanh ghi 16-bit).

*Ví dụ minh họa:*

```text
|-------------------|-------------------|  <-- EAX (32-bit)
                    |-------------------|  <-- AX  (16-bit)
                    |--------|----------|
                       AH        AL        <-- AH, AL (8-bit)
```

## III. Các cờ (Flags)

Cờ là các bit đơn lẻ biểu thị trạng thái (1 = Bật/Set, 0 = Tắt/Cleared). Trong reversing, bạn cần quan tâm nhất 3 cờ sau để hiểu các lệnh nhảy (Jump):

1. **Z-Flag (Zero Flag - Cờ số 0)**:
    * Quan trọng nhất (dùng trong 90% trường hợp).
    * Được bật (1) khi kết quả phép toán cuối cùng bằng 0.
    * Ví dụ: Lệnh `CMP A, B` (So sánh A và B). Nếu A bằng B, kết quả trừ nhau bằng 0, nên Z-Flag sẽ bật.

2. **O-Flag (Overflow Flag - Cờ tràn)**:
    * Dùng trong khoảng 4% trường hợp.
    * Được bật khi phép toán làm thay đổi bit cao nhất (bit dấu) của kết quả, gây sai lệch số học (tràn số).

3. **C-Flag (Carry Flag - Cờ nhớ)**:
    * Dùng trong khoảng 1% trường hợp.
    * Được bật khi phép cộng vượt quá giới hạn (tràn trên > FFFFFFFF) hoặc phép trừ nhỏ hơn giới hạn (tràn dưới < 0).

## IV. Đoạn (Segments) và Độ lệch (Offsets)

Một **Segment** (Đoạn) là một phần trong bộ nhớ nơi lưu trữ các chỉ lệnh máy (CS - Code Segment), dữ liệu (DS - Data Segment), ngăn xếp (SS - Stack Segment) hoặc chỉ là một đoạn bổ sung (ES - Extra Segment). Mỗi segment được chia thành các '**offsets**' (độ lệch).

Trong các ứng dụng 32-bit, các offset này được đánh số từ `00000000` đến `FFFFFFFF`.

Ký hiệu tiêu chuẩn cho segment và offset là:
`SEGMENT : OFFSET`
= Cùng với nhau, chúng trỏ đến một vị trí cụ thể (địa chỉ) trong bộ nhớ.

Hãy hình dung thế này cho dễ hiểu:

* **Segment** là một **trang** trong cuốn sách.
* **Offset** là một **dòng cụ thể** trên trang đó.

## V. Ngăn xếp (The Stack)

**Stack** (Ngăn xếp) là một phần trong bộ nhớ nơi bạn có thể lưu trữ những thứ khác nhau để sử dụng sau này.

Hãy coi nó như một chồng sách trong một chiếc hòm, nơi **cuốn sách cuối cùng được đặt vào là cuốn đầu tiên được lấy ra** (LIFO - Last In First Out). Hoặc tưởng tượng stack như một giỏ giấy nơi bạn bỏ các tờ giấy vào. Cái giỏ là Stack và tờ giấy là một địa chỉ bộ nhớ (được chỉ định bởi con trỏ stack - Stack Pointer) trong đoạn stack đó.

Hãy nhớ quy tắc sau: **tờ giấy cuối cùng bạn bỏ vào giỏ, là tờ đầu tiên bạn lấy ra!**

* Lệnh **`PUSH`** (đẩy): lưu nội dung của một thanh ghi vào stack.
* Lệnh **`POP`** (lấy ra): lấy nội dung được lưu cuối cùng từ stack ra và đặt nó vào một thanh ghi cụ thể.

## VI. CÁC LỆNH (INSTRUCTIONS)

*Lưu ý: Tất cả các giá trị trong ASM mnemonics (các lệnh gợi nhớ) **luôn luôn** là số thập lục phân (hexadecimal).*

Hầu hết các lệnh có hai toán hạng (operand) (ví dụ `add EAX, EBX`), nhưng một số chỉ có một (`not EAX`) hoặc thậm chí ba (`IMUL EAX, EDX, 64`).

Khi bạn thấy một lệnh có ghi `DWORD PTR [XXX]`, điều đó có nghĩa là giá trị DWORD (4 byte) tại **địa chỉ bộ nhớ** `[XXX]`. Hãy lưu ý rằng các byte được lưu theo thứ tự ngược lại trong bộ nhớ (WinTel CPUs sử dụng định dạng gọi là "**Little Endian**"). Tương tự với `WORD PTR [XXX]` (2 byte) và `BYTE PTR [XXX]` (1 byte).

Hầu hết các lệnh có 2 toán hạng có thể được sử dụng theo các cách sau (ví dụ với lệnh `ADD`):

```assembly
add eax, ebx                      ;; Register, Register (Thanh ghi, Thanh ghi)
add eax, 123                      ;; Register, Value (Thanh ghi, Giá trị)
add eax, dword ptr [404000]       ;; Register, Dword Pointer [value]
add eax, dword ptr [eax]          ;; Register, Dword Pointer [register]
add eax, dword ptr [eax+00404000] ;; Register, Dword Pointer [register+value]
add dword ptr [404000], eax       ;; Dword Pointer [value], Register
add dword ptr [404000], 123       ;; Dword Pointer [value], Value
add dword ptr [eax], eax          ;; Dword Pointer [register], Register
add dword ptr [eax], 123          ;; Dword Pointer [register], Value
add dword ptr [eax+404000], eax   ;; Dword Pointer [register+value], Register
add dword ptr [eax+404000], eax   ;; Dword Pointer [register+value], Register
add dword ptr [eax+404000], 123   ;; Dword Pointer [register+value], Value
```

---------------------------------------------------------------------------------------------

### 1. ADD (Addition - Phép cộng)

**Cú pháp:** `ADD đích, nguồn` (ADD destination, source)

Lệnh `ADD` cộng một giá trị vào một thanh ghi hoặc một địa chỉ bộ nhớ.

Lệnh này có thể bật **Z-Flag**, **O-Flag** và **C-Flag** (và một số cờ khác nhưng không cần thiết cho việc bẻ khóa).

### 2. AND (Logical And - Phép VÀ logic)

**Cú pháp:** `AND đích, nguồn` (AND destination, source)

Lệnh `AND` thực hiện phép toán logic AND trên hai giá trị.
Lệnh này **sẽ xóa** (clear/tắt) **O-Flag** và **C-Flag** và có thể bật **Z-Flag**.

Để hiểu rõ hơn về AND, hãy xem xét hai giá trị nhị phân sau:

```text
  1001010110
  0101001101
  ----------
  0001000100  (Kết quả)
```

Khi hai số **1** nằm thẳng hàng với nhau, kết quả của bit đó là **1**, nếu không: Kết quả là **0**.
Bạn có thể sử dụng `calc.exe` để tính toán AND một cách dễ dàng.

### 3. CALL (Gọi hàm)

**Cú pháp:** `CALL cái_gì_đó` (CALL something)

Lệnh `CALL` đẩy RVA (**R**elative **V**irtual **A**ddress - Địa chỉ ảo tương đối) của lệnh tiếp theo sau lệnh CALL đó vào Stack và sau đó gọi một chương trình con/thủ tục (sub program/procedure).

`CALL` có thể được sử dụng theo các cách sau:

```assembly
CALL 404000              ;; PHỔ BIẾN NHẤT: CALL ADDRESS (Gọi địa chỉ cụ thể)
CALL EAX                 ;; CALL REGISTER (Gọi thanh ghi) - NẾU EAX LÀ 404000 THÌ NÓ GIỐNG HỆT CÂU TRÊN
CALL DWORD PTR [EAX]     ;; GỌI ĐỊA CHỈ ĐƯỢC LƯU TẠI [EAX]
CALL DWORD PTR [EAX]     ;; GỌI ĐỊA CHỈ ĐƯỢC LƯU TẠI [EAX]
CALL DWORD PTR [EAX+5]   ;; GỌI ĐỊA CHỈ ĐƯỢC LƯU TẠI [EAX+5]
```

### 4. CDQ (Convert DWord to QWord)

**Cú pháp:** `CDQ`

`CDQ` là một lệnh luôn khiến người mới bắt đầu bối rối khi gặp lần đầu. Nó chủ yếu được sử dụng trước các phép chia và không làm gì khác ngoài việc **gán tất cả các byte của EDX** theo giá trị của **bit cao nhất của EAX**.

* Nghĩa là: nếu EAX < 80000000 (số dương), thì EDX sẽ là `00000000`.
* Nếu EAX >= 80000000 (số âm), thì EDX sẽ là `FFFFFFFF`.

### 5. CMP (Compare - So sánh)

**Cú pháp:** `CMP đích, nguồn`

Lệnh `CMP` so sánh hai thứ và có thể bật các cờ **C/O/Z** nếu kết quả phù hợp.

```assembly
CMP EAX, EBX        ;; So sánh EAX và EBX, bật Z-flag nếu chúng bằng nhau
CMP EAX, [404000]   ;; So sánh EAX với giá trị dword tại địa chỉ 404000
CMP [404000], EAX   ;; So sánh EAX với giá trị dword tại địa chỉ 404000
```

*Thực chất: CMP thực hiện phép TRỪ (SUB) nhưng không lưu kết quả, chỉ cập nhật Cờ.*

### 6. DEC (Decrement - Giảm)

**Cú pháp:** `DEC cái_gì_đó`

`DEC` được dùng để giảm một giá trị đi 1 đơn vị (tức là: `value = value - 1`).

`DEC` có thể được dùng theo các cách sau:

```assembly
DEC EAX             ;; Giảm EAX đi 1
DEC [EAX]           ;; Giảm dword được lưu tại [EAX]
DEC [401000]        ;; Giảm dword được lưu tại [401000]
DEC [EAX+401000]    ;; Giảm dword được lưu tại [EAX+401000]
```

Lệnh `DEC` có thể bật các cờ **Z/O** nếu kết quả phù hợp.

### 7. DIV (Division - Phép chia không dấu)

**Cú pháp:** `DIV số_chia` (DIV divisor)

`DIV` được dùng để chia **EAX** cho số chia (phép chia không dấu). Số bị chia luôn là EAX, thương số (kết quả) được lưu trong **EAX**, số dư (modulo) được lưu trong **EDX**.

Ví dụ:

```assembly
mov eax, 64      ;; EAX = 64h = 100 (thập phân)
mov ecx, 9       ;; ECX = 9
div ecx          ;; CHIA EAX CHO ECX
```

Sau phép chia: `EAX = 100 / 9 = 0B` (11 thập phân) và `EDX = 100 MOD 9 = 1` (số dư).

Lệnh `DIV` có thể bật các cờ **C/O/Z** nếu kết quả phù hợp.

### 8. IDIV (Integer Division - Phép chia số nguyên có dấu)

**Cú pháp:** `IDIV số_chia` (IDIV divisor)

`IDIV` hoạt động tương tự như `DIV`, nhưng `IDIV` là phép chia **có dấu** (signed division).
Lệnh `idiv` có thể bật các cờ **C/O/Z** nếu kết quả phù hợp.

### 9. IMUL (Integer Multiplication - Phép nhân số nguyên)

**Cú pháp:**

* `IMUL giá_trị`
* `IMUL đích, giá_trị_1, giá_trị_2`
* `IMUL đích, giá_trị`

`IMUL` nhân EAX với một giá trị (`IMUL value`), hoặc nhân hai giá trị và đặt kết quả vào thanh ghi đích (`IMUL dest, value, value`), hoặc nhân một thanh ghi với một giá trị (`IMUL dest, value`).

Nếu kết quả phép nhân quá lớn so với thanh ghi đích, các cờ **O/C** sẽ được bật. Cờ **Z** cũng có thể được bật.

### 10. INC (Increment - Tăng)

**Cú pháp:** `INC thanh_ghi` (INC register)

`INC` là lệnh ngược lại với lệnh `DEC`; nó tăng giá trị lên 1.
`INC` có thể bật các cờ **Z/O**.

### 11. INT (Interrupt - Ngắt)

**Cú pháp:** `INT đích` (INT dest)

Tạo ra một cuộc gọi đến trình xử lý ngắt (interrupt handler). Giá trị đích phải là một số nguyên (ví dụ: `Int 21h`).
`INT3` và `INTO` là các lệnh gọi ngắt không nhận tham số nhưng sẽ gọi trình xử lý cho ngắt 3 và 4 tương ứng.
*(`INT 3` là một breakpoint phần mềm phổ biến mà debugger hay dùng).*

### 12. JUMPS (Các lệnh Nhảy)

Đây là các lệnh nhảy quan trọng nhất và điều kiện cần có để chúng được thực thi.
*(Các lệnh quan trọng được đánh dấu **\*** và rất quan trọng được đánh dấu **\*\***)*

| Lệnh | Ý nghĩa (Tiếng Anh) | Điều kiện (Cờ) | Ghi chú (Tiếng Việt) |
| :--- | :--- | :--- | :--- |
| **JA\*** | Jump if (unsigned) above | CF=0 và ZF=0 | Nhảy nếu lớn hơn (số không dấu) |
| **JAE** | Jump if (unsigned) above or equal | CF=0 | Nhảy nếu lớn hơn hoặc bằng (số không dấu) |
| **JB\*** | Jump if (unsigned) below | CF=1 | Nhảy nếu nhỏ hơn (số không dấu) |
| **JBE** | Jump if (unsigned) below or equal | CF=1 hoặc ZF=1 | Nhảy nếu nhỏ hơn hoặc bằng (số không dấu) |
| **JC** | Jump if carry flag set | CF=1 | Nhảy nếu cờ nhớ (Carry) được bật |
| **JCXZ** | Jump if CX is 0 | CX=0 | Nhảy nếu CX bằng 0 |
| **JE\*\*** | Jump if equal | ZF=1 | **Nhảy nếu bằng** (Rất quan trọng) |
| **JECXZ** | Jump if ECX is 0 | ECX=0 | Nhảy nếu ECX bằng 0 |
| **JG\*** | Jump if (signed) greater | ZF=0 và SF=OF | Nhảy nếu lớn hơn (số có dấu) |
| **JGE\*** | Jump if (signed) greater or equal | SF=OF | Nhảy nếu lớn hơn hoặc bằng (số có dấu) |
| **JL\*** | Jump if (signed) less | SF != OF | Nhảy nếu nhỏ hơn (số có dấu) |
| **JLE\*** | Jump if (signed) less or equal | ZF=1 và SF != OF | Nhảy nếu nhỏ hơn hoặc bằng (số có dấu) |
| **JMP\*\***| Jump | - | **Luôn luôn nhảy** (Nhảy không điều kiện) |
| **JNA** | Jump if (unsigned) not above | CF=1 hoặc ZF=1 | Nhảy nếu không lớn hơn (tương đương JBE) |
| **JNAE** | Jump if (unsigned) not above or equal | CF=1 | Nhảy nếu không lớn hơn hoặc bằng (tương đương JB) |
| **JNB** | Jump if (unsigned) not below | CF=0 | Nhảy nếu không nhỏ hơn (tương đương JAE) |
| **JNBE** | Jump if (unsigned) not below or equal | CF=0 và ZF=0 | Nhảy nếu không nhỏ hơn hoặc bằng (tương đương JA) |
| **JNC** | Jump if carry flag not set | CF=0 | Nhảy nếu cờ nhớ không bật |
| **JNE\*\***| Jump if not equal | ZF=0 | **Nhảy nếu không bằng** (Rất quan trọng) |
| **JNG** | Jump if (signed) not greater | ZF=1 hoặc SF!=OF | Nhảy nếu không lớn hơn (tương đương JLE) |
| **JNGE** | Jump if (signed) not greater or equal | SF!=OF | Nhảy nếu không lớn hơn hoặc bằng (tương đương JL) |
| **JNL** | Jump if (signed) not less | SF=OF | Nhảy nếu không nhỏ hơn (tương đương JGE) |
| **JNLE** | Jump if (signed) not less or equal | ZF=0 và SF=OF | Nhảy nếu không nhỏ hơn hoặc bằng (tương đương JG) |
| **JNO** | Jump if overflow flag not set | OF=0 | Nhảy nếu cờ tràn không bật |
| **JNP** | Jump if parity flag not set | PF=0 | Nhảy nếu cờ chẵn lẻ không bật |
| **JNS** | Jump if sign flag not set | SF=0 | Nhảy nếu cờ dấu không bật (số dương) |
| **JNZ** | Jump if not zero | ZF=0 | Nhảy nếu không bằng 0 (tương đương JNE) |
| **JO** | Jump if overflow flag is set | OF=1 | Nhảy nếu cờ tràn được bật |
| **JP** | Jump if parity flag set | PF=1 | Nhảy nếu cờ chẵn lẻ được bật |
| **JPE** | Jump if parity is equal | PF=1 | Nhảy nếu chẵn lẻ bằng nhau |
| **JPO** | Jump if parity is odd | PF=0 | Nhảy nếu chẵn lẻ là lẻ |
| **JS** | Jump if sign flag is set | SF=1 | Nhảy nếu cờ dấu được bật (số âm) |

### 13. LEA (Load Effective Address)

**Cú pháp:** `LEA đích, nguồn` (LEA dest, src)

`LEA` có thể được coi tương tự như lệnh `MOV`. Nó không được sử dụng nhiều cho chức năng gốc (tính địa chỉ), mà thường dùng để thực hiện **phép nhân nhanh** như sau:

```assembly
lea eax, dword ptr [4*ecx+ebx]
```

Lệnh trên sẽ gán cho `EAX` giá trị bằng `4 * ECX + EBX`.

### 14. MOV (Move - Di chuyển/Sao chép)

**Cú pháp:** `MOV đích, nguồn` (MOV dest, src)

Đây là một lệnh dễ hiểu. `MOV` sao chép giá trị từ `nguồn` sang `đích`, và giá trị tại `nguồn` vẫn giữ nguyên.

Có một số biến thể của MOV:

* **MOVS/MOVSB/MOVSW/MOVSD EDI, ESI**: Các biến thể này sao chép byte/word/dword mà **ESI** trỏ tới, sang vùng nhớ mà **EDI** trỏ tới.
* **MOVSX**: Mở rộng toán hạng Byte hoặc Word thành kích thước Word hoặc Dword và **giữ nguyên dấu** của giá trị.
* **MOVZX**: Mở rộng toán hạng Byte hoặc Word thành kích thước Word hoặc Dword và **điền phần còn lại bằng số 0**.

### 15. MUL (Multiplication - Phép nhân không dấu)

**Cú pháp:** `MUL giá_trị`

Lệnh này giống với `IMUL`, ngoại trừ việc nó nhân số **không dấu**. Nó có thể bật các cờ **O/Z/F** (Overflow, Zero, Flag).

### 16. NOP (No Operation - Không làm gì cả)

**Cú pháp:** `NOP`

Lệnh này hoàn toàn **không làm gì cả**!
Đó chính là lý do tại sao nó được sử dụng rất thường xuyên trong reversing ;)
*(Ví dụ: Dùng để xóa bỏ một lệnh kiểm tra lỗi hoặc xóa bỏ một lệnh nhảy không mong muốn).*

### 17. OR (Logical Inclusive Or - Phép HOẶC logic)

**Cú pháp:** `OR đích, nguồn` (OR dest, src)

Lệnh `OR` kết nối hai giá trị bằng cách sử dụng phép OR logic.
Lệnh này xóa **O-Flag** và **C-Flag** và có thể bật **Z-Flag**.

Ví dụ minh họa:

```text
  1001010110
  0101001101
  ----------
  1101011111  (Kết quả)
```

Chỉ khi nào có hai số **0** nằm chồng lên nhau thì bit kết quả mới là **0**. Còn lại (có ít nhất một số 1) thì kết quả là **1**. Bạn có thể dùng `calc.exe` để tính thử.

### 18. POP

**Cú pháp:** `POP đích` (POP dest)

 `POP` lấy giá trị byte/word/dword tại `[ESP]` (đỉnh Stack) và đặt nó vào `đích`. Đồng thời, nó tăng Stack Pointer (ESP) lên bằng kích thước giá trị vừa lấy ra, để lệnh POP tiếp theo sẽ lấy giá trị kế tiếp.

### 19. PUSH

**Cú pháp:** `PUSH toán_hạng` (PUSH operand)

`PUSH` ngược lại với `POP`. Nó lưu một giá trị vào Stack và giảm Stack Pointer (ESP) đi bằng kích thước của giá trị vừa được push, để ESP luôn trỏ vào giá trị mới nhất vừa được PUSH vào.

### 20. REP/REPE/REPZ/REPNE/REPNZ

**Cú pháp:** `REP... lệnh_chuỗi` (ins)

Lặp lại lệnh xử lý chuỗi theo sau: Lặp lại lệnh `ins` cho đến khi `CX=0` hoặc cho đến khi điều kiện chỉ định (`ZF=1`, `ZF=0`...) được thỏa mãn.
Giá trị `ins` phải là một lệnh thao tác chuỗi như `CMPS`, `INS`, `LODS`, `MOVS`, `OUTS`, `SCAS`, hoặc `STOS`.

### 21. RET (Return - Trả về)

**Cú pháp:**

* `RET`
* `RET số` (digit)

`RET` không làm gì khác ngoài việc trả về (thoát) từ một đoạn mã đã được gọi bằng lệnh `CALL`.
`RET số` sẽ dọn dẹp Stack (cộng thêm `số` vào ESP) trước khi trả về.

### 22. SUB (Subtraction - Phép trừ)

**Cú pháp:** `SUB đích, nguồn` (SUB dest, src)

`SUB` ngược lại với lệnh `ADD`. Nó trừ giá trị của `nguồn` khỏi giá trị của `đích` và lưu kết quả vào `đích`.

Lệnh `SUB` có thể bật các cờ **Z/O/C**.

---

### 23. TEST

**Cú pháp:** `TEST toán_hạng1, toán_hạng2` (TEST operand1, operand2)

Lệnh này trong 99% trường hợp được dùng dưới dạng `TEST EAX, EAX`. Nó thực hiện phép **AND Logic** (giống lệnh AND) nhưng **không lưu** kết quả thay đổi vào toán hạng.

Nó chỉ cập nhật các cờ:

* Bật **Z-Flag** (ZF=1) nếu kết quả là 0 (Ví dụ `TEST EAX, EAX` mà EAX bằng 0).
* Xóa **Z-Flag** (ZF=0) nếu kết quả khác 0.
* Các cờ **O/C** luôn bị xóa.

### 24. XOR (Logical Exclusive Or - Phép XOR)

**Cú pháp:** `XOR đích, nguồn` (XOR dest, src)

Lệnh `XOR` thực hiện phép logic **Exclusive OR** (HOẶC loại trừ).
Lệnh này xóa **O-Flag** và **C-Flag**, và có thể bật **Z-Flag**.

Để hiểu rõ hơn về XOR, hãy xem hai giá trị nhị phân sau:

```text
  1001010110
  0101001101
  ----------
  1100011011  (Kết quả)
```

Quy tắc:

* Nếu hai bit **giống nhau** (cùng 0 hoặc cùng 1) -> Kết quả là **0**.
* Nếu hai bit **khác nhau** (một cái 0, một cái 1) -> Kết quả là **1**.

**Mẹo quan trọng:**
Trường hợp thường gặp nhất là `XOR EAX, EAX`.
Lệnh này sẽ gán **EAX = 0**, bởi vì khi bạn XOR một giá trị với chính nó, kết quả luôn luôn là 0. (Đây là cách nhanh nhất và phổ biến nhất để reset một thanh ghi về 0).

---

## VII. Các phép toán Logic (Logical Operations)

Dưới đây là **Bảng Tham Khảo** (Reference Table) các phép toán được sử dụng nhiều nhất:

| Phép toán (Operation) | Nguồn (src) | Đích (dest) | Kết quả (result) | Ghi chú (Tiếng Việt) |
| :--- | :---: | :---: | :---: | :--- |
| **AND** | 1 | 1 | **1** | Cả hai cùng là 1 |
| | 1 | 0 | **0** | |
| | 0 | 1 | **0** | |
| | 0 | 0 | **0** | |
| **OR** | 1 | 1 | **1** | |
| | 1 | 0 | **1** | Có ít nhất một số 1 |
| | 0 | 1 | **1** | |
| | 0 | 0 | **0** | Cả hai cùng là 0 |
| **XOR** | 1 | 1 | **0** | Giống nhau ra 0 |
| | 1 | 0 | **1** | Khác nhau ra 1 |
| | 0 | 1 | **1** | |
| | 0 | 0 | **0** | |
| **NOT** | 0 | N/A | **1** | Đảo ngược bit |
| | 1 | N/A | **0** | |

---------------------------------------------------------------------------------------------
*(Vâng, quả thực, đây đã là KẾT THÚC ☺ Xin lỗi nếu có sai sót trong văn bản này).*

---
**Tổng kết:**
Vậy là bạn đã nắm được các lệnh cơ bản nhất rồi đấy! Đừng lo nếu chưa nhớ hết ngay lập tức. Hãy vừa làm bài tập vừa tra cứu tài liệu này nhé. Chúc bạn thành công!
