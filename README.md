# FMCG-Sales-Project-SQL

In this project, I used SQL server to deal with the sales data from a FMCG brand. The work involved advanced join, union, subquery, update, case-when, rank over to answer the following question:


**- Sử dụng bảng bán hàng tháng 4 và 6 vừa tạo để tìm danh sách khách hàng có mua hàng trong tháng 4 nhưng không mua trong tháng 6 và ngược lại.**

**- So sánh số tiền mua hàng của từng khách hàng trong tháng 4 và tháng 6.**

**- Tính tổng số tiền tăng ròng của tháng 6 với tháng 4 của từng cửa hàng.**

**- Lấy ra tổng số tiền giao dịch của từng cửa hàng trong năm 2019 và xem nó chiếm bao nhiêu % so với tổng số tiền giao dịch của từng cửa hang trong bảng bán hang.**

**- Lấy ra số tiền giao dịch lớn nhất của từng nhân viên với từng cửa hàng trong năm 2019 xem chiếm tỷ trọng bao nhiêu trên tổng giao dịch của nhân viên đó trong cửa hàng trong năm 2019**

**- Tính tổng số tiền giao dịch của từng nhân viên với từng cửa hàng trong từng tháng năm 2019 xem chiếm tỷ trọng bao nhiêu trên tổng doanh thu của cửa hàng trong từng tháng đó của năm 2019**

**- Update bảng Discount cột MucChietKhau = 0.01 nếu MucToiThieu từ 0 > 5, MucChietKhau = 0.02 nếu MucToiThieu từ 6 > 10, MucChietKhau = 0.03 nếu MucToiThieu từ 11 > 30 còn lại mức chiết khấu là 0.05**

**- Lấy ra thông tin giao dịch có số tiền lớn nhất theo từng ngày và từng cửa hàng**

**- Lấy ra khách có số tiền lớn nhất theo từng ngày và từng cửa hàng**

Here are my SQL queries:

***1.	Sử dụng bảng bán hàng tháng 4 và 6 vừa tạo để tìm danh sách khách hàng có mua hàng trong tháng 4 nhưng không mua trong tháng 6 và ngược lại.***
```
SELECT *
INTO Portfolio..BANHANG_201904
FROM Portfolio..BANHANG
WHERE YEAR(TRANS_DATE) = 2019 AND MONTH(TRANS_DATE) = 04

SELECT *
INTO Portfolio..BANHANG_201906
FROM Portfolio..BANHANG
WHERE YEAR(TRANS_DATE) = 2019 AND MONTH(TRANS_DATE) = 06
```
```
SELECT A.CUS_ID
FROM Portfolio..BANHANG_201904 A
LEFT JOIN Portfolio..BANHANG_201906 B
	ON A.CUS_ID = B.CUS_ID
WHERE B.CUS_ID IS NULL
UNION
SELECT A.CUS_ID
FROM Portfolio..BANHANG_201906 A
LEFT JOIN Portfolio..BANHANG_201904 B
	ON A.CUS_ID = B.CUS_ID
WHERE B.CUS_ID IS NULL
```
![image](https://user-images.githubusercontent.com/118095331/217551731-72adac01-79df-4384-aa60-2c5f9a7a88cf.png)

***2.	So sánh số tiền mua hàng của từng khách hàng trong tháng 4 và tháng 6. ***

```
SELECT
	ISNULL(A.CUS_ID, B.CUS_ID) CUS_ID,
	ISNULL(A.TOTAL_AMT_04,0) TOTAL_AMT_04,
	ISNULL(B.TOTAL_AMT_06,0) TOTAL_AMT_06,
	(ISNULL(B.TOTAL_AMT_06,0) - ISNULL(A.TOTAL_AMT_04,0)) [NET 06vs04]
FROM
(
SELECT 
	CUS_ID, 
	SUM(LCY_AMT) TOTAL_AMT_04
FROM MCI_test..BANHANG_201904
GROUP BY CUS_ID
) A
FULL JOIN
(
SELECT 
	CUS_ID, 
	SUM(LCY_AMT) TOTAL_AMT_06
FROM MCI_test..BANHANG_201906
GROUP BY CUS_ID
) B
ON A.CUS_ID = B.CUS_ID
```
![image](https://user-images.githubusercontent.com/118095331/217551989-9def9b78-3ec6-4485-971f-f7e73e0dc80d.png)

***3.	Tính tổng số tiền tăng ròng của tháng 6 với tháng 4 của từng cửa hàng.***

```
SELECT
	ISNULL(A.STOREDID, B.STOREDID) STOREDID,
	ISNULL(A.TOTAL_AMT_04,0) TOTAL_AMT_04,
	ISNULL(B.TOTAL_AMT_06,0) TOTAL_AMT_06,
	(ISNULL(B.TOTAL_AMT_06,0) - ISNULL(A.TOTAL_AMT_04,0)) [NET 06vs04] 
FROM
(
SELECT 
	STOREDID, 
	SUM(LCY_AMT) TOTAL_AMT_04
FROM Portfolio..BANHANG_201904
GROUP BY STOREDID
) A
FULL JOIN
(
SELECT 
	STOREDID,  
	SUM(LCY_AMT) TOTAL_AMT_06
FROM Portfolio..BANHANG_201906
GROUP BY STOREDID
) B
ON A.STOREDID = B.STOREDID
```
![image](https://user-images.githubusercontent.com/118095331/217552346-805f9bb6-96d7-44b2-82ad-936ca89c71c4.png)

***4.	Lấy ra tổng số tiền giao dịch của từng cửa hàng trong năm 2019 và xem nó chiếm bao nhiêu % so với tổng số tiền giao dịch của từng cửa hang trong bảng bán hang.***

```
SELECT 
	ISNULL(A.STOREDID, B.STOREDID) STOREDID,
	ISNULL(A.AMT_2019,0) AMT_2019,
	ISNULL(B.TOTAL_AMT,0) TOTAL_AMT,
	CAST((ISNULL(A.AMT_2019,0)/B.TOTAL_AMT)*100 AS DECIMAL(10,2)) [% AMT_2019]
FROM
(
SELECT 
	STOREDID,
	SUM(LCY_AMT) AMT_2019
FROM Portfolio..BANHANG
WHERE YEAR(TRANS_DATE) = 2019
GROUP BY STOREDID
) A
FULL JOIN 
(
SELECT 
	STOREDID,
	SUM(LCY_AMT) TOTAL_AMT
FROM Portfolio..BANHANG
GROUP BY STOREDID
) B
ON A.STOREDID = B.STOREDID
```
![image](https://user-images.githubusercontent.com/118095331/217552555-d1756e40-3829-4ad9-85b8-9c56ad658b29.png)

***5.	Lấy ra số tiền giao dịch lớn nhất của từng nhân viên với từng cửa hàng trong năm 2019 xem chiếm tỷ trọng bao nhiêu trên tổng giao dịch của nhân viên đó trong cửa hàng trong năm 2019***

```
SELECT 
	ISNULL(A.SALE_ID, B.SALE_ID) SALE_ID,
	ISNULL(A.STOREDID, B.STOREDID) STOREDID,
	ISNULL(A.MAX_AMT,0) MAX_AMT,
	ISNULL(B.TOTAL_AMT,0) TOTAL_AMT,
	CAST((ISNULL(A.MAX_AMT,0)/B.TOTAL_AMT)*100 AS DECIMAL(10,2)) [% MAX_AMT]
FROM
(
SELECT
	STOREDID,
	SALE_ID,
	MAX(LCY_AMT) MAX_AMT
FROM Portfolio..BANHANG
WHERE YEAR(TRANS_DATE) = 2019
GROUP BY STOREDID, SALE_ID
) A
FULL JOIN 
(
SELECT
	STOREDID,
	SALE_ID,
	SUM(LCY_AMT) TOTAL_AMT
FROM Portfolio..BANHANG
WHERE YEAR(TRANS_DATE) = 2019
GROUP BY STOREDID, SALE_ID
) B
ON 
	A.SALE_ID = B.SALE_ID AND
	A.STOREDID = B.STOREDID
```
![image](https://user-images.githubusercontent.com/118095331/217552779-b08e54c2-8405-4c77-b02e-8dba1ec578d5.png)

***6.	Tính tổng số tiền giao dịch của từng nhân viên với từng cửa hàng trong từng tháng năm 2019 xem chiếm tỷ trọng bao nhiêu trên tổng doanh thu của cửa hàng trong từng tháng đó của năm 2019***

```
SELECT
	ISNULL(A.STOREDID,B.STOREDID) STOREDID,
	A.SALE_ID,
	ISNULL(A.[MONTH],B.[MONTH]) [MONTH],
	ISNULL(A.MONTH_AMT,0) MONTH_AMT,
	ISNULL(B.TOTAL_MONTH_AMT,0) TOTAL_MONTH_AMT,
	CAST((ISNULL(A.MONTH_AMT,0)/ISNULL(B.TOTAL_MONTH_AMT,0))*100 AS DECIMAL(10,4)) [% SALE_AMT]
FROM
(
SELECT
	STOREDID,
	SALE_ID,
	MONTH(TRANS_DATE) [MONTH],
	SUM(LCY_AMT) MONTH_AMT
FROM Portfolio..BANHANG
WHERE YEAR(TRANS_DATE) = 2019
GROUP BY
	STOREDID,
	SALE_ID,
	MONTH(TRANS_DATE)
) A
FULL JOIN
(
SELECT 
	STOREDID,
	MONTH(TRANS_DATE) [MONTH],
	SUM(LCY_AMT) TOTAL_MONTH_AMT
FROM Portfolio..BANHANG
WHERE YEAR(TRANS_DATE) = 2019
GROUP BY
	STOREDID,
	MONTH(TRANS_DATE)
) B
ON 
	A.STOREDID = B.STOREDID AND
	A.[MONTH] = B.[MONTH]
```
![image](https://user-images.githubusercontent.com/118095331/217553534-f9a7f0df-9f62-4de9-a8a3-9e29925ae51c.png)

***7.	Update bảng Discount cột MucChietKhau = 0.01 nếu MucToiThieu từ 0 > 5, MucChietKhau = 0.02 nếu MucToiThieu từ 6 > 10, MucChietKhau = 0.03 nếu MucToiThieu từ 11 > 30 còn lại mức chiết khấu là 0.05***

![image](https://user-images.githubusercontent.com/118095331/217553713-c6097342-4832-424d-bd87-b9b0514316e9.png)

```
UPDATE Portfolio..DISCOUNT 
SET MUC_CHIET_KHAU = 
	CASE
		WHEN MUC_TOI_THIEU BETWEEN 0 AND 5 THEN 0.01
		WHEN MUC_TOI_THIEU BETWEEN 6 AND 10 THEN 0.02
		WHEN MUC_TOI_THIEU BETWEEN 11 AND 30 THEN 0.03
		ELSE 0.05
	END
```

***8.  Lấy ra thông tin giao dịch có số tiền lớn nhất theo từng ngày và từng cửa hàng***

```
WITH T AS
(
SELECT 
	*,
	RANK () OVER (PARTITION BY STOREDID, TRANS_DATE ORDER BY LCY_AMT DESC) AS RNK
FROM Portfolio..BANHANG
)
SELECT 
	STOREDID,
	TRANS_DATE,
	CUS_ID,
	RETAIL_BILL,
	LCY_AMT
FROM T
WHERE RNK = 1
ORDER BY CAST(RIGHT(STOREDID,(LEN(STOREDID) - LEN('STORE '))) AS INT), TRANS_DATE
```
![image](https://user-images.githubusercontent.com/118095331/217555341-547c6dbd-115e-48b7-a6a9-80d88ce1f970.png)

***9.  Lấy ra khách có số tiền lớn nhất theo từng ngày và từng cửa hàng***
```
WITH T AS 
(
SELECT 
	STOREDID,
	TRANS_DATE,
	CUS_ID,
	SUM(LCY_AMT) [TOTAL AMOUNT],
	RANK () OVER (PARTITION BY STOREDID, TRANS_DATE ORDER BY SUM(LCY_AMT) DESC) AS RNK
FROM Portfolio..BANHANG
GROUP BY
	STOREDID,
	TRANS_DATE,
	CUS_ID
)
SELECT *
FROM T 
WHERE RNK = 1
ORDER BY CAST(RIGHT(STOREDID,(LEN(STOREDID) - LEN('STORE '))) AS INT), TRANS_DATE
```
![image](https://user-images.githubusercontent.com/118095331/217557599-56c4748e-df71-42ea-8921-3e1a52a117cd.png)
