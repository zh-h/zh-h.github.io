---
title: HTTP 请求 gzip 解压
date: 2018-04-10
tags: 
categories: [Java]
---
实际的 Web 项目中，会存在请求正文非常大的场景，例如发表长篇博客，提大量流水记录等等。这些数据如果能在本地压缩后再提交，就可以节省网络流量、减少传输时间。

一般采用的压缩方式是 gzip 请求正文会被gzip压缩过进行二进制传输，而 HTTP 头部依然是原始的文本，根据协议需要在头部注明编码 `Content-Encoding: gzip` 在服务端接收到请求后，如果支持这种格式的压缩，会把请求正文进行解压再处理。

参考[《如何压缩 HTTP 请求正文》](https://imququ.com/post/how-to-compress-http-request-body.html)

## 解压 gzip 请求
### Java
Java 内置了解压缩的流， 使用流的单项管道模式，可以把 gzip 的输入流，包装成原始的输入流。

```groovy
ByteArrayInputStream is = new ByteArrayInputStream(someGzipBytes)  // 放入压缩后的流
GzipInputStream gis = new GzipInputStream(is)
String content = gis.text
```

翻看了一遍 Spring 的接口实现，实在没有找到合适的方法进行`httpServletRequset.inputStream`的替换。使用自定义的`MessageConverter`也是比较麻烦，需要把原本的配置覆盖替换，不得知对其他转换器的影响。

另外一种方法就是使用 Servlet API 的 `Filter` 进行处理，在获取到 gzip 的请求正文时对他进行解压。
`GunzipFilter.groovy`

```groovy
/**
 * 处理gzip压缩过的请求体
 * <p>
 * 使用 web.xml 配置 filter
 * Spring Boot 需要在主类使用 @ServletComponentScan 才能加载
 */
@WebFilter(filterName = "gunzipFilter", urlPatterns = "/*")
public class GunzipFilter implements Filter {

    public static final Logger LOGGER = LoggerFactory.getLogger(GunzipFilter.class);

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        LOGGER.info("gunzip filter init");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        String requestURI = httpServletRequest.getRequestURI();
        LOGGER.info("request uri: {} Content-Encoding: {}", requestURI, httpServletRequest.getHeader("Content-Encoding"));
        if ("/".equals(requestURI)) {
            LOGGER.info("process gzip request");
            httpServletRequest = new GunzipInputStreamWrapper(httpServletRequest);
        }
        filterChain.doFilter(httpServletRequest, servletResponse);
    }

    @Override
    public void destroy() {

    }

    static class GunzipInputStreamWrapper extends HttpServletRequestWrapper {

        private ServletInputStream newServletInputStream;

        public GunzipInputStreamWrapper(HttpServletRequest request) {
            super(request);
        }

        @Override
        public ServletInputStream getInputStream() throws IOException {
            if (newServletInputStream == null) {
                ServletInputStream servletInputStream = super.getInputStream();
                GZIPInputStream gzipInputStream = new GZIPInputStream(servletInputStream);
                newServletInputStream = new ServletInputStream() {
                    @Override
                    public boolean isFinished() {
                        return true;
                    }

                    @Override
                    public boolean isReady() {
                        return true;
                    }

                    @Override
                    public void setReadListener(ReadListener readListener) {}

                    @Override
                    public int read() throws IOException {
                        return gzipInputStream.read();
                    }
                };
            }
            return newServletInputStream;
        }
    }
}

```

### OpenResty
OpenResty 可以通过使用 Lua 书写逻辑，可以导入使用 Lua 的类库，也可以通过 FFI 调用 C 语言编写的动态链接库。

需要下载 Lua 的 FFI zlib 库，然后通过 lua 脚本导入。

1. 下载 https://github.com/luapower/zlib/archive/master.tar.gz
2. 解压master.tar.gz
3. 将`linux64/libz.so` `zlib_h.lua` `zlib.lua`复制到`/usr/local/openresty/lualib/`
4. 在`/usr/local/openresty/lualib/`创建`gunzip.lua`文件
5. 在文件中输入
```
local ffi  = require "ffi"
local zlib = require "zlib"

local function reader(s)
    local done
    return function()
        if done then return end
        done = true
        return s
    end
end

local function writer()
    local t = {}
    return function(data, sz)
        if not data then return table.concat(t) end
        t[#t + 1] = ffi.string(data, sz)
    end
end

local encoding = ngx.req.get_headers()['Content-Encoding']
ngx.log(ngx.INFO, "encoding: "..encoding)

if encoding == 'gzip' or encoding == 'deflate' or encoding == 'deflate-raw' then
    ngx.req.clear_header('Content-Encoding');
    ngx.req.read_body()

    local body = ngx.req.get_body_data()

    if body then
        ngx.log(ngx.INFO, "unzip body")
        local write = writer()
        local map = {
            gzip = 'gzip', 
            deflate = 'zlib', 
            ['deflate-raw'] = 'deflate'
        }
        local format = map[encoding]
        zlib.inflate(reader(body), write, nil, format)
        ngx.req.set_body_data(write())
    end

```
6. 编辑`/usr/local/openresty/nginx/conf/nginx.conf`在`server`块插入一下内容
```
	# call request body to lua context
	location / {
	    # 非常重要，否则大文件解压因为被放到文件里导致读取的时候乱码
		client_body_buffer_size 2048k;  
		access_by_lua_file /usr/local/openresty/lualib/resty/gunzip.lua;
		proxy_pass http://192.168.150.226:8800;
    }

```
7. `/usr/local/openresty/nginx/sbin/nginx -s reload` 重载更新脚本。

OpenResty 映射对应需要请求的 URL 然后在对应的处理或者转发前执行 Lua 脚本，Lua 脚本判断请求类型读取请求体，并且使用 zlib 将其解压，然后再写回请求体，最后再把 `Content-Encoding` 清除，放置后端再做错误处理。

## 使用 gzip 压缩请求

### CURL
先用 gzip 工具压缩，然后 curl 命令行工具支持提交 gzip 的文件。
```bash
echo "{\"msg\":\"hello\"}" | gzip -c > data.txt.gz
curl -v --data-binary @data.txt.gz -H'Content-Type: application/json charset=UTF-8' -H'Content-Encoding: gzip' -X POST http://localhost:8800
```

### Java
Java 同样提供了内置的 gzip 工具类，可以使用 `GzipOutputStream` 将输出流替换成压缩过的输出流，然后将压缩后的流通过 Socket 提交。

OkHttP 提供自定义请求正文的方法，可以把请求正文的格式替换为 gzip
`GzipRequest.groovy`
```groovy
@Slf4j
class GzipRequest {


    static final MediaType JSON = MediaType.parse("application/json charset=utf-8")

    static
    final String CONTENT = new File('C:\\Users\\Administrator\\Documents\\data-flow.json').text

    static OkHttpClient client = new OkHttpClient.Builder()
            .addInterceptor(new GzipRequestInterceptor())
            .build()

    /**
     * @param args
     */
    public static void main(String[] args) {
        String content = '''{"msg":"hello"}'''
        content = CONTENT
        RequestBody body = RequestBody.create(JSON, content)
        Request request = new Request.Builder()
                .url('http://localhost:8800/?name=age')
                .post(body)
                .build()
        Response response = client.newCall(request).execute()
        String responseBody = response.body().string()
        log.info(responseBody)
    }

    final static class GzipRequestInterceptor implements Interceptor {
        @Override
        public Response intercept(Chain chain) throws IOException {
            Request originalRequest = chain.request()
            if (originalRequest.body() == null || originalRequest.header("Content-Encoding") != null) {
                return chain.proceed(originalRequest)
            }

            Request compressedRequest = originalRequest.newBuilder()
                    .header("Content-Encoding", "gzip")
                    .method(originalRequest.method(), gzip(originalRequest.body()))
                    .build()
            return chain.proceed(compressedRequest)
        }

        private RequestBody gzip(final RequestBody body) {
            return new RequestBody() {
                @Override
                public MediaType contentType() {
                    return body.contentType()
                }

                @Override
                public long contentLength() {
                    return -1 // We don't know the compressed length in advance!
                }

                @Override
                public void writeTo(BufferedSink sink) throws IOException {
                    BufferedSink gzipSink = Okio.buffer(new GzipSink(sink))
                    body.writeTo(gzipSink)
                    gzipSink.close()
                }
            }
        }
    }
}
```