# Dependency Injection trong Java

## Core Principle

Giả sử chúng ta có một ứng dụng sử dụng `EmailService` để gửi mail, mã nguồn sẽ như sau:

```java
package com.di.java.legacy;

public class EmailService {
    public void sendEmail(String message, String receiver){
        // Code logic để gửi mail
        System.out.println("Email sent to " + receiver + " with Message = " + message);
    }
}
```

Class `EmailService` có phương thức `sendEmail` với hai tham số là nội dung thư và địa chỉ người nhận. Ứng dụng sử dụng như sau:

```java
package com.di.java.legacy;

public class MyApplication {
    private EmailService email = new EmailService();

    public void processMessages(String msg, String receiver){
        // Thực hiện validate nội dung thư
        this.email.sendEmail(msg, receiver);
    }
}
```

Code client:

```java
package com.di.java.legacy;

public class ApplicationTest {
    public static void main(String[] args){
        MyApplication app = new MyApplication();
        app.processMessages("Hi Framgia", "framgia@framgia.com");
    }
}
```

### Hạn chế:

- `MyApplication` khởi tạo trực tiếp `EmailService`, khiến việc mở rộng khó khăn.
- Thêm chức năng như gửi SMS hoặc Facebook sẽ phải thay đổi nhiều class.
- Test trở nên khó khăn do object được khởi tạo cứng trong class.

### Giải pháp tạm thời - Constructor Injection:

```java
package com.di.java.legacy;

public class MyApplication {
    private EmailService email = null;

    public MyApplication(EmailService svc){
        this.email = svc;
    }

    public void processMessages(String msg, String receiver){
        this.email.sendEmail(msg, receiver);
    }
}
```

Vẫn chưa tối ưu vì client phải khởi tạo `EmailService`.

## Dependency Injection (DI)

### Nguyên tắc:

1. Service phải implement interface hoặc extend base class.
2. Class sử dụng chỉ tương tác qua interface.
3. Injector chịu trách nhiệm khởi tạo service và truyền vào class sử dụng.

### Bước 1: Interface `MessageService`

```java
package com.di.java.dependencyinjection.service;

public interface MessageService {
    public void sendMessage(String msg, String receiver);
}
```

### Bước 2: Implement Email và SMS Service

```java
package com.di.java.dependencyinjection.service;

public class EmailServiceImpl implements MessageService {
    public void sendMessage(String msg, String receiver){
        System.out.println("Email sent to " + receiver + " with Message = " + msg);
    }
}
```

```java
package com.di.java.dependencyinjection.service;

public class SMSServiceImpl implements MessageService {
    public void sendMessage(String msg, String receiver){
        System.out.println("SMS sent to " + receiver + " with Message = " + msg);
    }
}
```

### Bước 3: Interface `Consumer`

```java
package com.di.java.dependencyinjection.consumer;

public interface Consumer {
    public void processMessages(String msg, String receiver);
}
```

### Bước 4: Lớp `MyDIApplication` implement `Consumer`

```java
package com.di.java.dependencyinjection.consumer;

import com.di.java.dependencyinjection.service.MessageService;

public class MyDIApplication implements Consumer {
    private MessageService service;

    public MyDIApplication(MessageService svc){
        this.service = svc;
    }

    public void processMessages(String msg, String receiver){
        this.service.sendMessage(msg, receiver);
    }
}
```

### Bước 5: Interface `MessageServiceInjector`

```java
package com.di.java.dependencyinjection.injector;

import com.di.java.dependencyinjection.consumer.Consumer;

public interface MessageServiceInjector {
    public Consumer getConsumer();
}
```

### Bước 6: Implement Injector cho Email và SMS

```java
package com.di.java.dependencyinjection.injector;

import com.di.java.dependencyinjection.consumer.Consumer;
import com.di.java.dependencyinjection.consumer.MyDIApplication;
import com.di.java.dependencyinjection.service.EmailServiceImpl;

public class EmailServiceInjector implements MessageServiceInjector {
    public Consumer getConsumer() {
        return new MyDIApplication(new EmailServiceImpl());
    }
}
```

```java
package com.di.java.dependencyinjection.injector;

import com.di.java.dependencyinjection.consumer.Consumer;
import com.di.java.dependencyinjection.consumer.MyDIApplication;
import com.di.java.dependencyinjection.service.SMSServiceImpl;

public class SMSServiceInjector implements MessageServiceInjector {
    public Consumer getConsumer() {
        return new MyDIApplication(new SMSServiceImpl());
    }
}
```

### Bước 7: Client sử dụng DI

