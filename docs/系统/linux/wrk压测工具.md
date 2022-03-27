# wrk 压测工具

wrk 是一款针对 Http 协议的基准测试工具，它能够在单机多核 CPU 的条件下，使用系统自带的高性能 I/O 机制，如 epoll，kqueue 等，通过[多线程](https://so.csdn.net/so/search?from=pc_blog_highlight&q=多线程)和事件模式，对目标机器产生大量的负载。

## 一、wrk 安装

- 安装

  ```
  #下载wrk
  git clone https://github.com/wg/wrk
  #进入目录
  cd wrk
  #编译
  make
  ```

- 安装成功之后会在当前目录生成一个 wrk 的文件：

  ```
  ./wrk --help
  ```

- 增加软链接 （/home/ops/wengmq/wrk/wrk 为文件绝对路径）

  ```
  sudo ln -s /home/ops/wengmq/wrk/wrk /usr/local/bin
  ```

## 二、wrk 工具使用说明

- 查看说明：$ wrk --help

      Usage: wrk <options> <url>
        Options:
          -c, --connections <N>  Connections to keep open
          -d, --duration    <T>  Duration of test
          -t, --threads     <N>  Number of threads to use

          -s, --script      <S>  Load Lua script file
          -H, --header      <H>  Add header to request
              --latency          Print latency statistics
              --timeout     <T>  Socket/request timeout
          -v, --version          Print version details

        Numeric arguments may include a SI unit (1k, 1M, 1G)
        Time arguments may include a time unit (2s, 2m, 2h)

- 参数说明：

  ```
  -c				总的连接数（每个线程处理的连接数=总连接数/线程数）
  -d				测试的持续时间，如2s(2second)，2m(2minute)，2h(hour)
  -t				使用的线程数总数
  -s				加载的lua脚本文件
  -H				向请求头添加信息
  –latency	显示延迟统计
  –timeout	Socket/request 超时时间
  -v				显示版本详细信息
  ```

## 三、压测示例

- **示例**：使用 8 个线程，200 个连接，测试持续时间 30s, 显示延迟统计信息，请求地址为http://www.bing.com
- wrk -t8 -c200 -d30s --latency "http://www.bing.com"

  - ```
    输出：
    Running 30s test @ http://www.bing.com
      8 threads and 200 connections
      Thread Stats   Avg      Stdev     Max   +/- Stdev
        Latency    46.67ms  215.38ms   1.67s    95.59%
        Req/Sec     7.91k     1.15k   10.26k    70.77%
      Latency Distribution
         50%    2.93ms
         75%    3.78ms
         90%    4.73ms
         99%    1.35s
      1790465 requests in 30.01s, 684.08MB read
    Requests/sec:  59658.29
    Transfer/sec:     22.79MB
    ```

  - 以上使用 8 个线程 200 个连接，对 bing 首页进行了 30 秒的压测，并要求在压测结果中输出响应延迟信息。以下对压测结果进行简单注释：

    - ```
      Running 30s test @ http://www.bing.com （压测时间30s）
        8 threads and 200 connections （共8个测试线程，200个连接）
        Thread Stats   Avg      Stdev     Max   +/- Stdev
                    （平均值） （标准差）（最大值）（正负一个标准差所占比例）
          Latency    46.67ms  215.38ms   1.67s    95.59%
          （延迟）
          Req/Sec     7.91k     1.15k   10.26k    70.77%
          （处理中的请求数）
        Latency Distribution （延迟分布）
           50%    2.93ms
           75%    3.78ms
           90%    4.73ms
           99%    1.35s （99分位的延迟）
        1790465 requests in 30.01s, 684.08MB read （30.01秒内共处理完成了1790465个请求，读取了684.08MB数据）
      Requests/sec:  59658.29 （平均每秒处理完成59658.29个请求）
      Transfer/sec:     22.79MB （平均每秒读取数据22.79MB）
      ```

### 四、使用 Lua 脚本进行复杂测试

> 您可能有疑问了，你这种进行 GET 请求还凑合，我想进行 POST 请求咋办？而且我想每次的请求参数都不一样，用来模拟用户使用的实际场景，又要怎么弄呢？
>
> 对于这种需求，我们可以通过编写 Lua 脚本的方式，在运行压测命令时，通过参数 `--script` 来指定 Lua 脚本，来满足个性化需求。

#### 4.1 wrk 对 Lua 脚本的支持

wrk 支持在三个阶段对压测进行个性化，分别是启动阶段、运行阶段和结束阶段。每个测试线程，都拥有独立的 Lua 运行环境。

##### **启动阶段:**

```jsx
function setup(thread)
```

在脚本文件中实现 setup 方法，wrk 就会在测试线程已经初始化，但还没有启动的时候调用该方法。wrk 会为每一个测试线程调用一次 setup 方法，并传入代表测试线程的对象 thread 作为参数。setup 方法中可操作该 thread 对象，获取信息、存储信息、甚至关闭该线程。

```jsx
thread.addr             - get or set the thread's server address
thread:get(name)        - get the value of a global in the thread's env
thread:set(name, value) - set the value of a global in the thread's env
thread:stop()           - stop the thread
```

##### **运行阶段：**

```tsx
function init(args);
function delay();
function request();
function response(status, headers, body);
```

- `init(args)`: 由测试线程调用，只会在进入运行阶段时，调用一次。支持从启动 wrk 的命令中，获取命令行参数；
- `delay()`： 在每次发送请求之前调用，如果需要定制延迟时间，可以在这个方法中设置；
- `request()`: 用来生成请求, 每一次请求都会调用该方法，所以注意不要在该方法中做耗时的操作；
- `response(status, headers, body)`: 在每次收到一个响应时被调用，为提升性能，如果没有定义该方法，那么 wrk 不会解析 `headers` 和 `body`；

##### **结束阶段：**

```bash
function done(summary, latency, requests)
```

done() 方法在整个测试过程中只会被调用一次，我们可以从给定的参数中，获取压测结果，生成定制化的测试报告。

**自定义 Lua 脚本中可访问的变量以及方法：**

变量：wrk

```go
wrk = {
    scheme  = "http",
    host    = "localhost",
    port    = 8080,
    method  = "GET",
    path    = "/",
    headers = {},
    body    = nil,
    thread  = <userdata>,
  }
```

以上定义了一个 `table` 类型的全局变量，修改该 `wrk` 变量，会影响所有请求。

方法：

1. `wrk.fomat`
2. `wrk.lookup`
3. `wrk.connect`

上面三个方法解释如下：

```bash
function wrk.format(method, path, headers, body)

    wrk.format returns a HTTP request string containing the passed parameters
    merged with values from the wrk table.
    # 根据参数和全局变量 wrk，生成一个 HTTP rquest 字符串。

function wrk.lookup(host, service)

    wrk.lookup returns a table containing all known addresses for the host
    and service pair. This corresponds to the POSIX getaddrinfo() function.
    # 给定 host 和 service（port/well known service name），返回所有可用的服务器地址信息。

function wrk.connect(addr)

    wrk.connect returns true if the address can be connected to, otherwise
    it returns false. The address must be one returned from wrk.lookup().
    # 测试给定的服务器地址信息是否可以成功创建连接
```

#### 4.2 通过 Lua 脚本压测示例

**调用 POST 接口：**

```bash
wrk.method = "POST"
wrk.body   = "foo=bar&baz=quux"
wrk.headers["Content-Type"] = "application/x-www-form-urlencoded"
```

注意: wrk 是个全局变量，这里对其做了修改，使得所有请求都使用 POST 的方式，并指定了 body 和 Content-Type 头。

**自定义每次请求的参数：**

```jsx
request = function()
   uid = math.random(1, 10000000)
   path = "/test?uid=" .. uid
   return wrk.format(nil, path)
end
```

在 request 方法中，随机生成 1~10000000 之间的 uid，并动态生成请求 URL.

**每次请求前，延迟 10ms:**

```bash
function delay()
   return 10
end
```

**请求的接口需要先进行认证，获取 token 后，才能发起请求，咋办？**

```ruby
token = nil
path  = "/auth"

request = function()
   return wrk.format("GET", path)
end

response = function(status, headers, body)
   if not token and status == 200 then
      token = headers["X-Token"]
      path  = "/test"
      wrk.headers["X-Token"] = token
   end
end
```

上面的脚本表示，在 token 为空的情况下，先请求 `/auth` 接口来认证，获取 token, 拿到 token 以后，将 token 放置到请求头中，再请求真正需要压测的 `/test` 接口。

**压测支持 HTTP pipeline 的服务：**

```ruby
init = function(args)
   local r = {}
   r[1] = wrk.format(nil, "/?foo")
   r[2] = wrk.format(nil, "/?bar")
   r[3] = wrk.format(nil, "/?baz")

   req = table.concat(r)
end

request = function()
   return req
end
```

通过在 init 方法中将三个 HTTP 请求拼接在一起，实现每次发送三个请求，以使用 HTTP pipeline。

## 参考：

- https://blog.csdn.net/qq_36532540/article/details/106697503
- https://www.jianshu.com/p/81136edd6333
