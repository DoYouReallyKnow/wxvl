#  【$10,000】Arc 浏览器：UXSS+本地文件窃取+任意文件写入，路径穿越直通RCE！​​  
原创 骨哥说事  骨哥说事   2025-06-23 07:12  
  
<table><tbody><tr><td data-colwidth="557" width="557" valign="top" style="word-break: break-all;"><h1 data-selectable-paragraph="" style="white-space: normal;outline: 0px;max-width: 100%;font-family: -apple-system, system-ui, &#34;Helvetica Neue&#34;, &#34;PingFang SC&#34;, &#34;Hiragino Sans GB&#34;, &#34;Microsoft YaHei UI&#34;, &#34;Microsoft YaHei&#34;, Arial, sans-serif;letter-spacing: 0.544px;background-color: rgb(255, 255, 255);box-sizing: border-box !important;overflow-wrap: break-word !important;"><strong style="outline: 0px;max-width: 100%;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="color: rgb(255, 0, 0);"><strong><span style="font-size: 15px;"><span leaf="">声明：</span></span></strong></span><span style="font-size: 15px;"></span></span></strong><span style="outline: 0px;max-width: 100%;font-size: 18px;box-sizing: border-box !important;overflow-wrap: break-word !important;"><span style="font-size: 15px;"><span leaf="">文章中涉及的程序(方法)可能带有攻击性，仅供安全研究与教学之用，读者将其信息做其他用途，由用户承担全部法律及连带责任，文章作者不承担任何法律及连带责任。</span></span></span></h1></td></tr></tbody></table>#   
  
#   
  
****# 防走失：https://gugesay.com/archives/4472  
  
******不想错过任何消息？设置星标****↓ ↓ ↓**  
****  
#   
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlbXyV4tJfwXpicwdZ2gTB6XtwoqRvbaCy3UgU1Upgn094oibelRBGyMs5GgicFKNkW1f62QPCwGwKxA/640?wx_fmt=png&from=appmsg "")  
  
****  
在 Arc 浏览器公布了其漏洞赏金计划后，国外白帽小哥便开始了他的漏洞挖掘之旅，首先他利用逆向技能查看了浏览器的二进制文件：  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFuHPuFIfS0PiaxohHPjWoCdLGjAMtQynAVlibZLZayTA9QvqxR0e2BJibOg/640?wx_fmt=png&from=appmsg "")  
  
  
很快便发现了一些有趣的端点：  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFu6rt2t6TB0NPdYRrt9wqkqh1sHe5Qsg1tGG7zbMM8iaptLlYTibYRkCbA/640?wx_fmt=png&from=appmsg "")  
  
  
在浏览器中打开 URL：  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFuibWDKLpjXoNcSnxMCWMWo4icft1tl3Kq1sE92MGP6chQV20swukpylwg/640?wx_fmt=png&from=appmsg "")  
  
  
这是一个用于创建 boosts 的漂亮 UI，它只是带有一些自定义配置的扩展，可以在其中创建和编辑多种类型的 boosts，boosts 也将作为扩展自动安装，让我们看看里面的代码和可用路径：  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFuSAibDgyicvT3B9TX98WpjbeFW9JCG1fYUOeSvhohKlLWlQcUwsibnwxGw/640?wx_fmt=png&from=appmsg "")  
  
  
/play 端点很有趣，访问 arc://boost/play/test  
:  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFuxASr3jIHE3s4AicciaNqbMbS1ibDzVMnCxuNcRoibKQh01UJOgUHODjzHQ/640?wx_fmt=png&from=appmsg "")  
  
  
查看代码：  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFubbrSLWeGwRa8OHpbC4FUWtmojxpUEdT6CA7dUvWbpU5SmqD4VPC8Uw/640?wx_fmt=png&from=appmsg "")  
  
  
数值从 URL 中提取，然后使用 decompressFromEncodedURIComponent() 函数进行解压缩（假设它就像 Base64），然后传递给 JSON.parse()，之后再加载 boost。  
  
