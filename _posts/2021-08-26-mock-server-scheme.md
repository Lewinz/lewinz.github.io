---
layout: post
title: mock server 方案
categories: [mock, scheme]
description: mock server 方案
keywords: mock, scheme
---

## 项目背景
项目主要使用的是 golang，底层服务依赖第三方。由于一直处于项目功能实现阶段，测试类的编写还没有形成规范，最近需要使用 travis CI，所以需要好好弄弄测试。

## 方案设计历程
经过团队讨论（大佬指点），我们需要对现有的第三方依赖接口做 mock 处理。

### 独立的 server
开发一个独立的 mock 服务，通过页面或者数据上传的方式建立 mock 数据，可以设计延时、正向数据、反向数据等功能。

部署方式：单独部署维护在国内。  
缺点：开发成本大，mock 数据维护难度大，travis ci 机器在国外，存在网络问题。  
优点：同一请求可设置多种结果，可设计响应规则，mock 数据全面且易调整。  

### 缓存真实请求数据
在 dev 环境进行测试时，就将请求第三方的数据缓存到 DB 或者文件当中，可以制造适量的异常数据做反向测试。

优点：mock 数据不需要再额外维护，只需要将测试代码做到足够完善，设立一个测试代码标准即可。  
缺点：只能模拟在预期的业务异常数据，对于非预期异常的测试覆盖不够。  

### 与项目并行的 http 服务
在项目中另外编写一个 http 服务，充当 mock server 的角色，可解决网络问题。

### 最终方案
最终经历以上方面的方案讨论，最终决定在项目中编写一个 http 服务充当 mock server 的角色，在本地测试过程中记录真实请求和响应数据，以文件的形式提交到仓库。

## 代码实现
### 第一版
在运行 `go test` 运行测试之前，读取指定的文件目录下 mock 数据文件，按照数据中储存的信息注册 gin 框架 route，并启动 http 服务。

`go test` 运行期间，所有需要进行第三方调用的接口，通过 transport 转到 mock 请求处理中进行模拟。

实现思路：这个 mock server 相当于第三方服务的角色，从调用顺序，环境因素等角度都尽可能模拟出了真实情况

代码细节：在 http transport 中取出 request 的 url 与 body，response 中取 body，压缩之后存储在相关路径下，存储文件以 request method 为区分（最初想的是这个 mock data 文件会很大，单纯的想做一下文件分割）。在项目目录下添加 mock 目录，用来加载相关路径下的 mock 文件，注册 gin 服务，使用 request 的 url、method、param、body 作为唯一请求判断，返回响应的 response body。

![mock_server_scheme_1](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/mock_server_scheme_1.png)

缺点：
1. 需要启动一个单独的 mock server
2. 这个版本的实现，mock 数据是做了包级别隔离，同包下，同样参数的请求调用顺序不可控，response 的返回数据也不可控，mock 结果不稳定
#### 存储实现
![mock_server_scheme_2](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/mock_server_scheme_2.png)

存储 mock 数据是在原有 transport 的基础上再包裹一层存储数据的处理，在 http client 初始化的时候通过参数传入，下面举例说明一下代码的具体实现过程：

项目中原有的 http client 实现
``` golang
type Transport struct {
	Transport http.RoundTripper
}

// Client Provider
type Client struct {
	tr     *Transport
	client *http.Client
}

// NewClient ..
func NewClient() *Client {
	tr := &Transport{
		Transport: http.DefaultTransport,
	}

	return &Client{
		tr:     tr,
		client: &http.Client{Transport: tr},
	}
}
```

修改为
``` golang
func NewClient(transport ...http.RoundTripper) *Client {
	tr := &Transport{
		Transport: http.DefaultTransport,
	}

	if len(transport) > 0 {
		tr.Transport = transport[0]
	}

	return &Client{
		tr:     tr,
		client: &http.Client{Transport: tr},
	}
}
```

测试入口代码
``` golang
transport := InitTestMockTransport()

client := client.NewClient(&client.Config{
			AuthHost: inspurConfig.AuthHost,
			Username: inspurConfig.Username,
			Password: inspurConfig.Password,
		}, transport...)
```

