
# Go memo

## References

* `https://pkg.go.dev/std`

## Basic

### Definition

* Exported names
  * Goでは、最初の文字が大文字で始まる名前は、外部のパッケージから参照できるエクスポート（公開）された名前( exported name )です。 例えば、 Pi は math パッケージでエクスポートされています。小文字ではじまる pi や hoge などはエクスポートされていない名前です。パッケージをインポートすると、そのパッケージがエクスポートしている名前を参照することができます。 エクスポートされていない名前(小文字ではじまる名前)は、外部のパッケージからアクセスすることはできません。

    ```go
    package main

    import (
        "fmt"
        "math"
    )

    func main() {
        fmt.Println(math.pi) //Error
        fmt.Println(math.Pi) // OK
    }
    ```

* var vs const(注意)
  * 初期化子が与えられている場合、型を省略できます。その変数は初期化子が持つ型になります。(Variables with initializers)

    ```go
    var i, j  = 1, 2
    ```

    ```go
    package main

    import "fmt"

    const Pi = 3.14

    const (
        Username = "test_user"
        Password = "test_pass"
    )

    // var big int = 9223372036854775807 + 1 // 実行されるoverflow Error
    const big = 9223372036854775807 + 1 //まだ実行されていない, l17で実行される

    func main() {
        fmt.Println(Pi, Username, Password)
        fmt.Println(big - 1)
    }
    ```

* short variable declaration
  * 関数の中では、 var 宣言の代わりに、短い := の代入文を使い、暗黙的な型宣言ができます。  
    なお、関数の外では、キーワードではじまる宣言( var, func, など)が必要で、 := での暗黙的な宣言は利用できません。

    ```go
    func main() {
    var i, j int = 1, 2
    k := 3
    }
    ```

* string to int

    ```go
    var s string = "14"
    i, _ := strconv.Atoi(s) // 返り値int, error
    ```

* basic types
  * Go言語の基本型(組み込み型)は次のとおりです:

    ```go
    bool

    string

    int  int8  int16  int32  int64
    uint uint8 uint16 uint32 uint64 uintptr

    byte // uint8 の別名

    rune // int32 の別名
        // Unicode のコードポイントを表す

    float32 float64

    complex64 complex128
    ```

    * (訳注：runeとは古代文字を表す言葉(runes)ですが、Goでは文字そのものを表すためにruneという言葉を使います。)  
例では、いくつかの型の変数を示しています。また、変数宣言は、インポートステートメントと同様に、まとめて( factored )宣言可能です。  
int, uint, uintptr 型は、32-bitのシステムでは32 bitで、64-bitのシステムでは64 bitです。 サイズ、符号なし( unsigned )整数の型を使うための特別な理由がない限り、整数の変数が必要な場合は int を使うようにしましょう。

* Zero values
  * 変数に初期値を与えずに宣言すると、ゼロ値( zero value )が与えられます。  
ゼロ値は型によって以下のように与えられます:  
数値型(int,floatなど): 0  
bool型: false  
string型: "" (空文字列( empty string ))  

* Type conversions
  * 変数 v 、型 T があった場合、 T(v) は、変数 v を T 型へ変換します。

    ```go
    var i int = 42
    var f float64 = float64(i)
    var u uint = uint(f)
    ```

    よりシンプルに記述できます:

    ```fo
    i := 42
    f := float64(i)
    u := uint(f)
    ```

    C言語(double / intはdoubleになる)とは異なり、Goでの型変換は明示的な変換が必要です。

    ```go
    func main() {
    var x int = 5
    var y float64 = 2
    var z float64 = x / y <-ダメ
    var z float64 = float64(x) / y
    fmt.Printf("%T, %v", z, z)
    }
    ```

* Type inference
  * 明示的な型を指定せずに変数を宣言する場合( := や var = のいずれか)、変数の型は右側の変数から型推論されます。

    ```go
    var i int
    j := i // j is an int
    ```

    右側に型を指定しない数値である場合、左側の新しい変数は右側の定数の精度に基いて int, float64, complex128 の型になります:

    ```go
    i := 42           // int
    f := 3.142        // float64
    g := 0.867 + 0.5i // complex128
    ```

* constants
  * 定数( constant )は、 const キーワードを使って変数と同じように宣言します。
定数は、文字(character)、文字列(string)、boolean、数値(numeric)のみで使えます。
なお、定数は := を使って宣言できません。

    ```go
    func main() {
    // cannot assign to World (declared const)
    const World = "世界"
    World = "宇宙" 
    // OK
    var World = "世界"
    World = "宇宙" 

    fmt.Println("Hello", World)
    }
    ```

* int to string

    ```go
        h := "Hello World"
        fmt.Println(string(h[0]))
    ```

* array
  * [n]T 型は、型 T の n 個の変数の配列( array )を表します。  
  配列の長さは、型の一部分です。ですので、配列のサイズを変えることはできません。 これは制約のように思えますが、心配しないでください。 Goは配列を扱うための便利な方法を提供しています。

    ```go
        var a [2]int
        a[0] = 100
        a[1] = 200

        var b [2]int = [2]int{100, 200}
        b = append(b, 300)// Error
    ```

* slice
  * 配列は固定長です。一方で、スライスは可変長です。より柔軟な配列と見なすこともできます。 実際には、スライスは配列よりもより一般的です。

    ```go
        // appendできる
        var b []int = []int{100, 200}
        b = append(b, 300)
    ```

