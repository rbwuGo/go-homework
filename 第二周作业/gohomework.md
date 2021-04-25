## 20210425

#### 问题描述：
 ```
我们在数据库操作的时候，比如dao层中当遇到一个sql.ErrNoRows的时候，
是否应该Wrap这个error，抛给上层。为什么，应该怎么做请写出代码 ?
```

#### 代码实现：
```go
func QueryUserDataList(db *sql.DB)  ([]User, error) {
	var dataList []User
	rows, err := db.Query("SELECT id,name FROM user")
	if err != nil {
		return dataList, errors.WithMessage(err, "sqlQuery:Query: query failed")
	}
	defer rows.Close()

	for rows.Next() {
		var id int
		var username string
		err = rows.Scan(&id, &username)
		if err != nil {
			return dataList, errors.WithMessage(err, "sqlQuery:RowScan: scan failed")
			// continue
		}
		dataList = append(dataList, User{
			Id:   id,
			User: username,
		})
	}
	if err = rows.Err(); err != nil {
		return dataList, errors.WithMessage(rows.Err(), "sqlQuery:RowEnd: rows next failed")
	}
	return dataList, nil
}
```