```java
package com.di.java.dependencyinjection.test;

import com.di.java.dependencyinjection.consumer.Consumer;
import com.di.java.dependencyinjection.injector.EmailServiceInjector;
import com.di.java.dependencyinjection.injector.MessageServiceInjector;
import com.di.java.dependencyinjection.injector.SMSServiceInjector;

public class MyMessageDITest {
    public static void main(String[] args) {
        String msg = "Hi Pankaj";
        String email = "pankaj@abc.com";
        String phone = "4088888888";
        MessageServiceInjector injector = null;
        Consumer app = null;

        // Gửi email
        injector = new EmailServiceInjector();
        app = injector.getConsumer();
        app.processMessages(msg, email);

        // Gửi SMS
        injector = new SMSServiceInjector();
        app = injector.getConsumer();
        app.processMessages(msg, phone);
    }
}
```

## Cách khác: Setter Injection

```java
package com.di.java.dependencyinjection.consumer;

import com.di.java.dependencyinjection.service.MessageService;

public class MyDIApplication implements Consumer {

    private MessageService service;

    public MyDIApplication(){}

    public void setService(MessageService service) {
        this.service = service;
    }

    public void processMessages(String msg, String receiver){
        this.service.sendMessage(msg, receiver);
    }
}
```

```java
package com.di.java.dependencyinjection.injector;

import com.di.java.dependencyinjection.consumer.Consumer;
import com.di.java.dependencyinjection.consumer.MyDIApplication;
import com.di.java.dependencyinjection.service.EmailServiceImpl;

public class EmailServiceInjector implements MessageServiceInjector {
    public Consumer getConsumer() {
        MyDIApplication app = new MyDIApplication();
        app.setService(new EmailServiceImpl());
        return app;
    }
}
```

## Tổng kết

### Ưu điểm

- Tách biệt trách nhiệm (Separation of Concerns)
- Cắt giảm code trùng lặp
- Dễ mở rộng chức năng
- Unit test dễ dàng nhờ mock object

### Nhược điểm

- Nếu lạm dụng sẽ khó bảo trì vì lỗi chỉ xuất hiện tại runtime.



# Aspect Oriented Programming (AOP) – Lập trình hướng khía cạnh

Aspect Oriented Programming (AOP) là một kỹ thuật lập trình nhằm phân tách chương trình thành các module riêng rẽ, không phụ thuộc nhau, giống như các phòng ban trong một công ty (phòng kỹ thuật, phòng kế toán, phòng kinh doanh…).
Mỗi phòng thực hiện một nhiệm vụ riêng biệt nhưng kết hợp lại để vận hành toàn bộ công ty.

## Các khái niệm cơ bản

- **Lát cắt (Aspect)**: Một chức năng được tách riêng, có thể “xen vào” các module khác để thực hiện một hành động mà không ảnh hưởng đến module đó.
- **Điểm cắt (Join point)**: Vị trí mà lát cắt sẽ được chèn vào module khác.

### Ví dụ minh họa

Trong một công ty, tất cả nhân viên cần quyết toán thuế. Nếu ai cũng tự làm sẽ rất mất thời gian. Vì vậy, phòng kế toán sẽ đảm nhiệm chức năng này cho toàn công ty. Khi luật thuế thay đổi, chỉ cần cập nhật lại ở phòng kế toán mà không ảnh hưởng đến các phòng khác.

Trong chương trình, khi muốn ghi log cho tất cả các method, thay vì thêm log vào từng method, ta có thể dùng AOP để tách riêng chức năng log thành một module và xác định điểm cắt là đầu hoặc cuối mỗi method.

## Ưu điểm của AOP

- **Thiết kế đơn giản**: Theo nguyên lý “You aren’t gonna need it (YAGNI)”, chỉ cài đặt khi thật sự cần thiết.
- **Mã nguồn trong sáng**: Mỗi module chỉ làm đúng một nhiệm vụ.
- **Giảm trùng lặp**: Tránh code tangling và scattering.
- **Tái sử dụng cao**.

## Nhược điểm của AOP

- **Trừu tượng cao**: Khái niệm AOP thường khá khó hiểu với người mới.
- **Luồng chương trình phức tạp**: Gây khó khăn trong việc debug hoặc theo dõi chương trình.

## Thuật ngữ trong AOP

Giả sử bạn muốn tách chức năng log trong chương trình, các khái niệm AOP có thể được giải thích như sau:

- **Core concerns**: Những chức năng chính của chương trình (các method cần log).
- **Crosscutting concerns**: Những chức năng phụ tách biệt (ví dụ: log).
- **Join point**: Một điểm trong chương trình nơi có thể chèn hành vi bổ sung.
- **Pointcut**: Cách xác định join point.
- **Advice**: Đoạn code xử lý chức năng phụ (ví dụ: log), sẽ được chạy tại join point.


