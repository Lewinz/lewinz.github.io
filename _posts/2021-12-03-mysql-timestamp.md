---
layout: post
title: gorm/mysql timestamp 异常引发的 debug
categories: [golang, gorm, timestamp]
description: gorm/mysql timestamp 异常引发的 debug
keywords: golang, gorm, timestamp
---

## 问题上下文
因为业务需要，写了一个每小时执行一次的自动任务，用来生成每小时的计量数据  
计量表存储的数据为每小时为一个计量单位，需要考虑多节点重复在单个计量周期跑自动任务问题。即需要计量数据进行覆盖。

## 问题细节
项目使用 gorm 作为 orm 层框架，测试环境使用 gorm tag 建表，如果使用 time 作为时间字段类型，gorm 默认的 mysql 字段为 datatime(3)

那么第一个问题来了，datatime(3) 类型是包含毫秒存储在库中，计量自动任务进行去重时，时间过滤不方便，如果在 SQL 中强行转换时间格式进行比较，很不优雅。

解决方案是使用自定义时间类型 timestamp，在 timestamp 中重写 gorm 方法 `GormDataType`，并修改 `Value` 方法，设定 gorm 拼接 SQL 的时间格式，如下示例：
``` golang
package timestamp

import (
	"database/sql/driver"
	"fmt"
	"strings"
	"time"

	"github.com/araddon/dateparse"
)

// TimestampFormatLayout formatlayout, save in mysql
const TimestampFormatLayout = "2006-01-02 15:04:05"

// New return timestamp
func New() Timestamp {
	return Timestamp{}
}

// NewWithTime return a specified time's Timestamp
func NewWithTime(t time.Time) Timestamp {
	return Timestamp{Time: t}
}

// Now returns the current local time.
func Now() Timestamp {
	return Timestamp{Time: time.Now()}
}

// Unix returns the local Time corresponding to the given Unix time,
func Unix(sec int64, nsec int64) Timestamp {
	return Timestamp{time.Unix(sec, nsec)}
}

// Date return time with date
func Date(year int, month time.Month, day, hour, min, sec, nsec int, loc *time.Location) Timestamp {
	return Timestamp{time.Date(year, month, day, hour, min, sec, nsec, loc)}
}

// AutoSet auto set type
func AutoSet(data interface{}, fields ...string) {
	if m, ok := data.(map[string]interface{}); ok {
		for _, field := range fields {
			v, exists := m[field]
			if !exists {
				continue
			}
			if t, ok := v.(float64); ok {
				m[field] = Unix(int64(t), 0)
			}
			if t, ok := v.(int64); ok {
				m[field] = Unix(t, 0)
			}
		}
	}
}

// --------------------------------------------------------------------

// Timestamp unix timestamp
type Timestamp struct {
	time.Time
}

// Pointer return timestamp point
func (t Timestamp) Pointer() *Timestamp {
	return &t
}

// MarshalJSON implements the json.Marshaler interface.
func (t Timestamp) MarshalJSON() ([]byte, error) {

	if t.IsZero() || t.Time.UnixNano() == time.Unix(0, 0).UnixNano() {
		return []byte("null"), nil
	}

	ts := t.Unix()
	stamp := fmt.Sprint(ts)
	return []byte(stamp), nil
}

// UnmarshalJSON implements the json.Unmarshaler interface.
func (t *Timestamp) UnmarshalJSON(b []byte) error {
	var (
		err error
		str = strings.Replace(string(b), "\"", "", -1)
	)

	if str == "null" || str == "0" {
		return nil
	}

	t.Time, err = dateparse.ParseAny(str)
	return err
}

// --------------------------------------------------------------------

// GormDataType gorm type
func (Timestamp) GormDataType() string {
	return "datetime"
}

// Scan valueof time.Time
func (t *Timestamp) Scan(value interface{}) error {
	if value == nil {
		return nil
	}
	if v, ok := value.(time.Time); ok {
		*t = Timestamp{v}
		return nil
	}
	return fmt.Errorf("can not convert %v to timestamp", value)
}

// Value insert timestamp into mysql need this function.
func (t Timestamp) Value() (driver.Value, error) {
	if t.IsZero() || t.Time.UnixNano() == time.Unix(0, 0).UnixNano() {
		return nil, nil
	}

	return t.Truncate(time.Second), nil
}

```

这样，类型问题解决了。但是问题没有这么简单。

在后来的使用中发现，修改了 GormDataType 方法后，gorm 的自动填充 CreatedAt、UpdatedAt 功能失效，翻看 gorm 源码如下:
``` golang
if dataTyper, ok := fieldValue.Interface().(GormDataTypeInterface); ok {
		field.DataType = DataType(dataTyper.GormDataType())
	}

	if v, ok := field.TagSettings["AUTOCREATETIME"]; ok || (field.Name == "CreatedAt" && (field.DataType == Time || field.DataType == Int || field.DataType == Uint)) {
		if field.DataType == Time {
			field.AutoCreateTime = UnixTime
		} else if strings.ToUpper(v) == "NANO" {
			field.AutoCreateTime = UnixNanosecond
		} else if strings.ToUpper(v) == "MILLI" {
			field.AutoCreateTime = UnixMillisecond
		} else {
			field.AutoCreateTime = UnixSecond
		}
	}

	if v, ok := field.TagSettings["AUTOUPDATETIME"]; ok || (field.Name == "UpdatedAt" && (field.DataType == Time || field.DataType == Int || field.DataType == Uint)) {
		if field.DataType == Time {
			field.AutoUpdateTime = UnixTime
		} else if strings.ToUpper(v) == "NANO" {
			field.AutoUpdateTime = UnixNanosecond
		} else if strings.ToUpper(v) == "MILLI" {
			field.AutoUpdateTime = UnixMillisecond
		} else {
			field.AutoUpdateTime = UnixSecond
		}
	}

	if val, ok := field.TagSettings["TYPE"]; ok {
		switch DataType(strings.ToLower(val)) {
		case Bool, Int, Uint, Float, String, Time, Bytes:
			field.DataType = DataType(strings.ToLower(val))
		default:
			field.DataType = DataType(val)
		}
	}
```

发现 gorm 填充字段使用的就是 GormDataType 方法去判断。

解决办法：  
修改 GormDBDataType 方法，还原 GormDataType 方法，能够修改默认数据库字段类型的同时也不会影响字段填充。

``` golang
// GormDBDataType gorm type
func (Timestamp) GormDBDataType(db *gorm.DB, field *schema.Field) string {
	switch db.Dialector.Name() {
	case "mysql", "sqlite":
		return "datetime"
	case "postgres":
		return "timestamp"
	}
	return "datetime"
}
```