# Springboot mail



## 1.Maven Dependencies

### 1.1 Spring

Here is what we'll add for use in the plain vanilla Spring framework:

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
```



### 1.2 Spring Boot

And for Spring Boot:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
    <version>2.5.6</version>
</dependency>
```

The latest version is available in the [Maven Central](https://search.maven.org/classic/#search|ga|1|g%3A"org.springframework.boot" AND a%3A"spring-boot-starter-mail") repository.





## 2.Mail Server Properties

The interfaces and classes for Java mail support in the Spring framework are organized as follows:

1. **MailSender interface**: the top-level interface that provides basic functionality for sending simple emails
2. **JavaMailSender interface**: the subinterface of the above **MailSender**. It supports MIME messages and is mostly used in conjunction with the **MimeMessageHelper** class for the creation of a **MimeMessage**. It's recommended to use the **MimeMessagePreparator** mechanism with this interface.
3. **JavaMailSenderImpl class** provides an implementation of the **JavaMailSender** interface. It supports the **MimeMessage** and **SimpleMailMessage**.
4. **SimpleMailMessage class**: used to create a simple mail message including the from, to, cc, subject and text fields
5. **MimeMessagePreparator interface** provides a callback interface for the preparation of MIME messages.
6. **MimeMessageHelper class**: helper class for the creation of MIME messages. It offers support for images, typical mail attachments and text content in an HTML layout.

In the following sections, we show how to use these interfaces and classes.



### 2.1 Spring Mail Server Properties

Mail properties that are needed to specify, for example, the SMTP server may be defined using the *JavaMailSenderImpl*.

For Gmail, this can be configured as shown below:

```java
@Bean
public JavaMailSender getJavaMailSender() {
    JavaMailSenderImpl mailSender = new JavaMailSenderImpl();
    mailSender.setHost("smtp.gmail.com");
    mailSender.setPort(587);
    
    mailSender.setUsername("my.gmail@gmail.com");
    mailSender.setPassword("password");
    
    Properties props = mailSender.getJavaMailProperties();
    props.put("mail.transport.protocol", "smtp");
    props.put("mail.smtp.auth", "true");
    props.put("mail.smtp.starttls.enable", "true");
    props.put("mail.debug", "true");
    
    return mailSender;
}
```



### 2.2 Spring Boot Mail Server Properties

Once the dependency is in place, the next step is to specify the mail server properties in the ***application.properties*** file using the **spring.mail.*** namespace.

We can specify the properties for the Gmail SMTP server this way:

```properties
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=<login user to smtp server>
spring.mail.password=<login password to smtp server>
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
```

Some SMTP servers require a TLS connection, so we use the property *spring.mail.properties.mail.smtp.starttls.enable* to enable a TLS-protected connection.



#### 2.2.1. Gmail SMTP Properties

We can send an email via Gmail SMTP server. Have a look at the [documentation](https://support.google.com/mail/answer/13273?hl=en&rd=2) to see the Gmail outgoing mail SMTP server properties.

Our *application.properties* file is already configured to use Gmail SMTP (see the previous section).

Note that the password for our account should not be an ordinary password but an application password generated for our Google account. Follow this [link](https://support.google.com/accounts/answer/185833) to see the details and to generate your Google App Password.



#### 2.2.2. SES SMTP Properties

To send emails using Amazon SES, we set our *application.properties*:

```properties
spring.mail.host=email-smtp.us-west-2.amazonaws.com
spring.mail.username=username
spring.mail.password=password
spring.mail.properties.mail.transport.protocol=smtp
spring.mail.properties.mail.smtp.port=25
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
```

Please be aware that Amazon requires us to verify our credentials before using them. Follow the [link](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/verify-email-addresses.html) to verify your username and password.



## 3. Sending Email

Once dependency management and configuration are in place, we can use the aforementioned ***JavaMailSender*** to send an email.

Since both the plain vanilla Spring framework as well as the Boot version of it handle the composing and sending of emails in a similar way, we won't have to distinguish between the two in the subsections below.

### 3.1. Sending Simple Emails

Let's first compose and send a simple email message without any attachments:

```java
@Component
public class EmailServiceImpl implements EmailService {

    @Autowired
    private JavaMailSender emailSender;

    public void sendSimpleMessage(
      String to, String subject, String text) {
        ...
        SimpleMailMessage message = new SimpleMailMessage(); 
        message.setFrom("noreply@baeldung.com");
        message.setTo(to); 
        message.setSubject(subject); 
        message.setText(text);
        emailSender.send(message);
        ...
    }
}
```



Note that even though it's not mandatory to provide the *from* address, many SMTP servers would reject such messages. That's why we use the noreply@baeldung.com email address in our *EmailService* implementation.

请注意，即使不强制提供发件人地址，许多 SMTP 服务器也会拒绝此类邮件。这就是我们在 EmailService 实现中使用 noreply@baeldung.com 电子邮件地址的原因。



### 3.2. Sending Emails With Attachments

Sometimes Spring's simple messaging is not enough for our use cases.

For example, we want to send an order confirmation email with an invoice attached. In this case, we should use a *MIME* multipart message from *JavaMail* library instead of *SimpleMailMessage*. Spring supports *JavaMail* messaging with the *org.springframework.mail.javamail.MimeMessageHelper* class.

First of all, we'll add a method to the *EmailServiceImpl* to send emails with attachments:

3.2.发送带附件的电子邮件
有时 Spring 的简单消息传递对于我们的用例来说是不够的。

例如，我们要发送附有发票的订单确认电子邮件。在这种情况下，我们应该使用来自 JavaMail 库的 MIME 多部分消息，而不是 SimpleMailMessage。 Spring 使用 org.springframework.mail.javamail.MimeMessageHelper 类支持 JavaMail 消息传递。

首先，我们将向 EmailServiceImpl 添加一个方法来发送带有附件的电子邮件：

```java
@Override
public void sendMessageWithAttachment(
  String to, String subject, String text, String pathToAttachment) {
    // ...
    
    MimeMessage message = emailSender.createMimeMessage();
     
    MimeMessageHelper helper = new MimeMessageHelper(message, true);
    
    helper.setFrom("noreply@baeldung.com");
    helper.setTo(to);
    helper.setSubject(subject);
    helper.setText(text);
        
    FileSystemResource file 
      = new FileSystemResource(new File(pathToAttachment));
    helper.addAttachment("Invoice", file);

    emailSender.send(message);
    // ...
}
```





### 3.3. Simple Email Template

*SimpleMailMessage* class supports text formatting.

We can create a template for emails by defining a template bean in our configuration:

```java
@Bean
public SimpleMailMessage templateSimpleMessage() {
    SimpleMailMessage message = new SimpleMailMessage();
    message.setText(
      "This is the test email template for your email:\n%s\n");
    return message;
}
```

Now we can use this bean as a template for email and only need to provide the necessary parameters to the template:

```java
@Autowired
public SimpleMailMessage template;
...
String text = String.format(template.getText(), templateArgs);  
sendSimpleMessage(to, subject, text);
```



## 4. Handling Send Errors

*JavaMail* provides *SendFailedException* to handle situations when a message cannot be sent. But it is possible that we won't get this exception while sending an email to the incorrect address. The reason is the following:

The protocol specs for SMTP in RFC 821 specifies the 550 return code that the SMTP server should return when attempting to send an email to the incorrect address. But most of the public SMTP servers don't do this. Instead, they send a “delivery failed” email or give no feedback at all.

For example, Gmail SMTP server sends a “delivery failed” message. And we get no exceptions in our program.

So, we have a few options to handle this case:

1. Catch the *SendFailedException*, which can never be thrown.
2. Check our sender mailbox for the “delivery failed” message for some period of time. This is not straightforward, and the time period is not determined.
3. If our mail server gives no feedback at all, we can do nothing.



