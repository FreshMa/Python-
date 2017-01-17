#Python验证码识别
验证码识别可以归类于图像识别方面的问题，验证码识别的一般解决思路是图片降噪，图像分割，图像文本输出。这些东西要是自己实现的话可能有一定难度。所幸，如果是为了解决问题，我们可以不用关心它具体如何实现，而是直接去使用现成的工具，这里使用的工具主要是**Pytesseract**。

##pytesseract的安装
pytesseract依赖于PIL和tesseract-ocr，其中前者是图像处理库，后者是google的ocr识别引擎。

1. PIL安装

		pip install PIL
2. tesseract-ocr安装

	tesseract-ocr的项目地址是：[Github](https://github.com/tesseract-ocr)。

	它本身没有提供新版本的windows安装文件，但是有第三方版本[*UB Mannheim*](https://github.com/UB-Mannheim/tesseract/wiki)，下载安装即可。

3. pytesseract安装

		pip install pytesseract

##pytesseract的使用
假设在当前工作目录下有一张简单的验证码图片为code.png，对它进行识别的python代码如下：

	import pytesseract
	from PIL import Image

	image = Image.open('code.png')

	#image.load()

	result = pytesseract.image_to_string(image)
	print result

然后我们就可以看到识别结果了。如果验证码图片足够清晰且工整的话，准确率还是不错的。但是如果稍微有些扭曲，识别结果就不那么准确了。还好，官方提供了训练功能，可以让我们自己提升识别效果。这篇博文有具体介绍——[Tesseract-ocr样本训练](http://blog.csdn.net/firehood_/article/details/8433077)。

##可能遇到的问题
1. PIL (ref:[LandGray](http://blog.csdn.net/c465869935/article/details/51438576))

	在执行上述代码时，可能会出现如下错误：

	 	AttributeError: 'NoneType' object has no attribute 'bands'
	具体原因可能是PIL实现上的一个小bug，有两种方法来解决：
	- 修改PIL/Image.py文件中的split函数，有一句self.load()注释掉 
	- 将上述代码中注释掉的代码取消注释
2. pytesseract(ref:[微寒](http://blog.csdn.net/supercooly/article/details/51314659))

	解决了1中的问题后，再次运行可能会出现如下错误：

		File "*\lib\subprocess.py", line 958, in _execute_child startupinfo)
		WindowsError: [Error 2] 
	这个错误的原因是因为没有将tesseract.exe没有加在环境变量PATH中。可以通过如下方式解决：找到pytesseract的安装目录（如果通过pip安装，那么就在%python根目录%/Lib/site-packages/pytesseract），打开pytesseract.py文件，找到如下文件：

		# CHANGE THIS IF TESSERACT IS NOT IN YOUR PATH, OR IS NAMED DIFFERENTLY
		tesseract_cmd = 'tesseract'
	将tesseract\_cmd的值修改为tesseract-ocr.exe的完整路径，例如我的装在D:/Program Files (x86)/Tesseract-OCR目录下，那么就修改为：

		tesseract_cmd = 'D:/Program Files (x86)/Tesseract-OCR/tesseract'

