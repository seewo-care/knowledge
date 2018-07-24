> WebUploader是由Baidu WebFE(FEX)团队开发的一个简单的以HTML5为主，FLASH为辅的现代文件上传组件。在现代的浏览器里面能充分发挥HTML5的优势，同时又不摒弃主流IE浏览器，沿用原来的FLASH运行时，兼容IE6+，iOS 6+, android 4+。两套运行时，同样的调用方式，可供用户任意选用。

## 需求
项目需求是通过点击一个按钮使Modal出现，其中Modal中有文件上传按钮。点击该按钮可以自动上传文件。接入WebUploader（以下简称WU）后，遇到了一个问题。

## 现象
配置中的`#picker`元素没有被替换，WU渲染的文件上传按钮没有显示出来。

## 分析
官网给的配置如下：
```
var uploader = WebUploader.create({

    // swf文件路径
    swf: BASE_URL + '/js/Uploader.swf',

    // 文件接收服务端。
    server: 'http://webuploader.duapp.com/server/fileupload.php',

    // 选择文件的按钮。可选。
    // 内部根据当前运行是创建，可能是input元素，也可能是flash.
    pick: '#picker',

    // 不压缩image, 默认如果是jpeg，文件上传前会压缩一把再上传！
    resize: false
});
```
指定pick属性，WU将对页面中`#picker`元素进行一系列处理，最后生成如下的dom。
```
<span id="picker" class="webuploader-container">
    <div class="webuploader-pick">xxxx</div>
    <div id="rt_rt_1cj501c0fi8k1b2r1knh1dbt5t01" style="position: absolute; top: 0px; left: 0px; width: 1px; height: 1px; overflow: hidden;">
        <input type="file" name="file" class="webuploader-element-invisible" multiple="multiple">
        <label style="opacity: 0; width: 100%; height: 100%; display: block; cursor: pointer; background: rgb(255, 255, 255);"></label>
    </div>
</span>
```
可以看到，WU生成了一个`<input type="file"/>`dom节点，通过这个dom节点可以实现自动上传文件（需要设置auto属性为true）。

然而这个dom节点父元素div最初的宽高是1px的，以致于在页面无法点击到。查看WU源码（v0.1.5），其中看到：
```
refresh: function() {
    var shimContainer = this.getRuntime().getContainer(),
        button = this.options.button,
        width = button.outerWidth ?
                button.outerWidth() : button.width(),
    
        height = button.outerHeight ?
                button.outerHeight() : button.height(),
    
        pos = button.offset();
    
    width && height && shimContainer.css({
        bottom: 'auto',
        right: 'auto',
        width: width + 'px',
        height: height + 'px'
    }).offset( pos );
}        
```
`button`就是页面中生成的`.webuploder-pick`元素，`shimContainer`就是页面中生成的`#rt_rt_1cj501c0fi8k1b2r1knh1dbt5t01`。代码的意思是用`.webuploder-pick`的宽高去重置`#rt_rt_1cj501c0fi8k1b2r1knh1dbt5t01`的宽高，让其显示出来。

项目在这里出现了问题，通过debug得知，这里的获取的width和height为0。以致于WU生成的元素无法显示，从而就无法操作。经过一番探查，终于找到了原因。

**因为Modal一开始设置的样式中`display:none`。而`.webuploder-pick`是Modal的子元素。`display:none`的元素渲染引擎是不会渲染的。所以开始的时候`.webuploder-pick`的宽高为0。直到打开了Modal后，`.webuploder-pick`才有了宽高。但是这时候WU已经渲染过了，所以最终`#rt_rt_1cj501c0fi8k1b2r1knh1dbt5t01`的宽高还是0，页面没有显示。**

## 解决方案
知道了原因，解决办法是去掉`display:none`元素，取而代之的是`visibility: hidden`。`visibility: hidden`的元素渲染引擎是会渲染的，只是没显示出来。这时候WU渲染的时候能获取到`.webuploder-pick`的宽高，从而`#rt_rt_1cj501c0fi8k1b2r1knh1dbt5t01`也有了宽高，就显示在页面了。当通过按钮需要显示Modal的时候，就设置`visibility: visible`。

以上就是WebUploader渲染元素不可见的原因以及解决办法。回顾来看，其实原因很简单，只是开始都没想到那个点。

