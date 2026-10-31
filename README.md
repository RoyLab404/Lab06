# Lab 06 — Thực hành thử nghiệm và giải thích giải pháp cho bài tập Exercise 5.11 

| Thông tin | Nội dung |
|---|---|
| Họ và tên | Huỳnh Ngọc Khang |
| MSSV | 4651050116 |
| Lớp | CNTT46D |
| Đề bài | Kiểm thử đơn vị, stub và độ bao phủ nhánh |

## 1. Yêu cầu và nguồn sử dụng

Exercise 5.11 yêu cầu kiểm thử các phương thức public của `BiCardSource`, dùng stub để cô lập đối tượng được kiểm thử và đạt 100% branch coverage.

Bài thực hành này **giải thích và chạy lại lời giải có sẵn**, không xây dựng một bộ test mới. Lớp `Exercise11` lấy từ [lời giải của Martin P. Robillard](https://codesample.info/samples/designbook/solutions5/Exercise11.html). Trang lời giải nêu rằng mã có một lỗi được cài sẵn để bộ test phát hiện.

Các lớp `Card`, `CardSource`, `Rank`, `Suit` được chép nguyên từ `DesignBook/code-e3/e3/chapter4`. Chỉ bốn import trong lời giải đổi từ `designbook.chapter4` sang `e3.chapter4` để khớp package của repo hiện tại. Bản quyền và giấy phép trong mã mẫu được giữ nguyên. Trong repo, đường dẫn `solutions-code/chapter5/BiCardSource.java` được đề bài liên kết tới không tồn tại; lớp này có ngay cuối mã trên trang lời giải.

## 2. BiCardSource hoạt động như thế nào?

`BiCardSource implements CardSource` nên có thể được sử dụng thông qua cùng giao diện với một nguồn bài thông thường. Constructor nhận hai `CardSource` và lưu chúng vào `aSource1`, `aSource2`; không tạo bộ bài mới hoặc sao chép các lá bài.

- `draw()`: theo đặc tả, ưu tiên rút từ nguồn 1; chỉ rút từ nguồn 2 khi nguồn 1 rỗng.
- `isEmpty()`: chỉ trả về `true` khi cả hai nguồn đều rỗng.

Nhờ phụ thuộc vào interface `CardSource`, đối tượng có thể nhận nguồn thật hoặc stub mà không cần đổi mã. Đây chính là điểm cho phép cô lập hành vi cần kiểm thử.

## 3. Vì sao dùng CardSourceStub?

Nếu dùng một bộ bài thật, kết quả có thể phụ thuộc vào khởi tạo, thứ tự bài hoặc thao tác trộn bài. Lời giải dùng lớp lồng `CardSourceStub` chỉ chứa tối đa một lá để người kiểm thử kiểm soát chính xác trạng thái đầu vào.

Stub có trường `Optional<Card> aCard`:

| Thao tác | Ý nghĩa |
|---|---|
| `new CardSourceStub()` | Nguồn rỗng, chứa `Optional.empty()` |
| `new CardSourceStub(ACE)` | Nguồn có đúng đối tượng `ACE` |
| `isEmpty()` | Trả về phủ định của `aCard.isPresent()` |
| `draw()` | Kiểm tra nguồn còn bài, lấy lá ra, chuyển nguồn thành rỗng rồi trả lá đó |

Việc xóa lá sau khi rút giúp stub giữ hành vi cơ bản của một nguồn bài: rút một lá thì nguồn mất lá đó. `assert !isEmpty()` biểu diễn tiền điều kiện; cần bật `-ea` để Java thực thi câu lệnh `assert`. Khi tắt assertions, rút từ stub rỗng vẫn gây `NoSuchElementException` tại `Optional.get()`.

Ba fixture là `aEmpty`, `aContainsAce`, `aContainsTwo`. Hai lá mẫu `ACE` và `TWO` đều thuộc chất `CLUBS`, được lấy bằng `Card.get()`. Dùng hai lá khác nhau giúp phát hiện trường hợp rút nhầm nguồn khi cả hai đều còn bài.

Theo vòng đời mặc định của JUnit Jupiter, mỗi test có một đối tượng `Exercise11` mới, nên các trường fixture được khởi tạo lại. Việc một test rút cạn stub không làm thay đổi đầu vào của test khác. Các lá bài static được dùng chung vì chúng là các đối tượng bất biến.

## 4. Giải thích bảy test trong lời giải

### 4.1. Bốn test của isEmpty()

```java
return aSource1.isEmpty() && aSource2.isEmpty();
```

| Test | Nguồn 1 | Nguồn 2 | Kỳ vọng | Lý do |
|---|---|---|---|---|
| `testEmpty_True` | Rỗng | Rỗng | `true` | Không còn nguồn nào có thể rút |
| `testEmpty_False_FirstEmpty` | Rỗng | Có ACE | `false` | Nguồn 2 vẫn còn bài |
| `testEmpty_False_SecondEmpty` | Có ACE | Rỗng | `false` | Nguồn 1 vẫn còn bài |
| `testEmpty_False_NeitherEmpty` | Có ACE | Có TWO | `false` | Cả hai nguồn đều còn bài |

`assertTrue` và `assertFalse` kiểm tra kết quả có đúng đặc tả hay không. Test đầu dùng cùng `aEmpty` cho hai tham số; điều này hợp lệ vì chỉ đọc trạng thái rỗng, không rút hay thay đổi nguồn.

Toán tử `&&` đánh giá ngắn mạch: nếu nguồn 1 không rỗng, toán hạng đầu là `false`, nên `aSource2.isEmpty()` không được gọi. Hai dòng cuối bảng vì thế đi cùng đường thực thi, dù trạng thái dữ liệu khác nhau.

### 4.2. Ba test của draw()

| Test | Nguồn 1 | Nguồn 2 | Kết quả đúng |
|---|---|---|---|
| `testDraw_FirstEmpty` | Rỗng | Có ACE | Trả ACE từ nguồn 2 |
| `testDraw_SecondEmpty` | Có ACE | Rỗng | Trả ACE từ nguồn 1 |
| `testDraw_NeitherEmpty` | Có ACE | Có TWO | Trả ACE vì ưu tiên nguồn 1 |

`assertSame(ACE, ...)` kiểm tra kết quả là **chính đối tượng ACE** đã đưa vào stub. Chỉ kiểm tra kết quả khác `null` sẽ không phát hiện được việc trả nhầm TWO. Đây là kiểm tra danh tính đối tượng, phù hợp vì `BiCardSource` phải chuyển tiếp lá đã rút từ nguồn.

Không có test rút khi cả hai nguồn rỗng vì lời gọi đó vi phạm tiền điều kiện `!isEmpty()` của `CardSource.draw()`. Constructor được thực thi trong các test thông qua `new BiCardSource(...)`; nó chỉ gán hai trường và không có nhánh điều kiện cần bổ sung trường hợp kiểm thử.

## 5. Lỗi mà bộ test phát hiện

Trong mã gốc, điều kiện kiểm tra nguồn 1 còn bài nhưng nhánh đúng lại gọi `aSource2.draw()`; nhánh còn lại gọi `aSource1.draw()`. Hai nguồn được chọn ngược với đặc tả.

- Khi nguồn 1 rỗng, chương trình vẫn rút nguồn 1 và gặp lỗi tiền điều kiện.
- Khi nguồn 2 rỗng, chương trình lại rút nguồn 2 và gặp lỗi tiền điều kiện.
- Khi cả hai còn bài, chương trình rút TWO từ nguồn 2 trong khi test chờ ACE từ nguồn 1.

Thay đổi cần thiết mà lời giải hướng tới là đổi lại hai lệnh trả về:

```java
public Card draw() {
    if (!aSource1.isEmpty()) {
        return aSource1.draw();
    } else {
        return aSource2.draw();
    }
}
```

Sau thay đổi, nguồn 1 được ưu tiên và nguồn 2 đóng vai trò dự phòng. File trong `src` vẫn giữ lỗi gốc để có thể trình diễn việc test phát hiện lỗi. Bản sửa chỉ được tạo trong thư mục tạm khi thực nghiệm, không thay thế mã lời giải gốc trong Lab06.

## 6. Phân tích độ bao phủ nhánh

Phạm vi cần xét là **lớp BiCardSource**, không phải toàn bộ project, các lớp Card hay câu lệnh `assert` bên trong stub.

`draw()` có hai hướng của điều kiện `!aSource1.isEmpty()`. Test `testDraw_FirstEmpty` đi vào `else`; hai test còn lại đi vào `if`.

Với `isEmpty()`, cần xét cả đánh giá ngắn mạch của `&&`:

| Đường thực thi | Test đại diện |
|---|---|
| Toán hạng 1 sai → bỏ qua toán hạng 2 | `testEmpty_False_SecondEmpty` |
| Toán hạng 1 đúng → toán hạng 2 sai | `testEmpty_False_FirstEmpty` |
| Toán hạng 1 đúng → toán hạng 2 đúng | `testEmpty_True` |

Theo cách đếm các nhánh điều kiện trong bytecode thông thường, `draw()` có 2 nhánh, `isEmpty()` có 4 nhánh, tổng cộng 6 nhánh. Bộ test đi qua cả 6 hướng sau khi sửa lỗi, tương ứng 100% branch coverage của lớp. Test `testEmpty_False_NeitherEmpty` không thêm nhánh mới nhưng bổ sung tổ hợp đầu vào; test `testDraw_NeitherEmpty` kiểm tra rõ quy tắc ưu tiên nguồn 1.

**Bao phủ đủ nhánh không đồng nghĩa chương trình đúng.** Mã gốc vẫn có thể đi qua các hướng điều kiện trong khi các assertions của test thất bại. Vì vậy phải xem đồng thời kết quả test và báo cáo coverage. Bảy test cũng chưa chứng minh mọi chuỗi rút bài dài đều đúng; chẳng hạn bộ test không trực tiếp kiểm tra việc chuyển nguồn qua nhiều lần gọi liên tiếp.

## 7. Kết quả thực nghiệm

Đã biên dịch và chạy bằng Java 25, JUnit Platform Console Standalone 1.11.4 (JUnit Jupiter), có bật `-ea`:

| Phiên bản | Tổng test | Đạt | Lỗi |
|---|---:|---:|---:|
| Mã lời giải gốc | 7 | 4 | 3 |
| Bản thử tạm đổi lại hai nguồn trong `draw()` | 7 | 7 | 0 |

Log đầy đủ nằm ở [logs/original.txt](logs/original.txt) và [logs/corrected-experiment.txt](logs/corrected-experiment.txt). Với mã gốc, hai test rút từ nguồn rỗng báo `AssertionError`; test cả hai nguồn còn bài báo nhận TWO thay vì ACE.

Đây là kết quả chạy JUnit bằng dòng lệnh, **chưa phải kết quả thao tác trực tiếp trên Eclipse**. Phần 100% ở trên là phân tích nhánh từ mã; chưa có số đo EclEmma được xuất trong môi trường này.

## 8. Chạy và trình bày trên Eclipse

Project đã có `.project` tên `Lab06`, `.classpath` dùng JavaSE-25 và JUnit 5, cùng thiết lập UTF-8.

1. Chọn **File → Import → General → Existing Projects into Workspace**.
2. Chọn thư mục `Practice/Lab06`, đánh dấu project **Lab06**, rồi **Finish**. Nếu đang dùng workspace `Practice`, bỏ chọn **Copy projects into workspace** để dùng đúng thư mục hiện tại.
3. Trong **Project → Properties → Java Build Path**, kiểm tra JRE là JDK 25 và có **JUnit 5**. Nếu thiếu, dùng **Add Library → JUnit → JUnit 5**; nếu JRE báo lỗi, chọn một JDK 25 đã cài trong **Installed JREs**.
4. Mở `src/designbook/solutions5/Exercise11.java`, chuột phải → **Run As → JUnit Test**. Trong **Run Configurations → JUnit → Arguments**, đặt VM arguments là `-ea` để khớp thực nghiệm.
5. Mã gốc sẽ hiện 7 test, trong đó 3 test lỗi. Chọn từng test lỗi trong JUnit view để đối chiếu với phần 5.
6. Để minh họa bước sửa, đổi lại hai câu `return` theo đoạn mã ở phần 5, lưu và chạy lại. Kết quả mong đợi là 7 test đạt. Có thể hoàn tác hai dòng sau khi trình diễn để giữ nguyên mẫu ban đầu.
7. Nếu Eclipse có EclEmma, chuột phải lớp test → **Coverage As → JUnit Test**. Nếu chưa có, cài **EclEmma Java Code Coverage** qua Eclipse Marketplace và dùng phiên bản hỗ trợ Java 25. Trong Coverage view, mở riêng **BiCardSource** (nằm cùng file `Exercise11.java`) và chọn bộ đếm **Branches** để kiểm tra; không lấy phần trăm của toàn project làm kết quả của bài.

Khi thuyết trình, có thể trình bày theo trình tự: đặc tả hai nguồn → stub tạo trạng thái xác định → bốn test `isEmpty()` và ba test `draw()` → ba lỗi của mã gốc → đổi hai nguồn và chạy lại → giải thích các nhánh đã được bao phủ.

## 9. Tài liệu tham khảo

- [Đề Chapter 5, Exercise 11](https://github.com/prmr/DesignBook/blob/master/exercises/e-chapter5.md).
- [Lời giải chính thức Exercise 11](https://codesample.info/samples/designbook/solutions5/Exercise11.html).
- [Mã mẫu Chapter 4, edition 3](https://github.com/prmr/DesignBook/tree/master/code-e3/e3/chapter4).

Mã DesignBook được sử dụng cho bài thực hành học tập, giữ thông tin tác giả và giấy phép có sẵn trong từng file. Nội dung giải thích trong báo cáo được viết riêng dựa trên việc đọc mã và kết quả chạy thử.
