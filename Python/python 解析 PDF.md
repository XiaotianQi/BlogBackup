通过第三方库 `pdfminer3k` 解析 PDF。

涉及相关功能如下：

* PDFParser（文档分析器）
* PDFDocument（文档对象）
* PDFResourceManager（资源管理器）
* PDFPageInterpreter（解释器）
* PDFPageAggregator（聚合器）
* LAParams（参数分析器）

主要分为三块内容：构造文档对象，解析文档对象，提取所需内容。

![各个类之间关系](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/PDF-1.png)

![实现流程](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/PDF-2.jpg)

解析单个 `pdf` 中的文本内容，保存到 `txt` 文件中。

```python
# _*_coding:utf-8_*_
import os
from pdfminer.pdfparser import PDFParser, PDFDocument
from pdfminer.pdfinterp import PDFResourceManager, PDFPageInterpreter
from pdfminer.converter import PDFPageAggregator
from pdfminer.layout import LTTextBoxHorizontal, LAParams
from pdfminer.pdfinterp import PDFTextExtractionNotAllowed
from PyPDF2 import PdfFileReader, PdfFileWriter

'''
解析单个 pdf 中的文本内容，保存到 txt 文件中
'''

def pdfParse(pdfPath, outPath='outfile.txt', password=''):
    fp = open(pdfPath, 'rb')
    # 创建 pdf 文档分析器
    praser = PDFParser(fp)
    # 创建 PDF文档
    doc = PDFDocument()
    # 连接 分析器与文档对象
    praser.set_document(doc)
    doc.set_parser(praser)
    # 验证密码，如果没有密码，就使用空字符串
    doc.initialize(password)

    # 使用 pypdf2 获取总页数，显示转换进度
    pdfFile = PdfFileReader(fp)
    pageCount = pdfFile.getNumPages()
    count = 0
    print('Begin: ', pdfPath)

    # 检测文档是否提供 txt 转换，不提供就忽略
    if not doc.is_extractable:
        raise PDFTextExtractionNotAllowed
    else:
        # 创建 PDF 资源管理器
        rsrcmgr = PDFResourceManager()
        # 创建 参数分析器
        laparams = LAParams()
        # 创建 创建聚合器
        device = PDFPageAggregator(rsrcmgr, laparams=laparams)
        # 创建 PDF 解释器
        interpreter = PDFPageInterpreter(rsrcmgr, device)

        # 循环遍历列表，使用 .get_pages() 方法获取 page 列表
        for page in doc.get_pages():
            # 利用解释器的 process_page()方法解析读取单独页数
            interpreter.process_page(page)
            # layout 是一个 LTPage 对象，存放着 page 解析出的各种对象
            # 一般包括LTTextBox, LTFigure, LTImage, LTTextBoxHorizontal等等,
            # 想要获取文本就获得对象的text属性，
            # 使用聚合器get_result()方法获取页面内容
            layout = device.get_result()
            count += 1
            print('Page:{}/{}'.format(count, pageCount))
            for x in layout:
                if (isinstance(x, LTTextBoxHorizontal)):
                    with open(outPath, 'a') as f:
                        results = x.get_text()
                        f.write(results + '\n')
    fp.close()
    print('Well Done: ', outPath)

'''
解析多个 pdf 中的文本内容，保存到txt文件中
'''

def pdfFolder(folder, password=''):
    # 获取指定目录下面的所有文件
    files = os.listdir(folder)
    # 获取pdf类型的文件放到一个列表里面
    pdfFiles = [f for f in files if f.endswith(".pdf")]
    for pdfFile in pdfFiles:
        pdfPath = os.path.join(folder, pdfFile)
        # 输出路径和文件格式
        outPath = pdfFile.split(r'.')[0] + '.txt'
        pdfParse(pdfPath, outPath, password)
        print(outPath)

def splitPdf(pdfPath):
    pdfFile = PdfFileReader(open(pdfPath, 'rb'))
    pageCount = pdfFile.getNumPages()
    print(pageCount)
    for page in range(pageCount):
        pdfOut = PdfFileWriter()
        pdfOut.addPage(pdfFile.getPage(page))
        pdfOutPath = pdfPath.split(r'.')[0] + str(page) + '.pdf'
        pdfOut.write(open(pdfOutPath, 'wb'))
        outPath = pdfPath.split(r'.')[0] + str(page) + '.txt'
        pdfParse(pdfPath=pdfOutPath, outPath=outPath)
        os.remove(pdfOutPath)


if __name__ == '__main__':
    pdfParse('wtest.pdf')

```

以上代码功能缺陷：

* 只使用转换文本功能，其他未涉及；
* 无法转换页数过多的 PDF 文件。
  

***

参考：

https://www.cnblogs.com/wj-1314/p/9429816.html