mock_transport.go
``` golang
// NewMockTransport ..
func NewMockTransport(readDir, writeDir string) *MockTransport {
	transport := &MockTransport{
		Transport: Transport{
			Transport: http.DefaultTransport,
		},
		ServerParam: &MockServerParam{
			MockReadDir:  readDir,
			MockWriteDir: writeDir,
		},
	}

	if readDir != "" {
		transport.ServerParam.Engine = mock.HTTPTestGIN(readDir)
	}
	return transport
}

// MockTransport ..
type MockTransport struct {
	Transport
	ServerParam *MockServerParam
}

// MockServerParam ..
type MockServerParam struct {
	MockReadDir  string
	MockWriteDir string
	Engine       *gin.Engine
}

// RoundTrip ..
func (t *MockTransport) RoundTrip(req *http.Request) (resp *http.Response, err error) {
	reqBodyBackup := ""
	if req.Body != nil {
		reqBodyBackup = string(mock.CopyRequestBody(req))
	}

	if t.ServerParam.MockReadDir != "" {
		req.Header.Add(mock.RequestHeaderFuncName, mock.GetTestFunctionName())

		recorder := httptest.NewRecorder()
		t.ServerParam.Engine.ServeHTTP(recorder, req)
		resp = recorder.Result()
	} else {
		resp, err = t.Transport.RoundTrip(req)
	}

	if t.ServerParam.MockWriteDir != "" {
		t.SaveMockData(req, resp, reqBodyBackup, err)
	}
	return
}

// SaveMockData 单测保存 mock 数据
func (t *MockTransport) SaveMockData(req *http.Request, resp *http.Response, reqBodyBackUP string, reqErr error) {
	mockFilePath := mock.AppendMockFilePath(t.ServerParam.MockWriteDir)

	mockData := make(mock.Data)
	err := mock.ReadMockFile(mockFilePath, mockData)
	if err != nil {
		log.Fatalf("%v", err)
		return
	}

	mockMessage := mock.InitMessageMock(req, resp, reqBodyBackUP, reqErr)

	mockURL := mockMessage.RequestData.URL
	if val, ok := mockData[mockURL]; ok {
		mockData[mockURL] = append(val, mockMessage)
	} else {
		mockData[mockURL] = make([]*mock.MessageMock, 1)
		mockData[mockURL][0] = mockMessage
	}

	err = mock.WriteMockFile(mockFilePath, mockData)
	if err != nil {
		log.Fatalf("%v", err)
		return
	}
}
```

#### 读取实现
mock.go
``` golang
// HTTPTestGIN  获取 httptest 使用的 gin
func HTTPTestGIN(mockDirPath string) *gin.Engine {
	engine := gin.Default()

	dirs := strings.Split(mockDirPath, ",")

	mockData := make(Data)
	for i := 0; i < len(dirs); i++ {
		dirPath := dirs[i]

		err := filepath.Walk(dirPath, func(path string, info fs.FileInfo, err error) error {
			if info == nil || info.IsDir() {
				return nil
			}

			return ReadMockFile(path, mockData)
		})
		if err != nil {
			log.Fatal("mock data reload err")
		}
	}

	for k, v := range mockData {
		engineURL := strings.TrimPrefix(k, v[0].RequestData.Method+MockURLConnect)
		switch v[0].RequestData.Method {
		case http.MethodGet:
			engine.GET(engineURL, func(c *gin.Context) {
				engineRouteFunc(c, mockData)
			})
		case http.MethodPost:
			engine.POST(engineURL, func(c *gin.Context) {
				engineRouteFunc(c, mockData)
			})
		case http.MethodHead:
			engine.HEAD(engineURL, func(c *gin.Context) {
				engineRouteFunc(c, mockData)
			})
		case http.MethodPut:
			engine.PUT(engineURL, func(c *gin.Context) {
				engineRouteFunc(c, mockData)
			})
		case http.MethodPatch:
			engine.PATCH(engineURL, func(c *gin.Context) {
				engineRouteFunc(c, mockData)
			})
		case http.MethodDelete:
			engine.DELETE(engineURL, func(c *gin.Context) {
				engineRouteFunc(c, mockData)
			})
		case http.MethodOptions:
			engine.OPTIONS(engineURL, func(c *gin.Context) {
				engineRouteFunc(c, mockData)
			})
		}
	}

	engine.GET("/ping", func(c *gin.Context) {
		c.String(http.StatusOK, "pong")
	})

	return engine
}

// engineRouteFunc addRoute 请求 mock
func engineRouteFunc(c *gin.Context, mockData map[string][]*MessageMock) {
	var engineReqBody []byte
	if c.Request.Body != nil {
		engineReqBody, _ = ioutil.ReadAll(c.Request.Body)
	}

	mocks := mockData[c.Request.Method+MockURLConnect+c.Request.URL.Path]

	responseList := []*MessageMock{}
	for i := 0; i < len(mocks); i++ {
		// 判断条件：body / query / header 携带的 funcName 都一致
		if mocks[i].RequestData.Body == string(engineReqBody) &&
			mocks[i].RequestData.Param == c.Request.URL.RawQuery &&
			mocks[i].RequestData.FuncName == c.Request.Header.Get(RequestHeaderFuncName) {
			responseList = append(responseList, mocks[i])
		}
	}

	if len(responseList) == 1 {
		contextResponse(c, responseList[0])
	}

	if len(responseList) > 1 {
		for i := 0; i < len(responseList); i++ {
			if !responseList[i].Used {
				contextResponse(c, responseList[i])
				responseList[i].Used = true
				break
			}
		}
	}
}

// contextResponse ..
func contextResponse(c *gin.Context, mock *MessageMock) {
	if json.Valid([]byte(mock.ResponseData.Body)) {
		reader := strings.NewReader(mock.ResponseData.Body)
		c.DataFromReader(mock.ResponseData.StatusCode, int64(len(mock.ResponseData.Body)), "application/json; charset=utf-8", reader, nil)
	} else {
		c.String(mock.ResponseData.StatusCode, "%v", string(mock.ResponseData.Body))
	}
}
```
### 第二版
在第一版存储、读取 mock 文件操作不变情况下，对 mock server 部署方式、请求判定条件做出一些优化：
1. 将 transport 改为拔插式控件，消除对业务的耦合
2. 将原有的 gin 单独服务改为 gin + httptest，在 m *testing.M 方法中初始化 transport 的逻辑。
3. 原有的 m.run() 测试，包下测试方法执行顺序不能确定（文件层级一般情况下是按顺序的，但是不能保证一定是顺序的，同文件下方法执行顺序是不一定的），将所有测试类都改为 struct 的方法，用反射调用，反射获取 method 列表是按照方法名的 ASCLL 顺序的。
4. 将测试方法改为反射调用之后，将方法执行改为了异步执行。

