# LYKN CTF - Writeup Truy Vết Giao Dịch USDT

## Thông Tin Bài

Challenge cung cấp transaction hash ban đầu:

```text
d4500023a8114caaa640ab92bb8f73830a5303ccdfc4e9b0cf862bdae7ae336b
```

Yêu cầu của bài:

- Truy vết chuỗi chuyển tiền USDT qua nhiều ví.
- Xác định transaction hash của hop cuối cùng còn có thể truy vết được.
- Xác định ngày giao dịch theo định dạng `MM/DD/YYYY`.
- Tìm tên thực thể bị trừng phạt đứng sau hoạt động này.
- Định dạng flag:

```text
LYKNCTF{tx_hash:MM/DD/YYYY:ENTITY}
```

Flag đúng:

```text
LYKNCTF{7e401f8004084d4bf9f792535fdf5b89138a935d027b6b75ceb2dd3ac8838fab:03/21/2025:FUNNULL TECHNOLOGY INC}
```


## 1. Xác Định Chain Và Token


<img width="1500" height="1100" alt="image" src="https://github.com/user-attachments/assets/3c9ac413-3966-417d-9845-5de61d277f51" />


Thông tin cần ghi lại:

| Trường | Giá trị |
|---|---|
| Chain | TRON |
| Token | USDT TRC-20 |
| USDT contract | `TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t` |
| From | `TXk7Dor9GeRRpR5hbCGd4rBieM21v4BcwX` |
| To | `TNmRfnSUXZoWWzxcDDbf95eGQYXt1mJDt8` |
| Amount | `2700 USDT` |
| Date | `02/27/2025` |

Giao dịch đầu tiên:

```text
TXk7Dor9GeRRpR5hbCGd4rBieM21v4BcwX
  -> TNmRfnSUXZoWWzxcDDbf95eGQYXt1mJDt8
Amount: 2700 USDT
Tx: d4500023a8114caaa640ab92bb8f73830a5303ccdfc4e9b0cf862bdae7ae336b
```

Nhận xét:

- Đây là giao dịch USDT trên mạng TRON.
- Token contract `TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t` là USDT TRC-20.
- Ví nhận đầu tiên là `TNmRfnSUXZoWWzxcDDbf95eGQYXt1mJDt8`.

## 2. Truy Vết Ví Nhận Đầu Tiên

Sau giao dịch đầu tiên, tiền đi vào ví:

```text
TNmRfnSUXZoWWzxcDDbf95eGQYXt1mJDt8
```

Mở địa chỉ này hoặc xem phần transfer liên quan trong transaction. Ta cần tìm các giao dịch USDT sau thời điểm `02/27/2025`.

Các giao dịch quan trọng:
<img width="1677" height="176" alt="image" src="https://github.com/user-attachments/assets/bc132a63-0a2d-4887-97a7-004eb67b57e6" />


```text
d4500023...e336b
TXk7...BcwX -> TNmR...JDt8
2700 USDT
02/27/2025

5d1c353f...8181
TPfT...TX9 -> TNmR...JDt8
2522 USDT
03/07/2025

2ef09557...b9d9
TNmR...JDt8 -> TQMq9s...3tb
5222 USDT
03/21/2025
```

Nhận xét:

- Ví `TNmR...` nhận `2700 USDT` từ transaction ban đầu của challenge.
- Sau đó ví này nhận thêm `2522 USDT`.
- Tổng hai khoản là `2700 + 2522 = 5222 USDT`.
- Đến ngày `03/21/2025`, ví này chuyển ra đúng `5222 USDT`.

Vì vậy, hop tiếp theo của chuỗi laundering là:

```text
Tx: 2ef09557180070d4bfd274f771619b062fa9a1dec5087869b45e65003256b9d9
From: TNmRfnSUXZoWWzxcDDbf95eGQYXt1mJDt8
To:   TQMq9s5eqxzHW9CG4hgrWxVZaz4oZDo3tb
Amount: 5222 USDT
Date: 03/21/2025
```


<img width="1500" height="1100" alt="image" src="https://github.com/user-attachments/assets/56eab1ab-bcb7-48a6-84be-19e23383a6e8" />


## 3. Truy Vết Ví TQMq9s...

