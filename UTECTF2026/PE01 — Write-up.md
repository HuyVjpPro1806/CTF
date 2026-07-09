**UTECTF_WRITEUP: PE01**

**Tác giả:** Huy36
**Team:** K2_2H

**Cuộc thi:**  UTECTF  | Reverse Engineering 

---

## Mô tả challenge

![image](https://hackmd.io/_uploads/HyilbD8jxe.png)
![image](https://hackmd.io/_uploads/S13JGPUsel.png)

sau khi tải file main.exe và chạy thử ta thấy được chương trình bắt người dùng nhập vào 2 key, vậy công việc của chúng ta có thể đoán ra là tìm 2 key đúng để in được cờ


---

## Mục tiêu

* Tìm được 2 key chương trình yêu cầu để có thể nhận được flag có dạng `UTECTF{...}`

---

## Công cụ & Môi trường

* OS: Kali
* Công cụ: `IDA`

---

## Phân tích chi tiết (Step-by-step)

1. **Bước 1 — Thu thập thông tin**


```bash
kali@Huy:/mnt/d/ctf$ file main.exe
main.exe: PE32 executable (console) Intel 80386, for MS Windows, 13 sections
```
file là Windows PE 32-bit, kiến trúc x86 (32-bit), vì vậy phù hợp để dùng công cụ ida

2. **Bước 2 — Dịch ngược**
    
    * Ném vào ida ta dễ dàng thấy ngay tại hàm `init()`
    ![image](https://hackmd.io/_uploads/Byzz2YUixl.png)
    - Ở hàm này ta thấy người ra đề đã ngăn chặn việc chương     trình bị debug bằng cách dùng biến `result` để lưu giá     trị trả về từ hàm `IsDebuggerPresent()`
    - nếu phát hiện debug chương trình sẽ nhảy vào `main2()` (là giao diện giả nhầm đánh lừa) rồi tiếp tục nhảy vào `exit()` để kết thúc chương trình
    ![image](https://hackmd.io/_uploads/ByL9TF8sex.png)
    - Việc ta cần làm lúc này là đặt breakpoint ngay tại `test eax,eax` và thay đổi giá trị thanh ghi eax thành 0x0h
    ![image](https://hackmd.io/_uploads/ByHr0FIogl.png)



   * Sau khi thực hiện xong và tiếp tục debug chương trình sẽ nhảy đến hàm `main` thật sự, quan sát kĩ ta thấy:
   ![image](https://hackmd.io/_uploads/ryakPvLslx.png)
   chương trình in ra thông báo, nhận input từ người dùng và lưu vào key1 và key2, sau đó sẽ gọi hàm `check()`
   ![image](https://hackmd.io/_uploads/HkyYy9Lolx.png)
    -Ở đây mình thử nhập key1 là "123" và key2 là "456", cùng xem tiếp 2 key này sẽ được chương trình xử lí tiếp như thế nào
   * kiểm tra hàm `check()`:
    -- Hàm check nhận dữ liệu từ các biến toàn cục: ``a, b, c, aa, bb, cc, key1, key``
    -- Sử dụng các toán tử nạp chồng: ``operator*, operator+, operator-, operator==.``, sau khi xem thì đây chính là các hàm tính toán số lớn
    * Giờ tìm hiểu kĩ thêm hàm ``check`` sẽ làm những gì, trên ida cho phép chuyển sang mã giả C để dễ hiểu hơn:
    **a/**
    * Chương trình thực hiện việc copy qua các lệnh dưới hình, ở đây là v119 copy từ key2:
    ![image](https://hackmd.io/_uploads/S1o-7o8jeg.png)
    
    * sau đó chương trình dùng các toán tử được nạp chồng để bắt đầu tính toán
    ![image](https://hackmd.io/_uploads/SJrliKIogl.png)

        ``operator*(&v122, &v116, &v119);``
    -- v116 copy từ b 
    -- v119 copy từ key2.
    → v122= (b * key2) 
    
    ![image](https://hackmd.io/_uploads/ByKLcYLigx.png)

         ``operator*(&v113, &v107, &v110);``
         ``operator+(&v89, &v113, &v122);``
    -- v107 copy từ a
    -- v110 copy từ key1
    → v113 = a * key1
    → v89 = (a * key1) + (b * key2)
    
    **b/**
    ![image](https://hackmd.io/_uploads/rkUhWqUsex.png)

    ``operator*(v106, &v100, &v103);   // bb * key2``
    ![image](https://hackmd.io/_uploads/rk5zz9Usgg.png)

    ``operator*(v99, &v93, &v96);      // aa * key1``
    ``operator-(&v91, v99, v106);  //→ v91 = (aa * key1) - (bb * key2)``
    * Sau hai thao tác từ **a** và **b** chương trình sẽ đến phần so sánh quan trọng:
    ``v62 = operator==(&v107, &v110);   // (c == v89)``
    **c= a * key1 + b * key2**
    ``v62 = operator==(&v113, &v116);   // (cc == v91)``
    **cc= aa * key1 − bb * key2**
    
    * Vậy để in được kết quả ``Correct!`` và in được cờ ta phải thỏa cả hai phương trình trên, việc bây giờ bạn cần làm chỉ là dump dữ liệu từ các mảng **a, aa, b, bb và c, cc.**. Ở đây mình sẽ ví dụ cho việc dump mảng **a**
    * Double click vào biến a ta sẽ thấy : 
![image](https://hackmd.io/_uploads/r1VJc5Loel.png)
    Mảng a bắt đầu từ `8D71A0h` và kết thúc ở `8D721Ch`,tiếp tục double click vào ta sẽ thấy được dữ liệu trong mảng a
![image](https://hackmd.io/_uploads/rkv4Y5Uogl.png)
thực hiện tương tự với các mảng còn lại ta sẽ có đủ dữ liệu để tính toán được key1 và key 2
 



3. **Bước 3 — Khai thác**

   * Sau khi có đủ dữ liệu ta có thể viết một script python để giải hệ phương trình trên
   ![image](https://hackmd.io/_uploads/Sy19WuLoll.png)



4. **Bước 4 — Lấy flag**

   * Dùng 2 key trên vào chương trình và chờ đợi kết quả

    ![image](https://hackmd.io/_uploads/SkHQGOLixl.png)
    * Vậy là đã lấy được cờ :v 


---






