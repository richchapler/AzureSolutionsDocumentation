# KQL logic

```
datatable(s:string)
    [
    "ABCDEFGHIJKLMNOP1,2022-01-02T03:04:05.1234567Z"
    "ABCDEFGHIJKLMNOP1,2022-01-02T03:04:05.1234567ZABCDEFGHIJKLMNOP1,2022-01-02T03:04:05.1234567Z"
    "ABCDEFGHIJKLMNOP1,2022-01-02T03:04:05.1234567ZABCDEFGHIJKLMNOP1,2022-01-02T03:04:05.1234567ZABCDEFGHIJKLMNOP1,2022-01-02T03:04:05.1234567Z"
    ]
| project columns = extract_all(@'(.{17,}?202\d-.{22}Z)', s)
| mv-expand columns to typeof(string)
```

Regex explained:<br>
`.{17,}?202\d.-{22}Z`

*	`.{17,}` ... minimum 17 characters – assuming it should consume the starting line and skip the first time column (which can also contain ‘Z’ character)
*	`?` ... non-greedy (to consume till 202xxx is met)
*	`202/d.-` ... pattern of “202[0..9]-“ which should work till year 2030
*	`{22}` ... length of remaining 22 characters for datetime (which is 28 after subtracting 5 chars of prefix ‘2022-‘ and suffix ‘Z’)
