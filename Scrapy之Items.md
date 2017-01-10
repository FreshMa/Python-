##Scrapy之Items

爬取的目的就是从非结构化资源中提取处结构化的二数据。Scrapy爬虫可以以python字典的形式返回提取出的数据，但可能在大型项目中会缺乏结构。

Scrapy提供了 **Item** 类，来定义输出的数据格式。它提供了类字典的API用来声明可用的域。

###声明Items

可以使用类定义语法以及 **Field** 对象来声明Item，如下：

	import scrapy
	class Product(scrapy.Item):
		name = scrapy.Field()
		price = scrapy.Field()
		last_update = scrapy.Field(serializer = str)

###Item域

**Field** 对象是用来指定每一个域的元数据（metadata）。例如上例中last_updata域的serializer函数。**Field** 对象声明的item并没有作为类的属性来保持。它们可以通过 **Item.fields** 属性来访问。

###使用Items
	>>> product = Product(name = 'Desktop PC',price = 1000)
	>>> print product
	Product(name = 'Desktop PC',price = 1000)
	
	>>> product['name']
	Desktop PC
	>>> product.get('name')
	Desktop PC

	>>> 'name' in product
	True

	>>> product['last_updated'] = 'today'
	>>> product['last_updated']
	today

	>>> product.keys()
	['price','name']
	>>> product.items()
	[('price',1000),('name','Desktop PC')]

##Item Pipeline

当爬虫爬取出一个item后，它被送到Item Pipeline（管道）中，该管道通过一些顺序执行的组件处理item。

每一个item管道组件是实现了一个简单方式的Python类。它们接收一个item，然后对其执行某些动作，并决定是继续将其通过管道发送还是丢弃。

典型的item管道应用有：

- 清洗HTML数据
- 验证抓取的数据（检查item是否包含某些域）
- 查重并去重
- 将抓取的item存储到数据库中

###编写item 管道
一个item管道组件是一个必须实现 **process_item** 方法的类：

> process\_item(_self,item,spider_)

该方法被每一个item管道组件调用并且必须返回一个带有数据的字典，**Item** 对象或者抛出一个 **DropItem** 异常。被丢弃的items不再被后续的任何管道组件处理。

参数:

- item（Item对象或者字典）-被抓取的item
- spider（Spider对象）-抓取item的爬虫

此外，item管道也可以实现下面这些方法：
> open\_spider(_self,spider_)

当参数中的spider爬虫打开时被调用
> close\_spider(_self,spider_)

当参数中的spider爬虫被关闭时被调用
>from\_crawler(_cls,crawler_)

如果存在，该类方法（classmethod）会被调用来从 **Crawler** 创建一个管道对象。它必须返回一个新的管道实例。

###item pipeline例子
	#价格验证以及丢弃没有价格的items
	from scrapy.exceptions import DropItem
	
	class PricePipeline(object):
		vat_factor = 1.15
		def process_item(self,item,spider):
			if item['price']:
				if item['price_excludes_vat']:
					item['price'] = item['price']*self.vat_factor
				return item
			else:
				raise DropItem('Missing price in %s'%item)

#
	#将item写到json文件中
	import json
	class JsonWriterPipeline(object):
		def __init__(self):
			self.file = open('items.jl','wb')
		
		def process_item(self,item,spider):
			line = json.dumps(dict(item))+'\n'
			self.file.write(line)
			return item

#
	#重复元素过滤
	form scrapy.exceptions import DropItem
	
	class DuplicatesPipeline(object):
		
		def __init__(self):
			self.ids_seen = set()
		
		def process_item(self,item,spider):
			if item['id'] in self.ids_seen:
				raise DropItem(Duplicate item found:%s'%item)
			else:
				self.ids_seen.add(item['id'])
				return item

### 开启Item管道组件
为了开启Item管道组件，必须将它的类添加到 **ITEM_PIPELINES** 设置中，如下所示：

	ITEM_PIPELINES = {
		'myproject.pipelines.PricePipeline': 300,
		'myproject.pipelines.JsonWriterPipeline':800,
	}
其中的整数代表为该类分配的运行顺序，数字越低越先执行，通常被定义在0-1000范围内