幸运的是，压缩机制也有 compressToEncodedURIComponent() 函数来测试我们的Payload，让我们创建一个简单的对象进行测试：  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFuqz4f7ic2d8HzqF9iaTeWkTTXRDPZb18jlVDGyiagqw8mYrY71craabb9w/640?wx_fmt=png&from=appmsg "")  
  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFu2IPqj0eUrRjb18j9RjyJZJ3SfftiaCVnr5QvKnxsaRIeOYn20jHxnjA/640?wx_fmt=png&from=appmsg "")  
  
  
我们遇到了另一个问题，即JSON必须首先是一个数组，尝试修复它 u=’[{“x”:”y”}]’  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFulUcSNhUf0K1WKDBSTVsc5tIUnSavQgXSzd6AT0KHgRC6YKiaErh2laA/640?wx_fmt=png&from=appmsg "")  
  
  
让我们从用户界面创建一个 boost，并在代码中添加一个断点来查看配置情况。  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFulz73jXUHZbeApH0dQghNuQ3Ykyh4ariaH9XxncBDz1jh4xpKx3sGDWQ/640?wx_fmt=png&from=appmsg "")  
  
  
替换 boost 的配置包括 3 个对象，也就是 3 个文件：content.js 是扩展的Content Script脚本；boost.config.json 是 boost 的设置，其中包含 boost 的名称、范围和描述等信息，这些信息稍后会被用户界面使用；manifest.json 就是 Manifest。让我们创建第一个 boost：  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFuGZWa3w9zeHGWGawqibTwkuWsHKibm7P2PneW0CsEVLbiaC3g7ialN44OFg/640?wx_fmt=png&from=appmsg "")  
  
  
点击 Install Boost：  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFup8UnZHT6rsia6gJdPfSVzs2jMaicZib9roxqnjJJ6Jich1toSfQIdsQ4hw/640?wx_fmt=png&from=appmsg "")  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFuEDEYrmJn9tAVYnRUVNW0cLPZR5YC06ZZYk5DJnV4PxAo00H9q7CMbA/640?wx_fmt=png&from=appmsg "")  
  
boost 将会被安装，并且在我们的库文件中会创建一个包含所有 3 个文件的新文件夹，通过调整参数可以发现文件 boost.config.json 仅由 UI 和 Arc 使用来显示有关 boost 的信息，并且每个权限和访问权限都在 manifest.json 中声明，**这意味着我们可以欺骗 UI 来显示一个看似无害的 boost，它只会改变 example.com 的背景颜色，而在manifest 文件中，我们声明了所有权限，并可以访问所有内容。**  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFuf2zkOX8StV6W567Rs6PyOqcU7jKgSObrUqX3yk4PgrWklrSnOoHc5Q/640?wx_fmt=png&from=appmsg "")  
  
  
这本身就是一个很大的漏洞，它允许我们欺骗 boosts UI 并安装恶意 boosts，从而可以访问所有内容，甚至是 file:// URI  
，这意味着不仅仅是 UXSS，还可以读取本地文件，这是普通扩展无法做到的，但白帽小哥发现，当安装一个具有所有 URL 完全权限的扩展时，它将可以访问系统上的所有文件：  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFugYYuB92bfcSPds03zsDMFZB5fKJ4bOvzRoHrpzgt1HZ11XJdCjzTHA/640?wx_fmt=png&from=appmsg "")  
  
  
为了提升漏洞影响，白帽小哥尝试更深入的研究。  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFu03InOhFkxH1Lamu4EhzWp9kphknt7zZVNYznOfOde5MQAk35PCDI5g/640?wx_fmt=png&from=appmsg "")  
  
  
当设置配置数组时，每个对象内都有两个有趣的值，即名称和路径，作为攻击者，首先想到的便是路径遍历尝试，所以把这两个值都改成../styles.css  
，然后安装 boost：  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFuRvhibUWARUsaVS7EgyQc132E3XoMoUTYHibtKJfzAddibTgOib7iadgqkng/640?wx_fmt=png&from=appmsg "")  
  
  
文件在扩展文件夹的父目录中被创建，这意味着使用../ 进行路径遍历，就可以创建具有任意内容和路径的任意文件。  
  
