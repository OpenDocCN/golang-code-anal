# `trojan-go\statistic\mysql\mysql.go`

```go
package mysql

import (
    "context"  // 上下文包，用于控制goroutine的生命周期
    "database/sql"  // 数据库SQL操作包
    "fmt"  // 格式化包，用于格式化输出
    "strings"  // 字符串操作包
    "time"  // 时间包

    // MySQL Driver
    _ "github.com/go-sql-driver/mysql"  // 导入MySQL驱动，但不使用它

    "github.com/p4gefau1t/trojan-go/common"  // 导入trojan-go的common包
    "github.com/p4gefau1t/trojan-go/config"  // 导入trojan-go的config包
    "github.com/p4gefau1t/trojan-go/log"  // 导入trojan-go的log包
    "github.com/p4gefau1t/trojan-go/statistic"  // 导入trojan-go的statistic包
    "github.com/p4gefau1t/trojan-go/statistic/memory"  // 导入trojan-go的statistic/memory包
)

const Name = "MYSQL"  // 定义常量Name为"MYSQL"

type Authenticator struct {
    *memory.Authenticator  // 继承memory.Authenticator
    db             *sql.DB  // 数据库连接
    updateDuration time.Duration  // 更新间隔时间
    ctx            context.Context  // 上下文
}

func (a *Authenticator) updater() {
    // 无限循环，持续执行以下操作
    for {
        // 遍历所有用户
        for _, user := range a.ListUsers() {
            // 交换用户的上传和下载流量
            hash := user.Hash()
            // 重置用户的流量统计并获取重置前的上传和下载流量
            sent, recv := user.ResetTraffic()

            // 更新数据库中用户的上传和下载流量
            s, err := a.db.Exec("UPDATE `users` SET `upload`=`upload`+?, `download`=`download`+? WHERE `password`=?;", recv, sent, hash)
            if err != nil {
                // 如果更新失败，记录错误并继续下一个用户
                log.Error(common.NewError("failed to update data to user table").Base(err))
                continue
            }
            if r, err := s.RowsAffected(); err != nil {
                // 如果受影响的行数为0，说明用户不存在，删除该用户
                if r == 0 {
                    a.DelUser(hash)
                }
            }
        }
        // 记录日志，表示缓冲数据已写入数据库
        log.Info("buffered data has been written into the database")

        // 更新内存中的数据
        rows, err := a.db.Query("SELECT password,quota,download,upload FROM users")
        if err != nil || rows.Err() != nil {
            // 如果从数据库中获取数据失败，记录错误并等待一段时间后继续下一轮循环
            log.Error(common.NewError("failed to pull data from the database").Base(err))
            time.Sleep(a.updateDuration)
            continue
        }
        // 遍历查询结果
        for rows.Next() {
            var hash string
            var quota, download, upload int64
            // 从查询结果中获取数据
            err := rows.Scan(&hash, &quota, &download, &upload)
            if err != nil {
                // 如果获取数据失败，记录错误并终止当前循环
                log.Error(common.NewError("failed to obtain data from the query result").Base(err))
                break
            }
            // 如果用户的下载和上传流量小于配额或者配额小于0，添加该用户
            if download+upload < quota || quota < 0 {
                a.AddUser(hash)
            } else {
                // 否则删除该用户
                a.DelUser(hash)
            }
        }

        // 选择监听多个channel的情况
        select {
        // 等待一段时间后继续下一轮循环
        case <-time.After(a.updateDuration):
        // 如果收到退出信号，记录日志并退出循环
        case <-a.ctx.Done():
            log.Debug("MySQL daemon exiting...")
            return
        }
    }
}


func connectDatabase(driverName, username, password, ip string, port int, dbName string) (*sql.DB, error) {
    // 拼接数据库连接路径
    path := strings.Join([]string{username, ":", password, "@tcp(", ip, ":", fmt.Sprintf("%d", port), ")/", dbName, "?charset=utf8"}, "")
    // 打开数据库连接
    return sql.Open(driverName, path)
}

func NewAuthenticator(ctx context.Context) (statistic.Authenticator, error) {
    // 从上下文中获取配置信息
    cfg := config.FromContext(ctx, Name).(*Config)
    // 连接数据库
    db, err := connectDatabase(
        "mysql",
        cfg.MySQL.Username,
        cfg.MySQL.Password,
        cfg.MySQL.ServerHost,
        cfg.MySQL.ServerPort,
        cfg.MySQL.Database,
    )
    if err != nil {
        // 连接数据库失败，返回错误
        return nil, common.NewError("Failed to connect to database server").Base(err)
    }
    // 创建内存认证对象
    memoryAuth, err := memory.NewAuthenticator(ctx)
    if err != nil {
        // 创建内存认证对象失败，返回错误
        return nil, err
    }
    // 创建认证对象
    a := &Authenticator{
        db:             db,
        ctx:            ctx,
        updateDuration: time.Duration(cfg.MySQL.CheckRate) * time.Second,
        Authenticator:  memoryAuth.(*memory.Authenticator),
    }
    // 启动认证对象更新
    go a.updater()
    // 输出日志
    log.Debug("mysql authenticator created")
    // 返回认证对象
    return a, nil
}

func init() {
    // 注册认证对象创建函数
    statistic.RegisterAuthenticatorCreator(Name, NewAuthenticator)
}
```