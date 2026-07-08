# One Stroke - LYKN CTF Writeup

## 1. Thông tin challenge

- Tên challenge: `One Stroke`
- Tác giả: `@yuneko.dev (Ellie)`
- Điểm: `500`
- File được cung cấp: `chall.jpg`
- Flag format: `LYKNCTF{<uuid>}`

Đề bài:

```text
This monochrome image doesn't seem to contain anything useful.
Or perhaps you're just looking at it the wrong way.
```

Các hint quan trọng:

```text
Nearby pixels tend to stay nearby. Try following a path instead of rows.
It's just a pixel permutation.
```

Flag tìm được:

```text
LYKNCTF{0548fcfa-6785-4051-a54d-8006b2f4828b}
```

## 2. Nhận xét ban đầu

File challenge là một ảnh JPEG tên `chall.jpg`. Khi mở trực tiếp, ảnh nhìn giống một ảnh trắng có rất nhiều chấm/nét đen nhỏ bị rải đều. Nếu nhìn bằng mắt thường thì không thấy chữ, không thấy QR code, cũng không thấy pattern rõ ràng.

Ảnh có kích thước:

```text
1386 x 424
```

Tổng số pixel:

```text
1386 * 424 = 587664
```

Điểm cần chú ý là đề nói ảnh "monochrome", nhưng đây là file JPEG nên các pixel không hoàn toàn chỉ có hai giá trị đen/trắng tuyệt đối. Do nén JPEG, các nét đen bị lan thành nhiều mức xám gần đen, còn nền trắng cũng có nhiều giá trị gần trắng. Vì vậy không nên xử lý ảnh như một bitstream nhị phân ngay từ đầu.

Kiểm tra sơ bộ cho thấy:

- Không có flag trực tiếp trong metadata.
- Không có chuỗi `LYKNCTF` trong file khi đọc strings thông thường.
- Không có dữ liệu append lộ rõ phía sau JPEG.
- Các hướng LSB/bitplane cơ bản không cho ra flag.

Vậy trọng tâm không phải là giấu dữ liệu trong bit thấp, mà là vị trí pixel đã bị hoán vị.

## 3. Phân tích hint

Hint 1:

```text
It's just a pixel permutation.
```

Nghĩa là giá trị màu của pixel không bị mã hóa phức tạp. Pixel vẫn là pixel thật của ảnh gốc, chỉ bị đưa sang vị trí khác. Nếu tìm đúng phép hoán vị ngược, ảnh ban đầu sẽ hiện ra.

Hint 2:

```text
Nearby pixels tend to stay nearby.
```

Hint này loại bớt các kiểu shuffle ngẫu nhiên. Nếu pixel bị xáo trộn hoàn toàn random thì các pixel gần nhau trong ảnh gốc sẽ bay đi rất xa nhau. Ở đây tác giả nói pixel gần nhau vẫn có xu hướng ở gần nhau, nên phép hoán vị phải là một dạng biến đổi giữ locality.

Những phép giữ locality thường gặp trong ảnh/pixel puzzle:

- Snake path / boustrophedon path
- Spiral path
- Diagonal scan
- Morton order / Z-order
- Hilbert curve
- Gilbert curve, tức biến thể Hilbert dùng tốt cho hình chữ nhật

Hint 3:

```text
Try following a path instead of rows.
```

Bình thường ảnh được đọc theo hàng: từ trái sang phải, hết hàng này xuống hàng tiếp theo. Đề nhắc "following a path" nghĩa là không nên đọc pixel theo hàng, mà nên đọc theo một đường đi qua toàn bộ ảnh.

Kết hợp các hint lại:

Ảnh gốc đã bị biến thành một chuỗi pixel theo một path giữ locality, sau đó chuỗi đó được ghi lại theo vị trí khác. Muốn khôi phục thì phải tìm đúng path và đưa chuỗi pixel về lại đúng thứ tự.

## 4. Các hướng thử nhưng không đúng

Trước khi ra lời giải, mình đã thử các hướng đơn giản trước để bám theo hint từ dễ đến khó.

Các hướng đã thử:

- Đọc ảnh theo hàng bình thường.
- Đảo chiều ảnh, xoay ảnh, transpose ảnh.
- Threshold ảnh ở nhiều mức đen/trắng khác nhau.
- Kiểm tra metadata, strings, EXIF, ICC profile.
- Kiểm tra bitplane, LSB, đóng gói bit thành bytes.
- Thử row snake và column snake.
- Thử spiral từ ngoài vào trong và từ trong ra ngoài.
- Thử diagonal/anti-diagonal scan.
- Thử reshape sang nhiều kích thước factor khác nhau.
- Thử Morton/Z-order.
- Thử Hilbert curve trên canvas vuông/padded.
- Thử các phép affine/cat-map đơn giản.