通过该操作，发现还可以覆盖文件并显示完整的内容，例如覆盖 /etc/passwd 来破坏机器或覆盖二进制，或者覆盖其它程序使用的 .py 和 .js 文件。  
  
为了测试安全，白帽小哥使用了一个更安全的路径 LaunchAgent， 它是一个包含一堆 plist 配置的文件夹，它将在用户登录或系统启动时运行，一但创建了恶意 plist 文件，它将下载一张图片，将其设置为受害者壁纸，并使用一系列命令打开终端和经典的计算器。  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFuLy9SibicNJd0HAgMeMQEt6zpiaD4kpJkT38Gb0LE5vLoJajeibwrHBXtJw/640?wx_fmt=png&from=appmsg "")  
  
  
将压缩数据和路径遍历一起插入到UI中，以便将其插入到LaunchAgents文件夹中：  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFuQePnwKYJsfl4t7X6LiacgOSDXdiaS5UGxjIdj0BwgkDbzPv3ibWpjS24Q/640?wx_fmt=png&from=appmsg "")  
点击安装：  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFu8vB4FbdxtQjn6QENaoVeds7wPR5VTB5oPGcED2ILLNraNe9gpH4bxg/640?wx_fmt=png&from=appmsg "")  
  
  
成功插入恶意 plist，当系统重启时，代码成功执行：  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFuITbgLt7tqsYqJLy5quwlh4vNfpic3GcibULNRXR2Hn1iay0TS766WnWOA/640?wx_fmt=png&from=appmsg "")  
  
  
但仍然存在一个问题，即如何将 arc:// uri  
 发送给受害者，让受害者点击“安装”，该 URI 很特殊，无法通过正常的 HTTP 导航进入，用户必须将其粘贴到地址栏中并按下 Enter 或浏览器中其他具有更高权限的组件。  
  
我们可以创建一个扩展并将其上传到Chrome WebStore，有权使用Chrome.tabs.tabs.create导航到URI，然后当用户单击安装时，实现代码执行，但这也需要大量的用户交互，是否有更简单的实现方式呢？  
  
在浏览 Arc 时，白帽小哥看到了一个名为 Easels 的功能，它允许将其他网站嵌入到白板中并与其他人共享，嵌入功能在某种程度上类似于 iframe，但它允许我们嵌入包括arc://在内的每个页面：  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFuOd7eibiaZic1OGzLicspEbztibiab0uk53ibzoY6LrEd4lojpnv5NnicaD8b2Q/640?wx_fmt=png&from=appmsg "")  
  
  
我们可以分享一个类似https://arc.net/e/ED548B06-…  
的URL，受害者点击框架后就会安装boost，意味着只需要2次点击，而在用户界面中，启动程序只会更改 example.com 的内容，而受害者并不知道它拥有完全权限，下图概括了完整的攻击流程：  
  
![image.png](https://mmbiz.qpic.cn/sz_mmbiz_png/hZj512NN8jlI9wNBRf2cyoyIEUq75pFuoK6fqliczM4g3pMq9bKKdDf1PTNibrorcGP66BbkS7dLAHLa0E4hRriag/640?wx_fmt=png&from=appmsg "")  
  
  
后续，白帽小哥还发现了一种更加简单的方法，可以在受害者浏览器中打开特殊 URI，这将使攻击变为仅需一次单击的用户交互，即受害者访问 attack.com -> 重定向到 arc://boost/play/… -> 单击安装 -> pwn。  
  
最终白帽小哥获得了1 万美元的赏金奖励。  
  
原文：https://medium.com/@renwa/arc-browser-uxss-local-file-read-arbitrary-file-creation-and-path-traversal-to-rce-b439f2a299d1  
  
- END -  
  
  
**加入星球，随时交流：**  
  
**********（会员统一定价）：128元/年（0.35元/天）******  
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hZj512NN8jnMJtHJnShkTnh3vR3fmaqicPicANic6OEsobrpRjx5vG6mMTib1icuPmuG74h2bxC4eP6nMMzbs5QaSlw/640?wx_fmt=jpeg&from=appmsg "")  
  
**感谢阅读，如果觉得还不错的话，欢迎分享给更多喜爱的朋友～**  
  
