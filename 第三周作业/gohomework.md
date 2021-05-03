## 20210503

#### 问题描述：
 ```
基于errgroup实现一个http server的启动和关闭，
以及linux signal信号的注册和处理，要保证能够一个退出，全部注销退出；
```

#### 代码实现：
```go
var (
	ListenAddress = "127.0.0.1:30080"
)

func Response(resp http.ResponseWriter, req *http.Request) {
	timeNow := time.Now().UnixNano()
	io.WriteString(resp, fmt.Sprintf("srvTime: %d", timeNow))
}

func HttpServer(ctx context.Context) {
	mux := http.NewServeMux()
	mux.HandleFunc("/api", Response)
	app := &http.Server{
		Addr:     ListenAddress,
		Handler:  mux,
	}

	eg := errgroup.Group{}
	eg.Go(func() error{
		log.Println("Listen:", ListenAddress)
		err := app.ListenAndServe()
		if err != nil {
			if err == http.ErrServerClosed {
				return nil
			}
			log.Fatal(err)
		}
		return nil
	})
	eg.Go(func() error {
		for {
			select {
			case <-ctx.Done():
				_ = app.Shutdown(ctx)
				log.Println("app: Shutdown")
				return nil
			}
		}
	})
	_ = eg.Wait()
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	go HttpServer(ctx)

	ch := make(chan os.Signal, 1)
	signal.Notify(ch, syscall.SIGINT, syscall.SIGHUP, syscall.SIGQUIT, syscall.SIGTERM)
	for {
		s := <-ch
		switch s {
		case syscall.SIGINT, syscall.SIGQUIT, syscall.SIGTERM:
			cancel()
			time.Sleep(time.Second * 10)
			log.Println("app: service is stop.")
			return
		}
	}
}
```