测试执行部分的改动：
``` golang
// init_test.go
func TestMain(m *testing.M){
  ...

  exitCode := m.Run()
	os.Exit(exitCode)
}

// other test *.go
func TestA(t *testing.T) {
  // do something
  ...
}
```

===== >

``` golang
// init_test.go
var (
  flagMethodName = flag.String("m", "", "执行方法名")
)

type test struct {
	t          *testing.T
	wg         sync.WaitGroup
  ...
}

func TestMain(m *testing.M){
  flag.Parse()
  ...

  t := &test{
		t:          &testing.T{},
		testCore:   testCore,
		adminCore:  adminCore,
		testServer: testServer,
		testMisc:   testMisc,
		wg:         sync.WaitGroup{},
	}
  testVal := reflect.ValueOf(t)

  var params []reflect.Value
  params = append(params, reflect.ValueOf(t.t))
  if *flagMethodName != "" {
    methodNames := strings.Split(*flagMethodName, ",")
    for i := 0; i < len(methodNames); i++ {
      t.wg.Add(1)
      go testVal.MethodByName(methodNames[i]).Call(params)
    }
  } else {
    methodCount := testVal.NumMethod()
    for i := 0; i < methodCount; i++ {
      t.wg.Add(1)
      go testVal.Method(i).Call(params)
    }
  }
  t.wg.Wait()
}

// other test *.go
func (test *test) TestA(t *testing.T) {
  // do something
  ...

  test.wg.Done()
}
```

修改完成之后，进行 mock 数据制作和本地测试，发现结果不是那么稳定，日志每次都会出现不一样的异常信息（理论上 mock server 使用起来是稳定不变的）。

### 第三版
第二版做完之后，开始记录真实请求保存 mock 数据，准备做 mock 数据的提交了。但是本地跑好测试类之后，启用 read mock 总是有会一些非预期的请求错误和一些断言 faild，经过仔细回顾整个代码，发现几个问题：
1. 本地跑测试的时候，一些错误实际是发生了的，但是日志太多，并没有被关注到，最终测试结论却是 PASS，导致原因就是第二版将测试执行方式改为了反射，os.Exit 这个关键步骤没有了，所以很多错误就被忽视了。
2. 改为 goroutine 后，请求顺序更加随机了，返回的 response 顺序更加混乱。
3. 文件风格之后并没有提供什么便利，反而每次做 mock 数据的时候会需要时间思考文件名。
4. mock 数据的包级别隔离，并没有什么比较好的方法去设定测试执行顺序，且不影响测试过程中的其他表现。
5. 测试方法不够完善，有很强的业务状态依赖。

实现修改：
1. 将反射调用方法改回 m.run os.exit 方式
2. 将 mock 数据改为方法级别隔离，在 request 中添加一个 header 传入 funcname
3. 取消 mock 数据文件分隔
4. 完善测试方法，用轮询的方式解决状态依赖

最后实现效果：不论是记录 mock 数据还是读取 mock 数据，不需要再关注 mock server 的实现，只需要关注 mock 的两个环境变量，然后正常执行测试即可，本地测试时只需保证 PASS 且可重复执行即可。

### 仍可优化部分
- 反射调用测试类带来的是可以多协程执行，可考虑解决 os.exit 问题后重新改回反射。
- mock 数据很大程度依赖测试方法的代码质量。

## 经验总结
不要过度优化，某些问题等出现了再解决也不迟。