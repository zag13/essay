# Functional Options

Functional options 是在 Go 中实现干净的、雄辩的 API 的一种方法。Options 的实现函数被用来设置该 option 的值。

## Implementation

### Options

```
package file

type Options struct {
    UID         int
    GID         int
    Flags       int
    Contents    string
    Permissions os.FileMode
}

type Option func(*Options)

func UID(userID int) Option {
    return func(args *Options) {
        args.UID = userID
    }
}

func GID(groupID int) Option {
    return func(args *Options) {
        args.GID = groupID
    }
}

func Contents(c string) Option {
    return func(args *Options) {
        args.Contents = c
    }
}

func Permissions(perms os.FileMode) Option {
    return func(args *Options) {
        args.Permissions = perms
    }
}
```

### Constructor

```
package file

func New(filepath string, setters ...Option) error {
    // Default Options
    args := &Options{
        UID:         os.Getuid(),
        GID:         os.Getgid(),
        Contents:    "",
        Permissions: 0666,
        Flags:       os.O_CREATE | os.O_EXCL | os.O_WRONLY,
    }
    
    for _, setter := range setters {
        setter(args)
    }
    
    f, err := os.OpenFile(filepath, args.Flags, args.Permissions)
    if err != nil {
        return err
    } else {
        defer f.Close()
    }
    
    if _, err := f.WriteString(args.Contents); err != nil {
        return err
    }
    
    return f.Chown(args.UID, args.GID)
}
```

## Usage

```
emptyFile, err := file.New("/tmp/empty.txt")
if err != nil {
    panic(err)
}

fillerFile, err := file.New("/tmp/file.txt", file.UID(1000), file.Contents("Lorem Ipsum Dolor Amet"))
if err != nil {
    panic(err)
}
```
