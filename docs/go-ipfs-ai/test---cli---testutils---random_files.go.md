# `kubo\test\cli\testutils\random_files.go`

```go
package testutils

import (
    "fmt" // 导入格式化包
    "io" // 导入输入输出包
    "math/rand" // 导入随机数包
    "os" // 导入操作系统包
    "path" // 导入路径包
    "time" // 导入时间包
)

var (
    AlphabetEasy = []rune("abcdefghijklmnopqrstuvwxyz01234567890-_") // 定义包含简单字符的符文切片
    AlphabetHard = []rune("abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ01234567890!@#$%^&*()-_+= ;.,<>'\"[]{}() ") // 定义包含复杂字符的符文切片
)

type RandFiles struct {
    Rand         *rand.Rand // 随机数生成器
    FileSize     int // 每个文件的大小
    FilenameSize int // 文件名的长度
    Alphabet     []rune // 文件名的字符集

    FanoutDepth int // 目录层级
    FanoutFiles int // 每个目录中的文件数
    FanoutDirs  int // 每个目录中的子目录数

    RandomSize   bool // 随机文件大小
    RandomFanout bool // 随机目录结构
}

func NewRandFiles() *RandFiles {
    return &RandFiles{
        Rand:         rand.New(rand.NewSource(time.Now().UnixNano())), // 使用当前时间的纳秒数作为随机数种子
        FileSize:     4096, // 默认文件大小为 4096
        FilenameSize: 16, // 默认文件名长度为 16
        Alphabet:     AlphabetEasy, // 默认使用简单字符集
        FanoutDepth:  2, // 默认目录层级为 2
        FanoutDirs:   5, // 默认每个目录中的子目录数为 5
        FanoutFiles:  10, // 默认每个目录中的文件数为 10
        RandomSize:   true, // 默认随机文件大小为 true
    }
}

func (r *RandFiles) WriteRandomFiles(root string, depth int) error {
    numfiles := r.FanoutFiles // 获取每个目录中的文件数
    if r.RandomFanout {
        numfiles = rand.Intn(r.FanoutFiles) + 1 // 如果随机目录结构为 true，则随机生成文件数
    }

    for i := 0; i < numfiles; i++ {
        if err := r.WriteRandomFile(root); err != nil { // 写入随机文件
            return err
        }
    }

    if depth+1 <= r.FanoutDepth {
        numdirs := r.FanoutDirs // 获取每个目录中的子目录数
        if r.RandomFanout {
            numdirs = r.Rand.Intn(numdirs) + 1 // 如果随机目录结构为 true，则随机生成子目录数
        }

        for i := 0; i < numdirs; i++ {
            if err := r.WriteRandomDir(root, depth+1); err != nil { // 写入随机子目录
                return err
            }
        }
    }

    return nil
}

func (r *RandFiles) RandomFilename(length int) string {
    b := make([]rune, length) // 创建指定长度的符文切片
    for i := range b {
        b[i] = r.Alphabet[r.Rand.Intn(len(r.Alphabet))] // 在字符集中随机选择字符组成文件名
    }
    return string(b) // 返回文件名
}

func (r *RandFiles) WriteRandomFile(root string) error {
    filesize := int64(r.FileSize) // 获取文件大小
    # 如果随机大小为真，则随机生成文件大小
    if r.RandomSize:
        filesize = r.Rand.Int63n(filesize) + 1

    # 生成随机文件名的长度
    n := rand.Intn(r.FilenameSize-4) + 4
    # 生成随机文件名
    name := r.RandomFilename(n)
    # 拼接文件路径
    filepath := path.Join(root, name)
    # 创建文件
    f, err := os.Create(filepath)
    # 如果创建文件出错，则返回错误
    if err != nil:
        return fmt.Errorf("creating random file: %w", err)

    # 将随机数据拷贝到文件中
    if _, err := io.CopyN(f, r.Rand, filesize); err != nil:
        return fmt.Errorf("copying random file: %w", err)

    # 关闭文件
    return f.Close()
// WriteRandomDir 以随机生成的文件名创建一个随机目录，并在其中写入随机文件
func (r *RandFiles) WriteRandomDir(root string, depth int) error {
    // 如果深度超过了指定的层数，则直接返回
    if depth > r.FanoutDepth {
        return nil
    }

    // 生成一个随机长度的文件名
    n := rand.Intn(r.FilenameSize-4) + 4
    name := r.RandomFilename(n)
    // 将随机生成的文件名与根目录拼接成新的路径
    root = path.Join(root, name)
    // 创建目录，权限为755
    if err := os.MkdirAll(root, 0o755); err != nil {
        return fmt.Errorf("creating random dir: %w", err)
    }

    // 在随机目录中写入随机文件
    err := r.WriteRandomFiles(root, depth)
    if err != nil {
        return fmt.Errorf("writing random files in random dir: %w", err)
    }
    return nil
}
```