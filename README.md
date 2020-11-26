# Cross check API documentation
## 1. Thuật ngữ chính:
CrossCheck: Hệ thống cung cấp API cho kiểm tra giữa các tài liệu với nhau

App: Hệ thống sử dụng CrossCheck

Storage: Nơi lưu trữ dữ liệu trong CrossCheck

Document: Một tài liệu, ví dụ như 1 một bản ghi trong app

Indexing: Quá trình đẩy dữ liệu vào storage 

Searching: Quá trình tìm kiếm trùng lặp nội dung của 1 tài liệu với dữ liệu trong storage


## 2. Mô tả flow:

**Index**: Mỗi khi có một Document được tạo mới, hoặc cập nhật, App sẽ đẩy nội dung của Document đó lên CrossCheck để lưu trữ trong storage.

**Search**: Khi App cần kiểm tra trùng lặp của 1 Document, App sẽ gửi yêu cầu kiểm tra lên (bao gồm nội dung của document) và CrossCheck sẽ kiểm tra trùng lặp với các tài liệu trong Storage và  trả về kết quả

## 3. Mô tả các API
### 3.1 Base
#### Authorization

Tất cả các API đều ở dạng REST API với format dữ liệu request/response là JSON

Mỗi App sẽ được cấp một API Key để xác thực.

Tất cả request từ App gửi lên CrossCheck sẽ phải chèn API key vào header Authorization với giá trị là "Bearer<dấu cách>API_KEY"

Ví dụ:

``` sh
curl —location —request GET '{{host}}/index/1234' \
—header 'Authorization: Bearer API_KEY' \
```

#### Response và mã lỗi

CrossCheck sẽ trả về HTTP status code cơ bản như sau

**200**: CrossCheck thực hiện hành động thành công

**400**: Có lỗi trong dữ liệu đẩy lên

**401**: Chưa xác thực, kiểm tra lại API gửi lên

**404**: Sai đường dẫn hoặc không tìm thấy bản ghi

Với code = **200** và **400**: response sẽ có cấu trúc như sau:

``` json
{
    "code": "SUCCESS",
    "description": "Thành công",
    "data":{}
}
```

Trong đó:

> **code**: Mã lỗi, có 2 mã lỗi cơ bản là **SUCCESS** (thành công) và **ERROR** (lỗi không xác định). Các mã lỗi khác sẽ được mô tả riêng trong từng API
> 
> **description**: Mô tả của mã lỗi
> 
> **data**: data trả về (nếu có), trường này có thể null
> 


### 3.2 Push Document

API sẽ thêm hoặc cập nhật một Document vào Storage

Mỗi Document yêu cầu một khóa chính là **id**, App phải đảm bảo id của các Document không được trùng

**Lưu ý**: Id chỉ có ký tự chữ (a-zA-Z), số (0-9) và dấu gạch dưới (_)

Nếu push document với cùng id thì CrossCheck sẽ thực hiện thao tác cập nhật chứ không tạo mới bản ghi

#### Request gửi lên

> POST: {{host}}/index

```bash
curl --location --request POST '{{host}}/index' \
--header 'Authorization: Bearer API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "id": "abcdddd",
    "title": "Tiêu đề của Document",
    "url": "http://app.com/abcdddd",
    "content": "What does the maybeRefresh() return? If false, it means that index is being refreshed by a different thread (which takes time for bigger indices). Until this refresh is finished, other threads simply have to live with the stale reader/searcher.",
    "meta": "anything"
}'

```

> **id (required)**: id của Document. Chỉ có ký tự chữ (a-zA-Z), số (0-9) và dấu gạch dưới (_)
> 
> **title (optional)**: Tiêu đề của Document. Nên có để hiển thị trong kết quả khi kiểm tra
> 
> **url (optional)**: Đường dẫn của Document. Nên có để hiển thị trong kết quả khi kiểm tra
> 
> **content (required)**: Nội dung của tài liệu
> 
> **meta (optional)**: thông tin bổ sung của tài liệu 



#### Response trả về