* Slices are like references to arrays
  * スライスはどんなデータも格納しておらず、単に元の配列の部分列を指し示しています。  
スライスの要素を変更すると、その元となる配列の対応する要素が変更されます。  
同じ元となる配列を共有している他のスライスは、それらの変更が反映されます。  

    ```go
    func main() {
    names := [4]string{
        "John",
        "Paul",
        "George",
        "Ringo",
    }
    fmt.Println(names)

    a := names[0:2]
    b := names[1:3]
    fmt.Println(a, b)

    b[0] = "XXX"
    fmt.Println(a, b)
    fmt.Println(names)
    }
    ```

    結果

    ```debug
    [John Paul George Ringo]
    [John Paul] [Paul George]
    [John XXX] [XXX George]
    [John XXX George Ringo]
    ```

* Slice length and capacity ?
  * スライスは長さ( length )と容量( capacity )の両方を持っています。  
スライスの長さは、それに含まれる要素の数です。
スライスの容量は、**スライスの最初の要素から数えて、元となる配列の要素数**です。  
スライス s の長さと容量は len(s) と cap(s) という式を使用して得ることができます

    ```go
    func main() {
    s := []int{2, 3, 5, 7, 11, 13}
    printSlice(s)

    // Slice the slice to give it zero length.
    s = s[:0]
    printSlice(s)

    // Extend its length.
    s = s[:4]
    printSlice(s)

    // Drop its first two values.
    s = s[2:]
    printSlice(s)
    }
    ```

    結果

    ```debug
    len=6 cap=6 [2 3 5 7 11 13]
    len=0 cap=6 []
    len=4 cap=6 [2 3 5 7]
    len=2 cap=4 [5 7]
    ```

