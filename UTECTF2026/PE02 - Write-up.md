**UTECTF_WRITEUP: PE02**

**Tác giả:** Huy_120_YenLang
**Team:** K2_2H

**Cuộc thi:**  UTECTF  | Reverse Engineering 

---

## Mô tả challenge

![image](https://hackmd.io/_uploads/rycl9GYjle.png)

![image](https://hackmd.io/_uploads/H1YmqzYjex.png)


sau khi tải file main.exe và chạy thử ta thấy được chương trình bắt người dùng nhập vào **Flag:**, vậy chúng ta có thể đoán ra là chương trình có ẩn chứa gì đó để ta tìm ra cờ,  sau khi có cờ nhập vào để xét tính đúng sai


---

## Mục tiêu

* Tìm được flag đúng mà chương trình yêu cầu có dạng `UTECTF{...}`

---

## Công cụ & Môi trường

* OS: Kali/Ubuntu
* Công cụ: `IDA`

---

## Phân tích chi tiết (Step-by-step)

1. **Bước 1 — Thu thập thông tin**


```bash
kali@Huy:/mnt/d/ctf$ file program.exe
program.exe: PE32 executable (console) Intel 80386 (stripped to external PDB), for MS Windows, 8 sections
```
* file là Windows PE 32-bit, kiến trúc x86 (32-bit), vì vậy phù hợp để dùng công cụ IDA

2. **Bước 2 — Dịch ngược**
    
    * Ném vào ida và chạy thử xem sao 
    ![image](https://hackmd.io/_uploads/H1NOhfKjel.png)
    * Đây là những gì bạn nhận được :)))))

    * Vậy ta có thể đoán được người ra đề đã ngăn việc chương trình bị debug hãy tìm cách pass qua như ở **PE01** đã làm nhé
    * Khi nãy ta có thể thấy chương trình in ra thông báo **Flag:** sau đó thì nhận input từ người dùng, theo thói quen mình sẽ thử tìm xem nhưng chuỗi này nằm ở đâu nhé, có thể cũng sẽ có hàm xử lí chuỗi ở đó:
    * Nhấn tổ hợp phím **shift + f12** để có thể thấy tất cả các chuỗi có trong chương trình sau đó nhấn **ctrl + f** để search chuỗi 'Flag:'
    
    ![image](https://hackmd.io/_uploads/H1pR-XKilg.png)

    * Đây chính là hàm mà chúng ta cần xem xét, ta có thể dễ dàng thấy được hàm ``IsDebuggerPresent()``, tiếp tục làm tương tự như ở **PE01** để có thể tiếp tục đến các lệnh phía dưới nhé
    ![image](https://hackmd.io/_uploads/rJPbxmKogx.png)

   * Sau khi thực hiện xong và tiếp tục debug chương trình sẽ in chuỗi có nội dung là ''**Flag:**'' sau đó sẽ nhận input từ người dùng và lưu vào `dword_408020`:
   ![image](https://hackmd.io/_uploads/Sy-DWXFoeg.png)
   * Ở đây mình thử nhập chuỗi **abcd** và tiếp tục xem chương sẽ làm gì với chuỗi đó nhé
    ![image](https://hackmd.io/_uploads/SkaYAfKjex.png)

   * Sau đó chương trình tiếp tục thực hiện các lệnh mình chưa rõ để làm gì nhưng hãy cứ xem qua nhé:
   ![image](https://hackmd.io/_uploads/Skegg4Foxe.png)


     --  ` dword_4081E8 = (int)sub_401740;`: Gán con trỏ hàm sub_401740 vào dword_4081E8
    -- Tìm độ dài của một chuỗi trong bộ nhớ (byte_40173F) cho tới khi gặp ký tự -61 (0xC3)
    -- Gọi `sub_401780()`, hãy thử nhấn **f8** cho đến hàm này để xem nó làm gì nhé
    ![image](https://hackmd.io/_uploads/H1NOhfKjel.png)
    ![image](https://hackmd.io/_uploads/S1VHW4Yoge.png)

    * Tiếp tục là hàm anti-debug mà người ra đề đã cài vào để gây khó khăn cho việc tìm cờ hãy làm tương tự như lúc đầu nhé
    ![image](https://hackmd.io/_uploads/S1VCWVFoxg.png)
    * Sau khi đã pass qua rồi hãy quan sát:
    `v1 = dword_408020` là con trỏ tới chuỗi flag mà người         dùng nhập.

    *  `dword_4080D8 = 6 `→ có thể dùng làm offset hoặc         tham số cho hàm tiếp theo (sub_4015C0).

    * Vòng lặp copy input vào `dword_4050C0[]` từ chỉ số  thường là 0 tới chỉ số 35.

    * 36 ở đây là.... là chiều dài chuỗi flag tối đa mà chương trình kiểm tra
    * Tiếp hãy cùng xét tới hàm `sub_4015C0`:
    Hên quá ở đây ta không bị Rick Roll nữa :)))), nhưng đây cũng là hàm làm mình khó hiểu nhất nên mình sẽ nói kĩ ở đây
    
        ![image](https://hackmd.io/_uploads/B10OjNKsgg.png)
    * Ở đây ta thấy:
    -- `dword_4080DC` là biến đếm đệ quy, tăng 1 mỗi lần gọi hàm.

        -- `dword_4080E0[8 * a1 + a2] = v2;` → đánh dấu vị trí (a1,a2) đã xử lý bằng số thứ tự đệ quy.

        -- `dword_4080E0` trông giống bảng 8x8 dùng để ghi lại trạng thái.
    ![image](https://hackmd.io/_uploads/rJt72VKoex.png)

        
        -- `if ( dword_4080D8 * dword_4080D8 == v2 )` và ta đã biết ở trên `dword_4080D8 = 6`. Khi đạt tới 36 bước (6*6 = 36), hàm xử lý tiếp chuỗi flag, ở đâu ta cũng có thể nghi ngờ có liên quan đến việc xử lí input nhập vào khi số bước bằng với độ dài tối được copy từ input 
        
        -- Đây là quy trình **trộn và hoán vị**: Copy các giá trị từ `dword_4080E0` sang `dword_408040` với một số điều chỉnh **-1**. Kết quả là một mảng hoán vị chỉ số 0..35.
        -- Sau khi dump dữ liệu ở `dword_408040` ta được như sau
    
    ![image](https://hackmd.io/_uploads/Sk_68rFile.png)

        
    ![image](https://hackmd.io/_uploads/SygxTNFolg.png)
    
    * Ở đây được chia làm 2 phần đó là phần **thực hiện hoán vị**:
    ![image](https://hackmd.io/_uploads/BkEMyHFixl.png)
    Mình sẽ viết lại cho dễ hiểu hơn

    ![image](https://hackmd.io/_uploads/H1El1rKilx.png)

        
        + dword_4050C0 = input người dùng.

        + dword_40501C[]: là mảng tạm để chuỗi sau hoán vị 
        
        + dword_40501C[j] = dword_4050C0[v13] → xáo trộn input theo hoán vị trong dword_408040
    
    * Và phần **so sánh**:
    ![image](https://hackmd.io/_uploads/S13_grtogx.png)
    `dword_405020` : kết quả sau khi hoán vị của input.
    `dword_405160` : Sau khi dump trong IDA ta sẽ nhận được chuỗi đáng nghi có thể là cờ đã được hoán vị **"UtTlE3F4Th3kg0{C_<w_t3i3u1rgdk4_nn}n"**
    

    * Vậy ta có thể suy luận ra như sau: Chương trình yêu cầu ta nhập một chuỗi gồm 36 kí tự, chuỗi được nhập sau khi qua những bước hoán vị trên sẽ được so sánh với chuỗi **"UtTlE3F4Th3kg0{C_<w_t3i3u1rgdk4_nn}n"**. Vậy việc chúng ta cần làm chỉ là viết 1 script nhỏ nhầm hoán vị ngược lại để tìm flag đúng


3. **Bước 3 — Khai thác**

   * Sau khi có đủ dữ liệu ta có thể viết một script python để thực hiện hoán vị ngược lại để lấy được cờ
   ![image](https://hackmd.io/_uploads/Syv9QrFixl.png)

4. **Bước 4 — Lấy flag**

   * Dùng cờ trên nhập thử vào chương trình để kiểm tra

    ![image](https://hackmd.io/_uploads/HkN1VrFixe.png)

    * Vậy là đã lấy được cờ :v , đề rất nhiều số 36 nhé :))))


---






