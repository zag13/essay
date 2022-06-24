# 在 Go 中使用 smtp.SendMail 发送邮件

> [Send an email in Go with smtp.SendMail](https://www.ardanlabs.com/blog/2013/06/send-email-in-go-with-smtpsendmail.html)

我想在发生严重异常时从我的 TraceLog 包中发送一封电子邮件。 幸运的是，Go 的标准库中有一个在 net 包中名为 smpt 的包。当您查看文档时，您就会想要离开了。

我花了 20 分钟研究如何使用这个包。 在与参数和错误作斗争之后，我想出了这个示例代码：

```
package main

import (
    "bytes"
    "fmt"
    "net/smtp"
    "runtime"
    "strings"
    "text/template"
)

func main() {
    SendEmail(
        "smtp.1and1.com",
        587,
        "username@domain.com",
        "password",
        []string{"me@domain.com"},
        "testing subject",
        "<html><body>Exception 1</body></html>Exception 1")
    }

func catchPanic(err *error, functionName string) {
    if r := recover(); r != nil {
        fmt.Printf("%s : PANIC Defered : %v\n", functionName, r)

        // Capture the stack trace
        buf := make([]byte, 10000)
        runtime.Stack(buf, false)

        fmt.Printf("%s : Stack Trace : %s", functionName, string(buf))

        if err != nil {
            *err = fmt.Errorf("%v", r)
        }
    } else if err != nil && *err != nil {
        fmt.Printf("%s : ERROR : %v\n", functionName, *err)

        // Capture the stack trace
        buf := make([]byte, 10000)
        runtime.Stack(buf, false)

        fmt.Printf("%s : Stack Trace : %s", functionName, string(buf))
    }
}

func SendEmail(host string, port int, userName string, password string, to []string, subject string, message string) (err error) {
    defer catchPanic(&err, "SendEmail")

    parameters := struct {
        From string
        To string
        Subject string
        Message string
    }{
        userName,
        strings.Join([]string(to), ","),
        subject,
        message,
    }

    buffer := new(bytes.Buffer)

    template := template.Must(template.New("emailTemplate").Parse(emailScript()))
    template.Execute(buffer, &parameters)

    auth := smtp.PlainAuth("", userName, password, host)

    err = smtp.SendMail(
        fmt.Sprintf("%s:%d", host, port),
        auth,
        userName,
        to,
        buffer.Bytes())

    return err
}

func emailScript() (script string) {
    return `From: {{.From}}<br />
To: {{.To}}<br />
Subject: {{.Subject}}<br />
MIME-version: 1.0<br />
Content-Type: text/html; charset=&quot;UTF-8&quot;<br />
<br />
{{.Message}}`
}
```

不必在每次调用时都重新创建 auth 变量。 可以创建一次并重复使用。 我添加了我的 catchPanic 函数，以便您可以看到在测试代码时返回的任何异常。

如果您查看电子邮件的原始来源，您将看到 message 参数的工作原理：

```
Return-Path:
Delivery-Date: Wed, 12 Jun 2013 20:34:59 -0400
Received: from mout.perfora.net (mout.perfora.net [X.X.X.X])
    by mx.perfora.net (node=mxus0) with ESMTP (Nemesis)
    id 0MZTCn-1V3EP13 for me@domain.com; Wed, 12 Jun 2013 20:34:58 -0400
Received: from localhost (c-50-143-31-151.hsd1.fl.comcast.net [X.X.X.X])
    by mrelay.perfora.net (node=mrus4) with ESMTP (Nemesis)
    id 0Mhi4R-1V0RuN48ot-00MTc6; Wed, 12 Jun 2013 20:34:58 -0400
From: username@domain.com
To: me@domain.com
Subject: testing subject
MIME-version: 1.0
Content-Type: text/html; charset="UTF-8"
Message-Id: <0mhi4r- data-blogger-escaped-mrelay.perfora.net="">
Date: Wed, 12 Jun 2013 20:34:56 -0400
Envelope-To: me@domain.com


<html><body>Exception 1</body></html>
```

一如既往，我希望这个示例程序可以节省您的时间并帮助你解决麻烦。