Tiếp tục kiểm tra ví nhận:

```text
TQMq9s5eqxzHW9CG4hgrWxVZaz4oZDo3tb
```

Ví này nhận `5222 USDT` rồi chuyển đi gần như ngay lập tức:
<img width="1650" height="196" alt="image" src="https://github.com/user-attachments/assets/62f72b6e-3ad7-459a-ad60-1e1078f132d6" />


```text
2ef09557180070d4bfd274f771619b062fa9a1dec5087869b45e65003256b9d9
TNmR...JDt8 -> TQMq9s...3tb
5222 USDT

7e401f8004084d4bf9f792535fdf5b89138a935d027b6b75ceb2dd3ac8838fab
TQMq9s...3tb -> TJ7hh...Sdb
5222 USDT
```

Hop tiếp theo:

```text
Tx: 7e401f8004084d4bf9f792535fdf5b89138a935d027b6b75ceb2dd3ac8838fab
From: TQMq9s5eqxzHW9CG4hgrWxVZaz4oZDo3tb
To:   TJ7hhYhVhaxNx6BPyq7yFpqZrQULL3JSdb
Amount: 5222 USDT
Date: 03/21/2025
```


<img width="1500" height="1100" alt="image" src="https://github.com/user-attachments/assets/fa76343d-118e-43a6-bec5-721038ff00f8" />


## 4. Vì Sao Đây Là Hop Cuối Còn Truy Vết Được

Trong transaction `7e401f...`, địa chỉ nhận là:

```text
TJ7hhYhVhaxNx6BPyq7yFpqZrQULL3JSdb
```

Địa chỉ này được Tronscan gắn nhãn:

```text
Bitget 9
```

Trong ảnh OKLink của hop cuối có thể thấy giao dịch `7e401f...`, ngày `03/21/2025`, amount `5,222 USDT`, cùng nhãn Bitget ở phần from/to. Trên Tronscan, địa chỉ nhận tương ứng được gắn nhãn `Bitget 9`.

<img width="1500" height="1100" alt="image" src="https://github.com/user-attachments/assets/ce259124-24b1-41d0-ae31-bc50618d392a" />


Sau khi tiền đi vào ví sàn/high-volume exchange wallet:

- Cùng một địa chỉ nhận rất nhiều khoản deposit từ nhiều nguồn khác nhau.
- Cùng thời điểm có nhiều outgoing transaction đến nhiều địa chỉ khác.
- Không thể gán chắc chắn một outgoing cụ thể cho khoản deposit `5222 USDT` nếu không có dữ liệu nội bộ của sàn.

Vì vậy, hop cuối cùng còn có thể truy vết on-chain là:

```text
7e401f8004084d4bf9f792535fdf5b89138a935d027b6b75ceb2dd3ac8838fab
```

Ngày của hop này:

```text
03/21/2025
```

## 5. Xác Định Sanctioned Entity Bằng OFAC

Để tránh đoán sai entity, cần đối chiếu các địa chỉ trong chain với OFAC Sanctions List Search.

Mở trang chi tiết của bản ghi FUNNULL trên OFAC Sanctions List Search. Ảnh chụp trang OFAC:

<img width="1400" height="1000" alt="image" src="https://github.com/user-attachments/assets/e15b6850-94e6-4300-a9c1-bb2b76c1f60f" />


Trang OFAC cho thấy:

| Trường | Giá trị |
|---|---|
| Type | Entity |
| List | SDN |
| Entity Name | `FUNNULL TECHNOLOGY INC` |
| Program | `CYBER3` |
| Digital Currency Address - TRX | `TNmRfnSUXZoWWzxcDDbf95eGQYXt1mJDt8` |
| Aliases | `FUNNULL`, `FANG NENG CDN`, `FUNNULL LLC`, `FUNNULL INC`, `FUNNULL CDN` |

Địa chỉ TRX trên OFAC:

```text
TNmRfnSUXZoWWzxcDDbf95eGQYXt1mJDt8
```

chính là địa chỉ nhận của transaction challenge ban đầu. Vì vậy, sanctioned entity cần dùng trong flag là:

```text
FUNNULL TECHNOLOGY INC
```
