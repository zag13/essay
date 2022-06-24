# Publish & Subscribe Messaging Pattern

Publish-Subscribe 是一种消息传递模式，用于在不同组件之间传递消息（这些组件知道彼此的身份）。

它类似于 Observer 行为设计模式。Observer 和 Publish-Subscribe 的基本设计都是让那些对 `Event Messages` 感兴趣的用户与通知者解耦(Observers or Publishers)。
这意味着您不必对直接发送到特定接收者的消息进行编程。

为此，称为 "message broker" 或 "event bus" 的中间人接收已发布的消息，然后 将它们路由给订阅者。

## Implementation

```
type Message struct {
    // Contents
}


type Subscription struct {
    ch chan<- Message
    
    Inbox chan Message
}

func (s *Subscription) Publish(msg Message) error {
    if _, ok := <-s.ch; !ok {
        return errors.New("Topic has been closed")
    }

    s.ch <- msg
    
    return nil
}
```

```
type Topic struct {
    Subscribers    []Session
    MessageHistory []Message
}

func (t *Topic) Subscribe(uid uint64) (Subscription, error) {
    // Get session and create one if it's the first
    
    // Add session to the Topic & MessageHistory
    
    // Create a subscription
}

func (t *Topic) Unsubscribe(Subscription) error {
    // Implementation
}

func (t *Topic) Delete() error {
    // Implementation
}
```

```
type User struct {
    ID uint64
    Name string
}

type Session struct {
    User User
    Timestamp time.Time
}
```

## Improvements

事件可以通过使用无堆栈 goroutine 以并行方式发布。

一旦收件箱已满，可以通过使用有缓冲的收件箱处理掉队的订阅者并停止发送事件来提高性能。