Các hướng này chỉ tạo ra texture hoặc nhiễu có cấu trúc nhẹ, không hiện chữ rõ ràng. Điều này cho thấy:

- Path không phải row/column/spiral đơn giản.
- Path có khả năng là một space-filling curve giữ locality tốt hơn.
- Ngoài path ra có thể còn một thao tác dịch chuỗi pixel.

## 5. Ý tưởng đúng: Gilbert curve

Hilbert curve là một space-filling curve rất hay gặp khi cần đi qua mọi điểm trong một vùng 2D mà vẫn giữ locality. Tuy nhiên ảnh challenge có kích thước `1386 x 424`, không phải hình vuông lũy thừa của 2. Vì vậy dùng Hilbert curve chuẩn trên hình vuông rồi crop/pad không khớp hoàn toàn.

Gilbert curve là một biến thể tương tự Hilbert curve nhưng dùng được trực tiếp trên hình chữ nhật bất kỳ. Đây là lựa chọn rất hợp với hint:

- Nó là một path đi qua mỗi pixel đúng một lần.
- Nó giữ pixel gần nhau tương đối gần nhau.
- Nó không đọc ảnh theo hàng.
- Nó là một pixel permutation thuần túy.

Sau khi sinh Gilbert path cho hình chữ nhật `1386 x 424`, ta đọc ảnh theo path đó:

```python
sequence = image[ys, xs]
```

Trong đó `ys, xs` là danh sách tọa độ pixel theo thứ tự Gilbert curve.

Nếu chỉ đọc và ghi lại theo path thì vẫn chưa hiện flag rõ. Bước còn thiếu là chuỗi pixel đã bị dịch vòng.

## 6. Vì sao cần dịch vòng chuỗi pixel

Giả sử ta có một ảnh text ban đầu, rồi lấy toàn bộ pixel theo Gilbert path thành một chuỗi dài:

```text
p0, p1, p2, p3, ..., p587663
```

Nếu tác giả đổi điểm bắt đầu trên path, chuỗi có thể bị xoay vòng:

```text
p363196, p363197, ..., p587663, p0, p1, ...
```

Đây vẫn là pixel permutation. Các pixel gần nhau trên path vẫn gần nhau, chỉ khác điểm bắt đầu. Khi ghi lại sai điểm bắt đầu, chữ trong ảnh sẽ bị cắt và rải thành nhiều mảnh, nhìn giống nhiễu.

Vì vậy cần tìm offset dịch vòng đúng.

Offset đúng tìm được là:

```text
363196
```

Trong script, bước này được làm bằng:

```python
np.roll(sequence, 363196)
```

## 7. Cách tìm offset đúng

Sau khi biết path có thể là Gilbert curve, ta có thể brute force hoặc search offset. Không cần OCR ngay từ đầu, chỉ cần một scoring đơn giản dựa trên cấu trúc ảnh.

Ý tưởng scoring:

1. Dịch chuỗi pixel theo một offset.
2. Ghi lại ảnh theo Gilbert path.
3. Threshold ảnh thành vùng đen/trắng.
4. Đếm số pixel đen trên từng hàng và từng cột.
5. Ảnh đúng dạng text sẽ có:
   - Một số hàng chứa rất nhiều nét chữ.
   - Một số hàng gần như trắng.
   - Projection theo hàng/cột không còn đều như noise.

Khi thử các path khác, score không nổi bật. Khi thử Gilbert curve, vùng offset quanh `363209` nổi lên rất mạnh. Sau đó refine quanh vùng này thì offset tốt nhất là:

```text
363196
```

Kết quả sau khi áp dụng offset này là ảnh hiện rõ text flag lặp lại nhiều lần.

## 8. Script solve

Script solve được lưu trong file:

```text
solve_one_stroke.py
```

Phần quan trọng nhất của script:

```python
SHIFT = 363196

image = np.array(Image.open("chall.jpg").convert("L"))
height, width = image.shape

path = np.array(list(gilbert2d(0, 0, width, 0, 0, height)), dtype=np.int32)
ys, xs = path[:, 0], path[:, 1]

sequence = image[ys, xs]
restored = np.empty_like(image)
restored[ys, xs] = np.roll(sequence, SHIFT)

Image.fromarray(restored).save("solved.png")
```

Giải thích:

- `Image.open("chall.jpg").convert("L")`: đọc ảnh và chuyển sang grayscale.
- `gilbert2d(0, 0, width, 0, 0, height)`: sinh path Gilbert đi qua toàn bộ hình chữ nhật.
- `ys, xs`: tọa độ các pixel theo thứ tự path.
- `sequence = image[ys, xs]`: lấy pixel theo path thay vì theo hàng.
- `np.roll(sequence, SHIFT)`: dịch vòng chuỗi pixel để đưa đúng điểm bắt đầu về vị trí đúng.
- `restored[ys, xs] = ...`: ghi pixel đã dịch lại theo cùng path.
- `solved.png`: ảnh đã khôi phục.

## 9. Reproduce

Chạy lệnh sau trong thư mục chứa challenge:

```powershell
python solve_one_stroke.py
```

Output:

```text
wrote solved.png
LYKNCTF{0548fcfa-6785-4051-a54d-8006b2f4828b}
```

Sau khi chạy, file `solved.png` được tạo ra. Mở ảnh này sẽ thấy flag lặp lại rõ ràng:

```text
LYKNCTF{0548fcfa-6785-4051-a54d-8006b2f4828b}
```

## 10. Tóm tắt lời giải

Challenge đánh lừa người chơi bằng một ảnh nhìn như nhiễu đơn sắc. Tuy nhiên giá trị pixel không bị mã hóa, chỉ bị đổi vị trí.

Các hint dẫn đến hướng giải:

- `pixel permutation`: chỉ cần đảo lại vị trí pixel.
- `nearby pixels tend to stay nearby`: dùng path giữ locality.
- `following a path instead of rows`: không đọc ảnh theo row-major order.

Path đúng là Gilbert curve trên hình chữ nhật `1386 x 424`. Sau khi đọc pixel theo Gilbert path, dịch vòng chuỗi pixel `363196` vị trí và ghi lại theo cùng path, ảnh gốc hiện ra.

## 11. Flag

```text
LYKNCTF{0548fcfa-6785-4051-a54d-8006b2f4828b}
```

## 12. Chú thích thuật ngữ

**Monochrome image**

Ảnh đơn sắc. Trong challenge này, ảnh gần như chỉ gồm nền trắng và nét đen. Tuy nhiên vì file được lưu dưới dạng JPEG nên pixel không hoàn toàn chỉ có đúng hai giá trị `0` và `255`; quanh nét đen/trắng sẽ có nhiều mức xám do nén ảnh.

**Pixel**

Đơn vị nhỏ nhất tạo nên ảnh. Mỗi pixel có một giá trị màu hoặc mức sáng. Với ảnh grayscale, mỗi pixel thường là một số từ `0` đến `255`, trong đó `0` là đen và `255` là trắng.

**Grayscale**

Ảnh mức xám. Thay vì dùng ba kênh màu RGB, mỗi pixel chỉ cần một giá trị độ sáng. Trong script solve, ảnh được chuyển sang grayscale bằng:

```python
Image.open("chall.jpg").convert("L")
```

**Pixel permutation**

Hoán vị pixel. Nghĩa là các giá trị pixel không bị mã hóa hay thay đổi nội dung, mà chỉ bị đổi vị trí. Nếu tìm đúng cách đưa từng pixel về vị trí ban đầu, ảnh gốc sẽ hiện ra.

**Locality**

Tính cục bộ. Nếu một phép biến đổi giữ locality, các pixel gần nhau trong ảnh gốc vẫn có xu hướng nằm gần nhau sau khi biến đổi. Hint `Nearby pixels tend to stay nearby` đang ám chỉ tính chất này.

**Path**

Một đường đi qua các pixel. Thay vì đọc ảnh theo từng hàng từ trái sang phải, ta có thể đọc ảnh theo một đường đặc biệt đi qua toàn bộ pixel đúng một lần.

**Row-major order**

Cách đọc ảnh thông thường: đọc hết hàng đầu tiên từ trái sang phải, sau đó xuống hàng tiếp theo, cứ như vậy cho đến hết ảnh.

**Snake path / Boustrophedon path**

Kiểu đọc ziczac theo hàng hoặc cột. Ví dụ hàng đầu đọc trái sang phải, hàng sau đọc phải sang trái, hàng tiếp theo lại đọc trái sang phải. Đây là một path đơn giản nhưng không phải path đúng của challenge này.

**Spiral path**

Đường xoắn ốc. Có thể đi từ ngoài vào trong hoặc từ trong ra ngoài. Đây cũng là một kiểu path giữ các pixel gần nhau tương đối gần nhau.