* slice(make, cap)
  * スライスは、組み込みの make 関数を使用して作成することができます。 これは、動的サイズの配列を作成する方法です。
  * スライスへ新しい要素を追加するには、Goの組み込みの append を使います。 append についての詳細は [documentation](https://pkg.go.dev/builtin#append) を参照してみてください。

    ```go
    func append(s []T, vs ...T) []T
    ```

    上の定義を見てみましょう。 append への最初のパラメータ s は、追加元となる T 型のスライスです。 残りの vs は、追加する T 型の変数群です。  
append の戻り値は、追加元のスライスと追加する変数群を合わせたスライスとなります。  
もし、元の配列 s が、変数群を追加する際に容量が小さい場合は、より大きいサイズの配列を割り当て直します。 その場合、戻り値となるスライスは、新しい割当先を示すようになります。  
(スライスについてより詳しく学ぶには、[Slices: usage and internals](https://blog.golang.org/go-slices-usage-and-internals)を読んでみてください)

    ```go
        n := make([]int, 3, 5) 
        // len=3, cap=5, value=[0,0,0] 

        n = append(n, 1, 2, 3, 4, 5) 
        // len=8, cap=8, value=[0,0,0,1,2,3,4,5]

        a := make([]int, 3) 
        // len=3, cap=3, value=[0,0,0]

        b := make([]int, 0) 
        // len=0, cap=0, value=[], 0のスライスをメモリに確保

        var c []int 
        // len=0, cap=0, value=[], nil, メモリ確保しない
    ```

* make

    ```go
    func(t Type, size ...IntegerType) Type
    make on pkg.go.dev
    /*
    The make built-in function allocates and initializes an object of type slice, map, or chan (only). Like new, the first argument is a type, not a value. Unlike new, make's return type is the same as the type of its argument, not a pointer to it. The specification of the result depends on the type:

    Slice: The size specifies the length. The capacity of the slice is equal to its length. A second integer argument may be provided to specify a different capacity; it must be no smaller than the length. For example, make([]int, 0, 10) allocates an underlying array of size 10 and returns a slice of length 0 and capacity 10 that is backed by this underlying array.

    Map: An empty map is allocated with enough space to hold the specified number of elements. The size may be omitted, in which case a small starting size is 
    allocated.

    Channel: The channel's buffer is initialized with the specified buffer capacity. If zero, or the size is omitted, the channel is unbuffered.
    */
    ```

  * map

    ```go
    m := map[string]int{"apple": 100, "banana": 200}
    // map[apple:100 banana:200]

    m["new"] = 500
    // map[apple:100 banana:300 new:500]

    m2 := make(map[string]int)
    m2["pc"] = 5000
    // map[pc:5000]

        // panic
        // var m3 map[string]int // メモリを確保していない
        // m3["pc"] = 5000
        // fmt.Println(m3)

    var s []int// メモリを確保していない
    if s == nil {
        fmt.Println("Nil")
        // Nil
    // varで宣言した時はnilが入る
    ```

* Mutating Maps
  * m へ要素(elem)の挿入や更新:

    ```go
    m[key] = elem
    ```

    * 要素の取得:

    ```go
    elem = m[key]
    ```

    * 要素の削除:

    ```go
    delete(m, key)
    ```

    * キーに対する要素が存在するかどうかは、2つの目の値で確認します:

    ```go
    elem, ok = m[key]
    ```

    もし、 m に key があれば、変数 ok は true となり、存在しなければ、 ok は false となります。  
    なお、mapに key が存在しない場合、 elem はmapの要素の型のゼロ値となります。  
    Note: もし elem や ok をまだ宣言していなければ、次のように := で短く宣言できます:

    ```go
    elem, ok := m[key]
    ```

* byte(使う意味?)
  * ネットワーク
  * ファイル

    ```go
    func main() {
        b := []byte{72, 73}
        fmt.Println(b) // [72, 73]
        fmt.Println(string(b)) // HI

        c := []byte("HI")
        fmt.Println(c)
        fmt.Println(string(c)) // HI
    }
    ```

* func
  * 関数も変数です。他の変数のように関数を渡すことができます。  
関数値( function value )は、関数の引数に取ることもできますし、戻り値としても利用できます。
    * 変数名の 後ろ に型名を書くことに注意
    * Goでの戻り値となる変数に名前をつける( named return value )ことが できます。戻り値に名前をつけると、関数の最初で定義した変数名として扱われます。
    この戻り値の名前は、戻り値の意味を示す名前とすることで、関数のドキュメントとして表現するようにしましょう。  
    名前をつけた戻り値の変数を使うと、 return ステートメントに何も書かずに戻すことができます。これを "naked" return と呼びます。  
    例のコードのように、naked returnステートメントは、短い関数でのみ利用すべきです。長い関数で使うと読みやすさ( readability )に悪影響があります。

    ```go
    func add(x, y int) (int, int) {
        return x + y, x - y
    }

    func cal(price, item int) (result int) {
        // result := price * item とはできない
        result = price * item
        return
    }

    func convert(price int) float64 {
        return float64(price)
    }

    func main() {
        r1, r2 := add(10, 20)
        fmt.Println(r1, r2)

        r3 := cal(100, 2)
        fmt.Println(r3)

        f := func(x int) {
            fmt.Println("inner func", x)
        }
        f(1)

        func(x int) {
            fmt.Println("inner func", x)
        }(1)
    }
    ```

* closer
  * Goの関数は [closure](https://ja.wikipedia.org/wiki/%E3%82%AF%E3%83%AD%E3%83%BC%E3%82%B8%E3%83%A3) です。 クロージャは、それ自身の外部から変数を参照する関数値です。 この関数は、参照された変数へアクセスして変えることができ、その意味では、その関数は変数へ"バインド"( bind )されています。  
    例えば、 adder 関数はクロージャを返しています。 各クロージャは、それ自身の sum 変数へバインドされます。

    ```go
    func adder() func(int) int {
        sum := 0
        return func(x int) int {
            sum += x
            return sum
        }
    }

    func main() {
        pos := adder()
        for i := 0; i < 10; i++ {
            fmt.Println(
                pos(i),
        
            )
        }
    }
    ```

* Fibonacci closure

    ```go
    func fibonacci() func() int {
    a1 := 0
    a2 := 1
    return func() int {
        tmp := a1
        a1 = a2
        a2 += tmp
        return a1
        }
    }

    func main() {
        f := fibonacci()
        for i := 0; i < 10; i++ {
            fmt.Println(f())
        }
    }
    ```

* variadic arguments

    ```go
    func foo(params ...int) {
        fmt.Println(len(params), params)
        for _, param := range params {
            fmt.Println(param)
        }
    }

    func main() {
        foo()
        foo(10, 20)
        foo(10, 20, 30)

        s := []int{1, 2, 3}
        fmt.Println(s)
        foo(s...)
    }
    ```

### Flow control statements: for, if, else, switch and defer

* if
  * if ステートメントは、 for のように、条件の前に、評価するための簡単なステートメントを書くことができます。  
    ここで宣言された変数は、 if のスコープ内だけで有効です。

* for
  * 基本的に、 for ループはセミコロン ; で3つの部分に分かれています:  
    * 初期化ステートメント: 最初のイテレーション(繰り返し)の前に初期化が実行されます  
    * 条件式: イテレーション毎に評価されます  
    * 後処理ステートメント: イテレーション毎の最後に実行されます  
        (初期化と後処理ステートメントの記述は任意です)。  
    初期化ステートメントは、短い変数宣言によく利用します。その変数は for ステートメントのスコープ内でのみ有効です。  
    ループは、条件式の評価が false となった場合にイテレーションを停止します。  
    Note: 他の言語、C言語 や Java、JavaScriptの for ループとは異なり、 for ステートメントの3つの部分を括る括弧 ( ) はありません。なお、中括弧 { } は常に必要です。

    ```go
    func main() {
    sum := 0
    for i := 0; i < 10; i++ {
        sum += i
    }
    fmt.Println(sum)
    }
    ```

* For is Go's "while"
  * セミコロン(;)を省略することもできます。つまり、C言語などにある while は、Goでは for だけを使います。

    ```go
    func main() {
    sum := 1
    for sum < 1000 {
        sum += sum
    }
    ```

* Forever
  * ループ条件を省略すれば、無限ループ( infinite loop )になりますので、無限ループをコンパクトに表現できます。

    ```go
    func main() {
    for {
    }
    }
    ```

* range

    ```go
    l := []string{"python", "go", "java"}
    for i := 0; i < len(l); i++ {
        fmt.Println(i, l[i])
    }
    ```

    を

    ```go
    for i, v := range l {
        fmt.Println(i, v)
    }
    ```

    のように書ける。
    値だけ取り出したい場合

    ```go
    for _, v := range l {
        fmt.Println(v)
    }
    ```

    のように書く。

    ```go
    m := map[string]int{"apple": 100, "banana": 200}
    ```

    のとき  
    key, value取り出す

    ```go
    for k, v := range m {
        fmt.Println(k, v)
    }
    ```

    keyだけ取り出す

    ```go
    for k := range m {
        fmt.Println(k)
    }
    ```

    valueだけ取り出す

    ```go
    for _, v := range m {
        fmt.Println(v)
    }
    ```

* switch
  * switch ステートメントは if - else ステートメントのシーケンスを短く書く方法です。  
    Go の switch は C や C++、Java、JavaScript、PHP の switch と似ていますが、 **Go では選択された case だけを実行してそれに続く全ての case は実行されません。** これらの言語の各 case の最後に必要な **break ステートメントが Go では自動的に提供されます。** もう一つの重要な違いは Go の switch の case は定数である必要はなく、 **関係する値は整数である必要はない**ということです。

    ```go
    func main() {
        fmt.Println(runtime.GOOS)
        fmt.Print("Go runs on ")
        switch os := runtime.GOOS; os {
        case "darwin":
            fmt.Println("OS X.")
        case "linux":
            fmt.Println("Linux.")
        default:
            fmt.Printf("%s.\n", os)
        }
    }
    ```

* Switch with no condition
  * 条件のないswitchは、 ```switch true``` と書くことと同じです。  
    このswitchの構造は、長くなりがちな "if-then-else" のつながりをシンプルに表現できます。

    ```go
    func main() {
        t := time.Now()
        switch {
        case t.Hour() < 12:
            fmt.Println("Good morning!")
        case t.Hour() < 17:
            fmt.Println("Good afternoon.")
        default:
            fmt.Println("Good evening.")
        }
    }
    ```

* defer
  * defer ステートメントは、 defer へ渡した関数の実行を、呼び出し元の関数の終わり(returnする)まで遅延させるものです。  
    defer へ渡した関数の引数は、すぐに評価されますが、その関数自体は呼び出し元の関数がreturnするまで実行されません。

    ```go
    func foo() {
    defer fmt.Println("world foo")
    fmt.Println("hello foo")
    }

    func main() {
    defer fmt.Println("world")
    foo()
    fmt.Println("hello")
    ```

    結果

    ```debug
    hello foo
    world foo
    hello
    world
    ```

* Stacking defers
  * ```defer``` へ渡した関数が複数ある場合、その呼び出しはスタック( stack )されます。 呼び出し元の関数がreturnするとき、 defer へ渡した関数は LIFO(last-in-first-out) の順番で実行されます。  
    ```defer``` ステートメントについてさらに学ぶには、 [こちら(blog post)](https://blog.golang.org/defer-panic-and-recover)を読んでみてください。

    ```go
    func main() {
    fmt.Println("run")
    defer fmt.Println(1)
    defer fmt.Println(2)
    defer fmt.Println(3)
    fmt.Println("success")
    }
    ```

    結果

    ```debug
    run
    success
    3
    2
    1
    ```

* defer 使用例
  * ファイルオープン

    ```go
    file, _ := os.Open("./lesson.go")
    defer file.Close()
    data := make([]byte, 100)
    file.Read(data)
    fmt.Println(string(data))
    ```

* logging

    ```go
    func LoggingSettings(logFile string) {
        logfile, _ := os.OpenFile(logFile, os.O_RDWR|os.O_CREATE|os.O_APPEND, 0666)
        multiLogFile := io.MultiWriter(os.Stdout, logfile)
        log.SetFlags(log.Ldate | log.Ltime | log.Llongfile)
        log.SetOutput(multiLogFile)
    }

    func main() {
    LoggingSettings("test.log")
    _, err := os.Open("fdafdsafa")
    if err != nil {
        log.Fatalln("Exit", err) // コード終了
    }
    log.Println("logging!")
    log.Printf("%T %v", "test", "test")

    log.Fatalf("%T %v", "test", "test")
    log.Fatalln("error!!")

    fmt.Println("ok?")
    }
    ```

* Error Handring
  * サンプルコード

    ```go
    func main() {
        file, err := os.Open("./lesson.go")
        if err != nil {
            log.Fatalln("Error!")
        }
        defer file.Close()
        data := make([]byte, 100)
        count, err := file.Read(data)
        if err != nil {
            log.Fatalln("Error")
        }
        fmt.Println(count, string(data))

            // err := とするとエラー
        if err = os.Chdir("test"); err != nil {
            log.Fatalln("Error")
        }

    }
    ```

* panic, recover
  * サンプルコード

    ```go
    func thirdPartyConnectDB() {
        panic("Unable to connect database!")
    }

    func save() {
        defer func() {
            s := recover()
            fmt.Println(s)
            }()
            thirdPartyConnectDB()
    }

    func main() {
        save()
        fmt.Println("OK?")
    }
    ```

    結果

    ```debug
    Unable to connect database!
    OK?
    ```

### Pointer

* pointer

* new, make
  * new

    ```go
    func new(Type) *Type
    ```

    * メモリ確保

        ```go
        var p *int = new(int)
        fmt.Println(p)
        fmt.Println(*p)
        ```

        結果  

        ```debug
        0xc0000160c8
        0
        ```

    * メモリ確保なし

        ```go
        var p2 *int
        fmt.Println(p2)
        ```

        結果

        ```debug
        <nil>
        ```

  * make

    ```go
    func make(t Type, size ...IntegerType) Type
    ```

* struct
  * サンプルコード1

    ```go
    type Vertex struct {
    X, Y int
    S    string
    }
    // structno場合
    //一緒
    v6 := new(Vertex)
    v7 := &Vertex{}

    // slice, mapの場合
    s := make([]int, 0)// こっちの方がいい
    s := []int{}
    ```

  * Pointers to structs
    * フィールド X を持つstructのポインタ p がある場合、フィールド X にアクセスするには (*p).X のように書くことができます。 しかし、この表記法は大変面倒ですので、Goでは代わりに p.X と書くこともできます。
  * Struct Literals
    * structリテラルは、フィールドの値を列挙することで新しいstructの初期値の割り当てを示しています。  
Name: 構文を使って、フィールドの一部だけを列挙することができます(この方法でのフィールドの指定順序は関係ありません)。 例では X: 1 として X だけを初期化しています。  
& を頭に付けると、新しく割り当てられたstructへのポインタを戻します。

    ```go
    type Vertex struct {
        X, Y int
        S    string
    }

    func changeVertex(v Vertex) {
        v.X = 1000
    }

    func changeVertex2(v *Vertex) {
        // (*v).X = 1000
        v.X = 1000
    }

    v8 := Vertex{1, 2, "test"}
    changeVertex(v8)
    fmt.Println(v8)

    v9 := &Vertex{1, 2, "test"}
    changeVertex2(v9)
    fmt.Println(v9)
    ```

    結果

    ```debug
    {1 2 test}
    &{1000 2 test}
    ```

* Methods
  * Goには、クラス( class )のしくみはありませんが、型にメソッド( method )を定義できます。  
メソッドは、特別なレシーバ( receiver )引数を関数に取ります。  
レシーバは、 func キーワードとメソッド名の間に自身の引数リストで表現します。  
この例では、 **Area メソッドは v という名前の Vertex 型のレシーバを持つことを意味しています**。  
  * レシーバを伴うメソッドの宣言は、レシーバ型が同じパッケージにある必要があります。 他のパッケージに定義している型に対して、レシーバを伴うメソッドを宣言できません （組み込みの int などの型も同様です）。  
  * Pointer receivers
    * ポインタレシーバでメソッドを宣言できます。  
これはレシーバの型が、ある型 T への構文 `*T`があることを意味します。（なお、Tは `*int`のようなポインタ自身を取ることはできません）  
例では `*Vertex` に `Scale` メソッドが定義されています。  
ポインタレシーバを持つメソッド(ここでは `Scale` )は、レシーバが指す変数を変更できます。 レシーバ自身を更新することが多いため、**変数レシーバよりもポインタレシーバの方が一般的です**。  

    ```go
    type Vertex struct {
        X, Y int
    }

    func Area(v Vertex) int {
        return v.X * v.Y
    }

    func (v Vertex) Area() int {
    return v.X * v.Y
    }

    func (v *Vertex) Scale(i int) {
        v.X = v.X * i
        v.Y = v.Y * i
    }


    func main() {
        v := Vertex{3, 4}
        // fmt.Println(Area(v)) 関数
        fmt.Println(v.Area()) //method
        v.Scale(10)
        fmt.Println(v.Area()) //method
    }
    ```

    結果

    ```debug
    12
    1200
    ```

* コンストラクタとは
  * コンストラクタ（英：constructor）とはオブジェクト指向のプログラミング言語で登場する用語のひとつでありインスタンスを作成したタイミングで実行されるメソッドのこと

  ```go
    type Vertex struct {
        x, y int // 小文字にすることで他のパッケージから見えなくなる
    }
    //コンストラクタ
    func New(x, y int) *Vertex {
        return &Vertex{x, y}
    }
  ```

* Embedded

  ```go
    type Vertex struct {
        x, y int
    }

    // Embedded
    type Vertex3D struct {
        Vertex
        z int
    }

    func (v Vertex3D) Area3D() int {
        return v.x * v.y * v.z
    }

    func (v *Vertex3D) Scale3D(i int) {
        v.x = v.x * i
        v.y = v.y * i
        v.z = v.z * i
    }

    func New(x, y, z int) *Vertex3D {
        return &Vertex3D{Vertex{x, y}, z}
    }

    func main() {
        v := New(3, 4, 5)
        v.Scale3D(10)
        fmt.Println(v.Area3D())
    }
  ```

  結果

  ```debug
    6000
  ```

* non-structのメソッド

  ```go
    type MyInt int

    func (i MyInt) Double() int {
        fmt.Printf("%T %v\n", i, i)
        fmt.Printf("%T %v\n", 1, 1)
        return int(i * 2)
    }

    func main() {
        myInt := MyInt(10)
        fmt.Println(myInt.Double())
    }
  ```

* インターフェイスとダッグタイピング
  * Goのinterfaceは明示的に実装を宣言せず、実装側がinterfaceのシグネチャを満たしているかによって型が決まるDuck typingを採用している。
  * Interface(インターフェース) とはメソッドの型だけを定義した型のことです。  
  * interface(インターフェース) を利用することで、オブジェクト指向言語でいうところのポリモーフィズムと同様の機能を実現できます。

  ```go
    // Sayメソッドを実装していることを期待しています。
    type Human interface {
        Say() string
    }

    type Person struct {
        Name string
    }

    type Dog struct {
        Name string
    }

    func (p *Person) Say() string {
        p.Name = "Mr." + p.Name
        fmt.Println(p.Name)
        return p.Name
    }

    func DriveCar(human Human) {
        if human.Say() == "Mr.Mike" {
            fmt.Println("Run")
        } else {
            fmt.Println("Get out")
        }
    }

    func main() {
        var mike Human = &Person{"Mike"}
        var x Human = &Person{"X"}
        DriveCar(mike)
        DriveCar(x)
        // var dog Dog = Dog{"dog"}
        // DriveCar(dog)
    }
  ```

  ```go
    type I interface {
        M()
    }

    type T struct {
        S string
    }

    // This method means type T implements the interface I,
    // but we don't need to explicitly declare that it does so.
    func (t T) M() {
        fmt.Println(t.S)
    }

    func main() {
        var i I = T{"hello"}
        i.M()
    }
    ```

* Interface values
  * 下記のように、インターフェースの値は、値と具体的な型のタプルのように考えることができます:  
`(value, type)`  
インターフェースの値は、特定の基底になる具体的な型の値を保持します。  
インターフェースの値のメソッドを呼び出すと、その基底型の同じ名前のメソッドが実行されます。

    ```go
    type I interface {
        M()
    }

    type T struct {
        S string
    }

    func (t *T) M() {
        fmt.Println(t.S)
    }

    type F float64

    func (f F) M() {
        fmt.Println(f)
    }

    func main() {
        var i I

        i = &T{"Hello"}
        describe(i)
        i.M()

        i = F(math.Pi)
        describe(i)
        i.M()
    }

    func describe(i I) {
        fmt.Printf("(%v, %T)\n", i, i)
    }
    ```

    結果

    ```debug
    (&{Hello}, *main.T)
    Hello
    (3.141592653589793, main.F)
    3.141592653589793
    ```

* Interface values with nil underlying values
  * **インターフェース自体の中にある具体的な値が nil の場合、メソッドは nil をレシーバーとして呼び出されます。**  
いくつかの言語ではこれは null ポインター例外を引き起こしますが、**Go では nil をレシーバーとして呼び出されても適切に処理するメソッドを記述するのが一般的です(この例では M メソッドのように)。**  
**具体的な値として nil を保持するインターフェイスの値それ自体は非 nil であることに注意してください。**

    ```go
    type I interface {
        M()
    }

    type T struct {
        S string
    }

    func (t *T) M() {
        if t == nil {
            fmt.Println("<nil>")
            return
        }
        fmt.Println(t.S)
    }

    func main() {
        var i I

        var t *T
        i = t
        describe(i) //(<nil>, *main.T)
        i.M()// <nil>

        i = &T{"hello"}
        describe(i)// (&T{"hello"}, *main.T)
        i.M()// hello
    }

    func describe(i I) {
        fmt.Printf("(%v, %T)\n", i, i)
    }
    ```

* Nil interface values
  * nil インターフェースの値は、値も具体的な型も保持しません。  
呼び出す 具体的な メソッドを示す型がインターフェースのタプル内に存在しないため、 nil インターフェースのメソッドを呼び出すと、ランタイムエラーになります。

    ```go
    type I interface {
        M()
    }

    func main() {
        var i I
        describe(i)
        i.M()
    }

    func describe(i I) {
        fmt.Printf("(%v, %T)\n", i, i)
    }
    ```

    結果

    ```debug
    (<nil>, <nil>)
    panic: runtime error: invalid memory address or nil pointer dereference
    [signal SIGSEGV: segmentation violation code=0x1 addr=0x0 pc=0x47f427]
    ```

* type assertion
  * インターフェイスに作用する
  * 型アサーション は、インターフェースの値の基になる具体的な値を利用する手段を提供します。  
`t := i.(T)`
この文は、インターフェースの値 `i` が具体的な型 `T` を保持し、基になる `T` の値を変数 `t` に代入することを主張します。  
`i` が `T` を保持していない場合、この文は panic を引き起こします。  
インターフェースの値が特定の型を保持しているかどうかを テスト するために、型アサーションは2つの値(基になる値とアサーションが成功したかどうかを報告するブール値)を返すことができます。  
`t, ok := i.(T)`  
`i` が `T` を保持していれば、 `t` は基になる値になり、 `ok` は真(true)になります。  
そうでなければ、 `ok` は偽(false)になり、 `t` は型 `T` のゼロ値になり panic は起きません。  
この構文と map から読み取る構文との類似点に注意してください。

    ```go
    func main() {
        var i interface{} = "hello"

        s := i.(string)
        fmt.Println(s)

        s, ok := i.(string)
        fmt.Println(s, ok)

        f, ok := i.(float64)
        fmt.Println(f, ok)

        f = i.(float64)
        fmt.Println(f)
    }
    ```

    結果

    ```debug
    hello
    hello true
    0 false
    panic: interface conversion: interface {} is string, not float64
    ```

* switch type
  * 型switch はいくつかの型アサーションを直列に使用できる構造です。  
型switchは通常のswitch文と似ていますが、型switchのcaseは型(値ではない)を指定し、それらの値は指定されたインターフェースの値が保持する値の型と比較されます。

  ```go
      switch v := i.(type) {
      case T:
          // here v has type T
      case S:
          // here v has type S
      default:
          // no match; here v has the same type as i
      }
  ```

  型switchの宣言は、型アサーション i.(T) と同じ構文を持ちますが、特定の型 T はキーワード type に置き換えられます。  
  このswitch文は、インターフェースの値 i が 型 T または S の値を保持するかどうかをテストします。 T および S の各caseにおいて、変数 v はそれぞれ 型 T または S であり、 i によって保持される値を保持します。 defaultの場合(一致するものがない場合)、変数 v は同じインターフェース型で値は i となります。

* Stringer
  * もっともよく使われているinterfaceの一つに fmt パッケージ に定義されている Stringer があります:  

    ```go
    type Stringer interface {
        String() string
    }
    ```

    ```go
    type Person struct {
        Name string
        Age  int
    }

    func (p Person) String() string {
        return fmt.Sprintf("My name is %v.", p.Name)
    }

    func main() {
        mike := Person{"Mike", 22}
        fmt.Println(mike)
    }
    ```

  Stringer インタフェースは、stringとして表現することができる型です。 fmt パッケージ(と、多くのパッケージ)では、変数を文字列で出力するためにこのインタフェースがあることを確認します。

* カスタムエラー
  * サンプルコード

    ```go
    type UserNotFound struct {
        Username string
    }

    func (e *UserNotFound) Error() string {
        return fmt.Sprintf("User not found: %v", e.Username)
    }

    func myFunc() error {
        // Something wrong
        ok := false
        if ok {
            return nil
        }
        return &UserNotFound{Username: "mike"}
    }

    func main() {
        if err := myFunc(); err != nil {
            fmt.Println(err)
        }
    }
    ```

* Readersについて

* Imagesについて

    ```go
    package image

    type Image interface {
        ColorModel() color.Model
        Bounds() Rectangle
        At(x, y int) color.Color
    }
    ```

* goroutine
  *goroutine (ゴルーチン)は、Goのランタイムに管理される軽量なスレッドです。  

  ```go
    go f(x, y, z)
  ```

  と書けば、新しいgoroutineが実行されます。  
  f , x , y , z の評価は、実行元(current)のgoroutineで実行され、 f の実行は、新しいgoroutineで実行されます。  
  goroutineは、同じアドレス空間で実行されるため、共有メモリへのアクセスは必ず同期する必要があります。 sync パッケージは同期する際に役に立つ方法を提供していますが、別の方法があるためそれほど必要ありません。

    ```go
    func goroutine(s string, wg *sync.WaitGroup) {
        defer wg.Done()
        for i := 0; i < 5; i++ {
            //time.Sleep(100 * time.Millisecond)
            fmt.Println(s)
        }
    }

    func normal(s string) {
        for i := 0; i < 5; i++ {
            //time.Sleep(100 * time.Millisecond)
            fmt.Println(s)
        }
    }

    func main() {
        var wg sync.WaitGroup
        wg.Add(1)
        go goroutine("world", &wg)
        normal("hello")
        // time.Sleep(2000 * time.Millisecond)
        wg.Wait()
    }
    ```

* Channels
  * チャネル( Channel )型は、チャネルオペレータの <- を用いて値の送受信ができる通り道です。  

    ```go
    ch <- v    // v をチャネル ch へ送信する
    v := <-ch  // ch から受信した変数を v へ割り当てる
    ```

    (データは、矢印の方向に流れます)
    マップとスライスのように、チャネルは使う前に以下のように生成します:

    ```go
    ch := make(chan int)
    ```

    通常、片方が準備できるまで送受信はブロックされます。これにより、明確なロックや条件変数がなくても、goroutineの同期を可能にします。

    ```go
    func sum(s []int, c chan int) {
        sum := 0
        for _, v := range s {
            sum += v
        }
        c <- sum // send sum to c
    }

    func main() {
        s := []int{7, 2, 8, -9, 4, 0}

        c := make(chan int)
        go sum(s[:len(s)/2], c)
        go sum(s[len(s)/2:], c)
        x, y := <-c, <-c // receive from c

        fmt.Println(x, y, x+y)
    }
    ```

    結果

    ```debug
    -5 17 12(順番が逆になるのはなぜ?)
    ```

* Buffered Channels
  * チャネルは、 バッファ ( buffer )として使えます。 バッファを持つチャネルを初期化するには、 make の２つ目の引数にバッファの長さを与えます:

    ```go
    ch := make(chan int, 100)
    ```

    バッファが詰まった時は、チャネルへの送信をブロックします。 バッファが空の時には、チャネルの受信をブロックします。

* Range and Close
  * **送り手は、これ以上の送信する値がないことを示すため、チャネルを close できます。受け手は、受信の式に2つ目のパラメータを割り当てることで、そのチャネルがcloseされているかどうかを確認できます:**

    ```go
    v, ok := <-ch
    ```

    受信する値がない、かつ、チャネルが閉じているなら、 ok の変数は、 false になります。  

    **ループの for i := range c は、チャネルが閉じられるまで、チャネルから値を繰り返し受信し続けます。**  
    注意: **送り手のチャネルだけをclose**してください。受け手はcloseしてはいけません。 もしcloseしたチャネルへ送信すると、パニック( panic )します。  
    もう一つ注意: チャネルは、ファイルとは異なり、通常は、closeする必要はありません。 **closeするのは、これ以上値が来ないことを受け手が知る必要があるときにだけです。 例えば、 range ループを終了するという場合です。**

* Select
  * select ステートメントは、goroutineを複数の通信操作で待たせます。  
    select は、複数ある case のいずれかが準備できるようになるまでブロックし、準備ができた case を実行します。 もし、複数の case の準備ができている場合、 case はランダムに選択されます。

    ```go

    func fibonacci(c, quit chan int) {
        x, y := 0, 1
        for {
            select {
            case c <- x:
                x, y = y, x+y
            case <-quit:
                fmt.Println("quit")
                return
            default:
            // Default Selection
            }
        }
    }

    func main() {
        c := make(chan int)
        quit := make(chan int)
        go func() {
            for i := 0; i < 10; i++ {
                fmt.Println(<-c)
            }
            quit <- 0
        }()
        fibonacci(c, quit)
    }
    ```

* Default Select
  *  サンプル

    ```go

    func main() {
    tick := time.Tick(100 * time.Millisecond)
    boom := time.After(500 * time.Millisecond)
    OuterLoop:
        for {
            select {
            case <-tick:
                fmt.Println("tick.")
            case <-boom:
                fmt.Println("BOOM!")
                break OuterLoop
                // return
            default:
                fmt.Println("    .")
                time.Sleep(50 * time.Millisecond)
            }
        }
        fmt.Println("##############")
    }
    ```

    結果

    ```debug
        .
        .
    tick.
        .
        .
    tick.
        .
        .
    tick.
        .
        .
    tick.
        .
        .
    BOOM!
    ##############
    ```

* Producer, Consumer
  * サンプル
    * func consumerのforループの後にclose(ch)を書いてはいけない理由: producerのループはgorutineで並列処理をしているが、もしgorutineの処理が長い場合は、producerの処理が終わる前にcloseしてしまう可能性がある。そのためconsumerの処理が終わった後にcloseしている。

    ```go
    func producer(ch chan int, i int) {
        // Something
        ch <- i * 2
    }

    func consumer(ch chan int, wg *sync.WaitGroup) {
        for i := range ch {
            func() {
                defer wg.Done()// func が終わったら呼ばれる
                fmt.Println("process", i*1000)
            }()
        }
        fmt.Println("################")
    }

    func main() {
        var wg sync.WaitGroup
        ch := make(chan int)

        // Producer
        for i := 0; i < 10; i++ {
            wg.Add(1)
            go producer(ch, i)
        }

        // Consumer
        go consumer(ch, &wg)
        wg.Wait()
        // close(ch) //closeはmainで書く。
        time.Sleep(2 * time.Second)
        fmt.Println("Done")
    }
    ```

    ```debug
    process 0
    process 6000
    process 2000
    process 4000
    process 16000
    process 10000
    process 8000
    process 14000
    process 18000
    process 12000
    Done
    ```

    ################が表示されない。これは、consumerはchを待っているから。forループが終了していないから。

* fan-out fan-in
  * タスクを複数のgoroutineに分割して、順序通りに処理させたい場合に利用します。  
fan-inは、複数の入力を1つのchannelにまとめて受信するパターン。論理ゲートが処理できる入力数。複数のgoroutineで処理されたchannelを１本に集約する処理  
fan-outは、複数の関数（goroutine）が、1つのchannelから値を読み取り送信するパターン。１本のpipelineの流れを複数に分ける処理（同時に複数のgoroutineを起動する処理）fan-outはCPUに作業を分配するため、Workersパターンとも呼ばれる。  
  * 次の例では、producerとmulti2とmulti4、main関数をchannelを通して接続し、
複数の処理を並行に、かつタスク分割して順々に実行しています。
処理が終わったchannelは、closeしないとrangeループから抜けられずに、
deadlockを引き起こすため、注意が必要。

    ```go
    func producer(first chan int) {
        defer close(first)
        for i := 0; i < 10; i++ {
            first <- i
        }
    }

    func multi2(first <-chan int, second chan<- int) {
        defer close(second)
        for i := range first {
            second <- i * 2
        }
    }

    func multi4(second <-chan int, third chan<- int) {
        defer close(third)
        for i := range second {
            third <- i * 4
        }
    }

    func main() {
        first := make(chan int)
        second := make(chan int)
        third := make(chan int)

        go producer(first)
        go multi2(first, second)
        go multi4(second, third)
        for result := range third {
            fmt.Println(result)
        }
    }
    ```

    ```debug
    0
    8
    16
    24
    32
    40
    48
    56
    64
    72
    ```

* sync.Mutex
  * チャネルが、goroutine間で通信するための素晴らしい方法であることを見てきました。  
しかし、通信が必要ない場合はどうでしょう？ コンフリクトを避けるために、一度に**1つのgoroutineだけが変数にアクセスできるようにしたい**場合はどうでしょうか？  
このコンセプトは、排他制御( mutual exclusion )と呼ばれ、このデータ構造を指す一般的な名前は mutex (ミューテックス)です。  
Goの標準ライブラリは、排他制御をsync.Mutexと次の二つのメソッドで提供します:  
`Lock`  
`Unlock`

    ```go
    type Counter struct {
        v   map[string]int
        mux sync.Mutex
    }

    func (c *Counter) Inc(key string) {
        c.mux.Lock()
        defer c.mux.Unlock()
        c.v[key]++
    }

    func (c *Counter) Value(key string) int {
        c.mux.Lock()
        defer c.mux.Unlock()
        return c.v[key]
    }

    func main() {
        // c := make(map[string]int)
        c := Counter{v: make(map[string]int)}
        go func() {
            for i := 0; i < 10; i++ {
                // c["key"] += 1
                c.Inc("Key")
            }
        }()
        go func() {
            for i := 0; i < 10; i++ {
                // c["key"] += 1
                c.Inc("Key")
            }
        }()
        time.Sleep(1 * time.Second)
        fmt.Println(c, c.Value("Key"))
    }
    ```

* パッケージ
  * インポートできない

  * time
  * regex 正規表現
  * sort
  * iota 連番
  * context タイムアウト処理, ユーザーからのリクエスト
  * ioutil IO関係 パケットの読み込み書き込み

* ライブラリ
  * http net/http, net/url
  * json encoding/json marshal, unmarshal
  * hmac sha256

* サードパーティのパッケージ
  * semaphore 起動するゴルーチンの数を指定
  * ini Configファイル読み込む
  * talib 株価分析
  * go-sqlite3