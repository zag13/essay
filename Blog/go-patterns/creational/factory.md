# Factory Method Pattern

Factory method 设计模式可以创建一个不必指定确切类型的对象。

## Implementation

示例展示了如何提供具有不同后端的数据存储，例如内存、磁盘存储.

### Types

```
package data

import "io"

type Store interface {
    Open(string) (io.ReadWriteCloser, error)
}
```

### Different Implementations

```
package data

type StorageType int

const (
    DiskStorage StorageType = 1 << iota
    TempStorage
    MemoryStorage
)

func NewStore(t StorageType) Store {
    switch t {
    case MemoryStorage:
        return newMemoryStorage( /*...*/)
    case DiskStorage:
        return newDiskStorage( /*...*/)
    default:
        return newTempStorage( /*...*/)
    }
}
```

## Usage

使用工厂方法，用户可以指定他们想要的存储类型。

```
s, _ := data.NewStore(data.MemoryStorage)
f, _ := s.Open("file")

n, _ := f.Write([]byte("data"))
defer f.Close()
```