## 文件上传和下载api

### 下载
文件下载其实就是普通的api，直接给前端文件位置就可以了，就不多说了

### 上传
- POST
- form-data
- 用@RequestParam(value = "file",required = true)  MultipartFile file就可以接受到了
- 具体实现
``` java 
@RequestMapping(value = "/upload",method = RequestMethod.POST)
    public BaseApi upload (HttpServletRequest request,
                           @RequestParam(value = "file",required = true) MultipartFile file,
                           @RequestParam(value = "tag") int tag) throws Exception{
   log.info("进入了upload,tag是 "+tag);
    try{
        String filename = file.getOriginalFilename();
        String path = request.getServletContext().getRealPath("/images/");
        log.info("文件名字是 "+filename+" 文件路径是 "+path);
        File filepath = new File(path,filename);
        //判断路径是否存在，如果不存在就创建一个
        if (!filepath.getParentFile().exists()) {
            filepath.getParentFile().mkdirs();
        }
        //将上传文件保存到一个目标文件当中
        file.transferTo(new File(path + File.separator + filename));
        //返回文件的路径
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("path",path+File.separator+filename);
        return new BaseApi("ok",0,jsonObject);
    }catch (Exception e){
        e.printStackTrace();
        return new BaseApi("error",1,null);
    }

}

```

> 传入tag和file成功返回文件路径
