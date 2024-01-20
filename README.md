# HttpClientFactory

### Why to use ?
<p>
  HttpClient has been designed to be re-used for multiple calls. Even across multiple threads. The HttpClientHandler has Credentials and Cookies that are intended to be re-used across calls. Having a new HttpClient instance requires re-setting up all of that stuff. Also, the DefaultRequestHeaders property contains properties that are intended for multiple calls. Having to reset those values on each request defeats the point.</p>
<p>
Another major benefit of HttpClient is the ability to add HttpMessageHandlers into the request/response pipeline to apply cross cutting concerns. These could be for logging, auditing, throttling, redirect handling, offline handling, capturing metrics. All sorts of different things. If a new HttpClient is created on each request, then all of these message handlers need to be setup on each request and somehow any application level state that is shared between requests for these handlers also needs to be provided.</p>
<p>
The more you use the features of HttpClient, the more you will see that reusing an existing instance makes sense.</p>
<p>
However, the biggest issue, in my opinion is that when a HttpClient class is disposed, it disposes HttpClientHandler, which then forcibly closes the TCP/IP connection in the pool of connections that is managed by ServicePointManager. This means that each request with a new HttpClient requires re-establishing a new TCP/IP connection.</p>
<p>
From my tests, using plain HTTP on a LAN, the performance hit is fairly negligible. I suspect this is because there is an underlying TCP keepalive that is holding the connection open even when HttpClientHandler tries to close it.</p>
<p>
On requests that go over the internet, I have seen a different story. I have seen a 40% performance hit due to having to re-open the request every time.</p>
<p>
I suspect the hit on a HTTPS connection would be even worse.</p>
<p>
My advice is to keep an instance of HttpClient for the lifetime of your application for each distinct API that you connect to.
</p>
