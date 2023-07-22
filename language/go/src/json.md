# json

## 文章

- [Documentation](https://pkg.go.dev/encoding/json)

## 示例

```
type X struct {
	A  string  `json:"a,omitempty"`
	B  *string `json:"b,omitempty"`
	C  *string `json:"c,omitempty"`
	Y1 Y       `json:"y1,omitempty"`
	Y2 *Y      `json:"y2,omitempty"`
}

type Y struct {
	A *string `json:"a,omitempty"`
}


func main() {
	json1 := `{}`
	var x1 X
	if err := json.Unmarshal([]byte(json1), &x1); err != nil {
		panic(err)
	}
	fmt.Printf("%#v\n", x1)
	json1_1, _ := json.Marshal(x1)
	fmt.Println(string(json1_1))
	// Output:
	// main.X{A:"", B:(*string)(nil), C:(*string)(nil), Y1:main.Y{A:(*string)(nil)}, Y2:(*main.Y)(nil)}
	// {"y1":{}}

	json2 := `{"a":null, "b":null, "y1":{}, "y2":{}}`
	var x2 X
	if err := json.Unmarshal([]byte(json2), &x2); err != nil {
		panic(err)
	}
	fmt.Printf("%#v\n", x2)
	json2_1, _ := json.Marshal(x1)
	fmt.Println(string(json2_1))
	// Output:
	// main.X{A:"", B:(*string)(nil), C:(*string)(nil), Y1:main.Y{A:(*string)(nil)}, Y2:(*main.Y)(0xc00012c080)}
	// {"y1":{}}
}
```