**Diagonal scan**

Cách đọc ảnh theo các đường chéo thay vì theo hàng ngang. Kiểu này hay xuất hiện trong xử lý ảnh hoặc một số bài stego, nhưng trong challenge này không phải hướng đúng.

**Morton order / Z-order**

Một cách sắp xếp điểm 2D thành chuỗi 1D bằng cách xen kẽ bit của tọa độ `x` và `y`. Morton order giữ locality ở mức tương đối, nhưng không tốt bằng Hilbert/Gilbert curve trong nhiều trường hợp.

**Space-filling curve**

Đường cong lấp đầy không gian. Trong ngữ cảnh ảnh số, có thể hiểu là một đường đi qua toàn bộ pixel của ảnh. Các curve như Hilbert hoặc Gilbert thường được dùng vì chúng giữ locality tốt.

**Hilbert curve**

Một loại space-filling curve nổi tiếng. Hilbert curve đi qua các điểm trong một vùng 2D theo cách giữ các điểm gần nhau vẫn khá gần nhau trên chuỗi 1D. Hilbert chuẩn thường thuận tiện nhất với hình vuông kích thước lũy thừa của 2.

**Gilbert curve**

Một biến thể thực dụng của Hilbert curve dùng được trực tiếp cho hình chữ nhật bất kỳ. Vì ảnh challenge có kích thước `1386 x 424`, Gilbert curve phù hợp hơn Hilbert curve chuẩn. Đây là path đúng trong lời giải.

**Cyclic shift / Dịch vòng**

Dịch một chuỗi theo kiểu phần bị đẩy ra cuối sẽ quay lại đầu, hoặc ngược lại. Ví dụ:

```text
abcdef
```

Dịch vòng sang phải 2 vị trí sẽ thành:

```text
efabcd
```

Trong lời giải, chuỗi pixel theo Gilbert path được dịch vòng `363196` vị trí.

**Offset**

Độ lệch. Trong bài này, offset là số vị trí cần dịch vòng chuỗi pixel để đưa ảnh về đúng điểm bắt đầu. Offset đúng là:

```text
363196
```

**`np.roll`**

Hàm của NumPy dùng để dịch vòng một mảng. Trong script:

```python
np.roll(sequence, SHIFT)
```

dùng để dịch vòng chuỗi pixel theo offset đã tìm được.

**Threshold**

Ngưỡng chuyển ảnh xám thành ảnh đen/trắng. Ví dụ nếu chọn threshold `128`, pixel nhỏ hơn `128` được coi là đen, pixel lớn hơn hoặc bằng `128` được coi là trắng. Threshold hữu ích khi muốn đo cấu trúc chữ hoặc vùng đen trong ảnh.

**Projection**

Trong xử lý ảnh, projection thường là phép đếm số pixel đen theo từng hàng hoặc từng cột. Nếu ảnh chứa text, một số hàng sẽ có nhiều pixel đen hơn hẳn, còn một số hàng gần như trống. Điều này giúp phát hiện candidate đúng khi brute force offset.

**Scoring**

Cách chấm điểm candidate tự động. Thay vì mở từng ảnh bằng mắt, ta tính một số đặc trưng như độ lệch số pixel đen theo hàng/cột. Candidate có score nổi bật sẽ được kiểm tra kỹ hơn.

**LSB**

Viết tắt của `Least Significant Bit`, tức bit thấp nhất của giá trị pixel. Nhiều bài stego giấu dữ liệu trong LSB, nhưng challenge này không đi theo hướng đó vì hint nói rõ đây là pixel permutation.

**Bitplane**

Một lớp bit của ảnh. Ví dụ với pixel 8-bit, có 8 bitplane từ bit thấp nhất đến bit cao nhất. Kiểm tra bitplane là một hướng stego phổ biến, nhưng trong bài này không cho ra flag.

**Metadata / EXIF**

Thông tin phụ gắn trong file ảnh, ví dụ loại camera, thời gian chụp, phần mềm tạo ảnh, GPS, thumbnail, comment. Một số challenge giấu flag trong metadata, nhưng file này không chứa flag theo hướng đó.

**JPEG compression**

Cơ chế nén ảnh JPEG. JPEG là nén mất dữ liệu, nên các cạnh đen/trắng có thể bị tạo thành nhiều mức xám xung quanh. Vì vậy ảnh nhìn là monochrome nhưng khi đọc pixel sẽ có nhiều giá trị khác nhau, không chỉ đen/trắng tuyệt đối.