Ví dụ response trả về:
```json
{
    "code": "SUCCESS",
    "description": "SUCCESS",
    "data": null
}

```


### 3.3. Get Document

Lấy thông tin 1 Document trong Storage

#### Request

> GET: {{host}}/index/{{document_id}}

Trong đó document_id là id của Document

```bash
curl —location —request GET '{{host}}/index/1234' \
—header 'Authorization: Bearer API_KEY'
```

#### Response

Ví dụ response trả về

``` json
{
    "code": "SUCCESS",
    "description": "SUCCESS",
    "data": {
        "id": "1234",
        "title": "xin chao",
        "url": "xxxx",
        "content": "noi dung của tài liệu",
        "meta": "xxxx"
    }
}

```

### 3.4. DELETE Document

Xóa một document trong storage

#### Request

> DELETE: {{host}}/index/{{document_id}}

Trong đó document_id là id của Document

``` sh
curl —location —request DELETE '{{host}}/index/1234' \
—header 'Authorization: Bearer API_KEY'
```

#### Response

``` json
{
    "code": "SUCCESS",
    "description": "SUCCESS",
    "data": null
}
```


### 3.5. Search Document

Kiểm tra trùng lặp của một Document với dữ liệu đã có trong storage

#### Request

> POST: {{host}}/search

``` bash
curl --location --request POST '{{host}}/search' \
--header 'Content-Type: application/json' \
--data-raw '{
    "content": "nội dung của tài liệu cần kiểm tra",
    "documentId": "22"
}'

```

> **content (required)**: Nội dung của tài liệu. Yêu cầu tối thiểu 100 ký tự.
> 
> **documentId (optional)**: Id của tài liệu. Nếu tài liệu đã từng được index vào Storage, thì cần đẩy trường này lên để đảm bảo kết quả không chứa chính tài liệu đó.


#### Response

``` json
{
    "code": "SUCCESS",
    "description": "SUCCESS",
    "data": {
        "score": 90,
        "sentences": [
          {
            "startChar": 0,
            "endChar": 40,
            "content": "đây là văn bản kiểm tra",
            "score": 100,
            "similarities": [
                {
                  "score": 100,
                  "content": "đây là văn bản kiểm tra trong hệ thống",
                  "title": "tài liệu mẫu",
                  "id": "document_1",
                  "url": "https://app.com/1"
                },
                {
                  "score": 90,
                  "content": "đó là văn bản kiểm tra ",
                  "title": "tài liệu mẫu 2",
                  "id": "document_2",
                  "url": "https://app.com/2"
                }
            ]
          },
          {
            "startChar": 41,
            "endChar": 80,
            "content": "tích hợp kiểm tra nhóm giữa App và CrossCheck",
            "score": 80,
            "similarities": [
                {
                  "score": 80,
                  "content": "App và CrossCheck tích hợp kiểm tra nhóm với nhau",
                  "title": "tài liệu mẫu",
                  "id": "document_1",
                  "url": "https://app.com/1"
                }
            ]
          }
			]
    }
}

```

Mô tả:

> **data**: Nội dung kết quả trùng lặp
> 

Trong **data**:

> **score**:  Điểm trùng lặp của cả document
> 
> **sentences**: Các câu văn bị trùng lặp
> 

Mỗi phần từ trong **sentences** bao gồm:

> **startChar**: Vị trí của ký tự đầu tiên của câu văn trong cả Document
> 
> **endChar**: Vị trí của ký tự đầu tiên của câu văn trong cả Document
> 
> **content**: Nội dung của câu văn
> 
> **score**: Điểm trùng lặp của câu văn
> 
> **similarities**: Danh sách các câu trong Storage trùng lặp với câu hiện tại
> 

Mỗi phần tử trong **similarities** bao gồm:
> **score**:  Điểm trùng lặp của câu văn so với câu được đem đi kiểm tra
> 
> **content**: Nội dung của câu văn
> 
> **title**:  Tiêu đề của Document trong Storage
> 
>  **id**:  id của Document trong Storage
> 
>  **url**:  Đường dẫn của Document trong Storage

