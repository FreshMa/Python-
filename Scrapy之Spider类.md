#Scrapy之Spider类

爬虫的爬取工作可以概括为以下行为的循环：

1. 从生成初始的爬取请求开始，爬取第一个URL，并且指定一个回调函数，该回调函数会处理对应请求的响应。第一个请求通过调用 **start\_requests** 函数来获得，该默认函数会为 **start_urls** 中指定的 urls生成 Request。**parse** 函数是 Request 中默认的回调函数。
2. 在回调函数中，你可以分析响应（通常是网页），并返回带有提取出来数据的字典（dict），Item对象，Request对象，或者这些对象的迭代器。这些 Requests 同样也包含一个回调函数，并且会被Scrapy下载，再使用特定的回调函数来处理这些请求的响应。
3. 在回调函数中，通常使用 **Selectors** 来分析网页内容，并且使用分析得到的数据生成items。
4. 最后从爬虫返回的items通常被持久化到数据库中（使用**Item Pipeline**）或者写到文件中（使用**Feed Export**）

##scrapy.Spider
scrapy.Spider类是最简单的爬虫类。该类不提供任何特定的功能，仅仅提供一个默认的 **start\_requests()** 的实现，该函数从**start_urls** 属性中发送请求，并为获得的响应调用**parse**方法。

Spider类中有如下属性：
> name

爬虫的名字，需要是唯一的。
> allowed\_domains
 
可选的列表，包含爬虫被允许爬取的域名。
> start\_urls

爬虫开始爬取的站点URL列表。所以，它包含第一批被爬取的网页，之后的URL将从起始URL包含的数据中获得。

> crawler

该属性被**from\_crawler()**这个类方法在初始化类后设置，并关联到Crawler对象上
> from\_crawler(crawler,\*args,\*\*kwargs)

Scrapy使用这个类方法来创建你的爬虫。
> start\_reqeusts()

该方法必须使用初始的Requests返回一个可迭代的对象，以供爬虫爬取。当爬虫开启且没有特定的URL被指定时，这个方法就会被Scrapy调用。如果制订了特定的URL，那么Scrapy会使用 **make\_requests\_from\_url()** 方法来创建Requests。

这个方法只会被调用一次，所以使用生成器实现它是安全的。

该方法默认的实现时使用 **make\_requests\_from\_url()**方法来为**start\_urls**中的每一个URL生成请求

如果想改变爬虫开始爬取时的Requests（请求）时，需要重写这个方法。例如，如果需要从发送POST请求的登陆开始，你应当：

	class MySpiser(scrapy.Spider):
	name = 'myspider'
	def start_requests(self):
		return [scrapy.FormRequest('http://www.example.com/login',
				formdata = {'user':'john','pass':'secret'},
				callback = self.logged_in)]
	def logged_in(self,response):
		#提取接下来要爬取的链接，并为它们返回带有回调函数的Request
		pass
>make\_reqeusts\_from\_url(url)

接受一个URL参数并返回一个 **Request**对象（列表）的方法。该方法用在**start_request()** 方法中，用来构建初始请求。除非被覆盖，否则这个方法使用 **parse** 函数作为默认的回调函数。
>parse(response)

在没有指定回调函数的情况下，该方法是Scrapy默认的处理已下载响应的回调函数。

**parse**方法负责处理响应并返回抓取到的数据或者更多的待抓取URL。和其他Request回调函数一样，它也需要返回可迭代的 **Request** 或者 **Item** 对象

----------

###Example

	import scrapy
	from myproject.items import MyItem

	class Myspider(scrapy.Spider):
		name = 'example.com'
		allowed_domains = ['example.com']
		
		def start_requests(self):
			yield scrapy.Request('http://www.example.com/1.html',self.parse)
			yield scrapy.Request('http://www.example.com/2.html',self.parse)
			yield scrapy.Request('http://www.example.com/3.html',self.parse)

		def parse(self,response):
			for h3 in response.xpath('//h3').extract():
				yield MyItem(title=h3)
			
			for url in response.xpath('//a/@href').extract():
				yield scrapy.Request(url,callback = self.parse)
