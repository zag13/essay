# json

## 文章

- [Documentation](https://pkg.go.dev/encoding/json)

## 示例

```
type X struct {
  A string  `json:"a,omitempty"`
  B *string `json:"b,omitempty"`
  C *string `json:"c,omitempty"`
}

func main() {
  js1 := `{}`
  var x1 X
  if err := json.Unmarshal([]byte(js1), &x1); err != nil {
      panic(err)
  }
  fmt.Printf("%#v\n", x1) // main.X{A:"", B:(*string)(nil), C:(*string)(nil)}
}
```