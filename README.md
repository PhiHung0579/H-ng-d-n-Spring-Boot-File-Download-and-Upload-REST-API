# Hướnng-dẫn-Spring-Boot-File-Download-and-Upload-REST-API
## **Tổng quan**

Dự án này là một ứng dụng Spring Boot được thiết kế để xử lý việc tải lên và tải xuống file sử dụng REST API. Ứng dụng cho phép người dùng tải lên các file (ví dụ: hình ảnh nhà hàng, thực đơn) và truy xuất chúng khi cần.

## **Cấu trúc**

- **FileService**: Quản lý các thao tác với file bao gồm lưu trữ và tải file.
- **RestaurantController**: Xử lý các yêu cầu HTTP cho việc tải lên và tải xuống file.
- **ResponseData**: Một lớp hỗ trợ để định dạng dữ liệu phản hồi.

## **Bắt đầu**

### **Yêu cầu**

- Java 8 trở lên
- Maven
- Spring Boot

### **Cấu hình**

1. **Tệp Application Properties**

   Tạo một tệp `application.properties`hoặc`application.yml` trong thư mục `src/main/resources` với nội dung sau:

   ```properties/yml
   fileUpload.rootPath=<đường_dẫn_đến_thư_mục_lưu_trữ>
   Thay <đường_dẫn_đến_thư_mục_lưu_trữ> bằng đường dẫn đến thư mục bạn muốn lưu trữ các file.
### **Sử dụng**
**1. Tải lên File**


Để tải lên file, gửi một yêu cầu POST đến /restaurant với file như một tham số form-data:

Sử dụng Postman để thử nghiệm:

Mở Postman.

Tạo một yêu cầu POST mới.

URL: http://localhost:8080/restaurant

Chuyển đến tab "Body".
Chọn "form-data".

Thêm một trường mới với tên là "file" và chọn loại là "File".

Chọn file bạn muốn tải lên.

Nhấn "Send".

**2. Tải xuống File**

Để tải xuống file, gửi một yêu cầu GET đến /restaurant/file/{filename}:

Sử dụng Postman để thử nghiệm:

Mở Postman.

Tạo một yêu cầu GET mới.

URL: http://localhost:8080/restaurant/file/{filename}

Thay {filename} bằng tên file bạn muốn tải xuống.

Nhấn "Send".

File sẽ được tải xuống và bạn có thể xem nó trong tab "Body".
## **Giải thích Mã**

### **FileService**

Lớp `FileService` triển khai giao diện `FileServiceImp` và cung cấp các phương thức để lưu trữ và tải file.

**Các thuộc tính:**

- `rootPath`: Thuộc tính cấu hình cho thư mục gốc nơi các file sẽ được lưu trữ.
- `root`: Đối tượng Path đại diện cho thư mục gốc.

**Các phương thức:**

- `init()`: Khởi tạo thư mục gốc. Nếu thư mục không tồn tại, sẽ tạo mới.
- `saveFile(MultipartFile file)`: Lưu trữ file được tải lên vào thư mục gốc. Trả về `true` nếu file được lưu thành công, ngược lại trả về `false`.
- `loadFile(String fileName)`: Tải file từ thư mục gốc. Trả về đối tượng `Resource` đại diện cho file nếu file tồn tại và có thể đọc, ngược lại trả về `null`.

### **RestaurantController**

Lớp `RestaurantController` xử lý các yêu cầu HTTP cho việc tải lên và tải xuống file.

**Các thuộc tính:**

- `fileServiceImp`: Đối tượng `FileServiceImp` được autowired để xử lý các thao tác với file.

**Các endpoint:**

- `POST /restaurant`: Xử lý việc tải lên file. Sử dụng phương thức `saveFile` của `FileService` để lưu trữ file tải lên. Trả về một đối tượng `ResponseEntity` chứa trạng thái thành công.
- `GET /restaurant/file/{filename}`: Xử lý việc tải xuống file. Sử dụng phương thức `loadFile` của `FileService` để tải file được yêu cầu. Trả về một đối tượng `ResponseEntity` chứa tài nguyên file với các header phù hợp cho việc tải xuống.

### **ResponseData**

Một lớp hỗ trợ đơn giản để định dạng dữ liệu phản hồi. Nó chứa một thuộc tính `data` để lưu trữ nội dung phản hồi.


## **Code FileService.java**
```FileService.java
   // FileService.java
package com.npphung.Osahaneat.service;

import com.npphung.Osahaneat.service.imp.FileServiceImp;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.Resource;
import org.springframework.core.io.UrlResource;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;

@Service
public class FileService implements FileServiceImp {
    @Value("${fileUpload.rootPath}")
    private String rootPath;
    private Path root;

    private void init(){
        try{
            root = Paths.get(rootPath);
            if(Files.notExists(root)){
                Files.createDirectories(root);
            }
        } catch (Exception e){
            System.out.println("Error create folder:" + e.getMessage());
        }
    }

    @Override
    public boolean saveFile(MultipartFile file) {
        try{
            init();
            Files.copy(file.getInputStream(), root.resolve(file.getOriginalFilename()), StandardCopyOption.REPLACE_EXISTING);
            return true;
        } catch(Exception e){
            System.out.println("Error save file:" + e.getMessage());
            return false;
        }
    }

    @Override
    public Resource loadFile(String fileName) {
        try{
            init();
            Path file = root.resolve(fileName);
            Resource resource = new UrlResource(file.toUri());
            if(resource.exists() || resource.isReadable()){
                return resource;
            }
        } catch (Exception e){
            System.out.println("Error upload file:" + e.getMessage());
        }
        return null;
    }
}





// RestaurantController.java
package com.npphung.Osahaneat.controller;

import com.npphung.Osahaneat.payload.ResponseData;
import com.npphung.Osahaneat.service.imp.FileServiceImp;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.Resource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

@RestController
@RequestMapping("/restaurant")
public class RestaurantController {
    @Autowired
    FileServiceImp fileServiceImp;

    @PostMapping
    public ResponseEntity<?> createRestaurant(@RequestParam MultipartFile file){
        ResponseData responseData = new ResponseData();
        boolean isSuccess = fileServiceImp.saveFile(file);
        responseData.setData(isSuccess);
        return new ResponseEntity<>(responseData, HttpStatus.OK);
    }

    @GetMapping("file/{filename:.+}")
    public ResponseEntity<?> getFileRestaurant(@PathVariable String filename){
        Resource resource = fileServiceImp.loadFile(filename);
        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + resource.getFilename() + "\"")
                .body(resource);
    }
}


// ResponseData.java
package com.npphung.Osahaneat.payload;

public class ResponseData {
    private Object data;

    public Object getData() {
        return data;
    }

    public void setData(Object data) {
        this.data = data;
    }
}


Lưu ý: Bạn cần tạo tệp FileServiceImp.java như sau:

// FileServiceImp.java
package com.npphung.Osahaneat.service.imp;

import org.springframework.core.io.Resource;
import org.springframework.web.multipart.MultipartFile;

public interface FileServiceImp {
    boolean saveFile(MultipartFile file);
    Resource loadFile(String fileName);
}


Sau khi hoàn thành các bước trên, bạn có thể sử dụng Postman để thử nghiệm việc tải lên và tải xuống file như đã hướng dẫn.Nhóm 10 chúc các bạn thành công !!!


 
