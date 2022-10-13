<img src="https://raw.githubusercontent.com/zakaria-chahboun/ZakiQtProjects/master/IMAGE1.png">

Genmessage is a message generator to easy translate your Go application.

- You only have to fill the `messages.toml` file.
- You can use variables as placeholers in your translation template.
- Fast! We don't use `text/template` pacakge in the exported file, We just use `fmt.Sprintf`.
- Clear functions parameters, No more ambiguity with maps!

## Installation
```bash
go install github.com/zakaria-chahboun/genmessage@latest
```
## Usage
You have to create a `messages.toml` file, example:

````toml
# messages.toml file

[[Messages]]
Code = "err_user_access_denied"
Status = "danger"
[Messages.Templates]
english = "Incorrect Username or Password! Try again."
arabic = "إسم المستخدم أو كلمة المرور غير صحيحة! حاول من جديد."

[[Messages]]
Code = "last_date_bill_pay"
Status = "info"
Variables = {date="datetime"}
[Messages.Templates]
english = "The last date for paying bills is {date}."
arabic = "آخر أجل لتستديد الفواتير هو {date}"

[[Messages]]
Code = "err_stock_limit_exceeded"
Status = "warning"
Variables = {name="string", quantity="int"}
[Messages.Templates]
english = "Stock limit exceeded! Only {quantity} left in stock {name}."
arabic = "تم تجاوز حد المخزون! لم يتبقى سوى {quantity} من مخزون {name}."
````

or just create it by `-init` tag
```bash
genmessage -init
```

## Generate go file
```bash
# you will export "messages.go" by default
genmessage 

# or export a custom file name: e.g "name.go"
genmessage -export name
```
The result will be like that:

```go
package messages

import (
	"fmt"
	"time"
)

type Message struct {
	Code    string
	Status  string
	Message string
}

/* new type */
type lang string

/* to store locally the currect languege used in app */
var currectLang lang

/* to set the currect languege used in app */
func SetCurrectLang(languege lang) {
	currectLang = languege
}

/* enum: Message.Code */
const (
	ErrUserAccessDenied   = "err_user_access_denied"
	LastDateBillPay       = "last_date_bill_pay"
	ErrStockLimitExceeded = "err_stock_limit_exceeded"
)

/* enum: Message.Status */
const (
	StatusDanger  = "danger"
	StatusInfo    = "info"
	StatusWarning = "warning"
)

/* enum: Message.Templates.{lang} */
const (
	LangEnglish lang = "english"
	LangArabic  lang = "arabic"
)

func CreateErrUserAccessDenied() (m *Message) {
	m = &Message{}
	m.Code = ErrUserAccessDenied
	m.Status = StatusDanger
	switch currectLang {
	case LangArabic:
		m.Message = fmt.Sprintf("إسم المستخدم أو كلمة المرور غير صحيحة! حاول من جديد.")
	case LangEnglish:
		m.Message = fmt.Sprintf("Incorrect Username or Password! Try again.")
	}
	return
}

func CreateLastDateBillPay(
	date time.Time,
) (m *Message) {
	m = &Message{}
	m.Code = LastDateBillPay
	m.Status = StatusInfo
	switch currectLang {
	case LangArabic:
		m.Message = fmt.Sprintf("آخر أجل لتستديد الفواتير هو %v", date.Format("2006-01-02 15:04:05"))
	case LangEnglish:
		m.Message = fmt.Sprintf("The last date for paying bills is %v.", date.Format("2006-01-02 15:04:05"))
	}
	return
}

func CreateErrStockLimitExceeded(
	name string,
	quantity int,
) (m *Message) {
	m = &Message{}
	m.Code = ErrStockLimitExceeded
	m.Status = StatusWarning
	switch currectLang {
	case LangArabic:
		m.Message = fmt.Sprintf("تم تجاوز حد المخزون! لم يتبقى سوى %d من مخزون %s.", quantity, name)
	case LangEnglish:
		m.Message = fmt.Sprintf("Stock limit exceeded! Only %d left in stock %s.", quantity, name)
	}
	return
}
```

In error case, You will have a pretty cool error messages *(thanks to [cute](https://github.com/zakaria-chahboun/cute) package)* 👀:

<img src="./screenshot/01.png" alter="error screenshot" width=700>


## Fields
* Required fields 👔:
  - Code
  - [Messages.Templates]
* Optional fields 🤷:
  - Status
  - Variables

In *[Messages.Templates]* there is no rule to create the name of languages fields. You can write any field name you want:
```toml
[Messages.Templates]
english = "...."
arabic = "...."

#or 

[Messages.Templates]
en = "...."
ar = "...."

#or

[Messages.Templates]
anglais = "...."
arabe = "...."

#or 🦊

[Messages.Templates]
lang1 = "...."
lang2 = "...."
lang3 = "...."
```

But the languages fields must be identical in all messages ⚠.

## Variables and Types
You can add Variables in your templates, These types are allowed:

`int`, `float`, `string`, `date`, `time`, `datetime`

Of course you can add these variables as placeholders in templates. You can call a variable many times in the template 👍🏻:

```toml
[[Messages]]
Code = "test"
Variables = {name="string"}
[Messages.Templates]
en = "My name is {name}, Can you call me {name} 👀?"
```

## Use the message for error handling 🔥
You can use the Message as an error. Just add another go file in the same location/package name. e.g `errors.go`:

```go
package messages

func (this *Message) Error() string {
	return this.Code + " : " + this.Message
}
```

Now you can easily return it like an error:

```go
package main

import (
  "fmt"
  "your-module/messages"
)

func main() {
  // choose a language
  messages.SetCurrectLang(messages.LangEnglish)
  
  err := login("username","123456")
  if err != nil {
    panic(err)
  }
}

func login(name, pass string) error {
  // a way of login
  // ....
  // error case
  return CreateErrUserAccessDenied()
}
```

-----
twitter: [@zaki_chahboun](https://twitter.com/Zaki_Chahboun)