你好，我是温铭。前面我们讲过，在 Lua 中， table 是唯一的数据结构。与之对应的一个事实是，共享内存字典 shared dict，是你在 OpenResty 编程中最为重要的数据结构。它不仅支持数据的存放和读取，还支持原子计数和队列操作。基于 shared dict，你可以实现多个 worker 之间的缓存和通信，以及限流限速、流量统计等功能。你可以把 shared dict 当作简单的 Redis 来使用，只不过 shared dict 中的数据不能持久化，所以你存放在其中的数据，一定要考虑到丢失的情况。数据共享的几种方式在编写 OpenResty Lua 代码的过程中，你不可避免地会遇到，在一个请求的不同阶段、不同 worker 之间共享数据的情况，还可能需要在 Lua 和 C 代码之间共享数据。所以，在正式介绍 shared dict 的 API 之前，先让我们了解一下，OpenResty 中常见的几种数据共享的方法；并学会根据实际情况，选择较为合适的数据共享方式。第一种是 Nginx 中的变量。它可以在 Nginx C 模块之间共享数据，自然的，也可以在 C 模块和 OpenResty 提供的 lua-nginx-module 之间共享数据，比如下面这段代码：location /foo { set $my_var ''; # this line is required to create $my_var at config time content_by_lua_block { ngx.var.my_var = 123; ... } }不过，使用 Nginx 变量这种方式来共享数据是比较慢的，因为它涉及到 hash 查找和内存分配。同时，这种方法有其局限性，只能用来存储字符串，不能支持复杂的 Lua 类型。第二种是 ngx.ctx，可以在同一个请求的不同阶段之间共享数据。它其实就是一个普通的 Lua 的 table，所以速度很快，还可以存储各种 Lua 的对象。它的生命周期是请求级别的，当一个请求结束的时候，ngx.ctx 也会跟着被销毁掉。下面是一个典型的使用场景，我们用 ngx.ctx 来缓存 Nginx 变量 这种昂贵的调用，并在不同阶段都可以使用到它：location /test {

location /foo {
set $my_var ''; # this line is required to create $my_var at config time
content_by_lua_block {
ngx.var.my_var = 123;
...
}
}
