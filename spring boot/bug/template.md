## 问题描述
- Error resolving template\"upload\", template might not exist or might not be accessible by any of the configured Templat...
- 写接口的时候找我controller里的路径总是出错

## 解决方法
- 把@Controller改成@RestController
> 下面用@RequestMapping(value="",method=..)  不知道这个用不用改，改了上面应该就好了

## 遗留问题
- @Controller和@RestController的区别
- @RequestMapping和@GetMapping的区别

## 解决历程
> 网上都是什么多了/,是静态文件错，花费很长时间也很崩溃
