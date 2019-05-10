# 实践中遇到的一些问题

## gitbook3.2.3在windows下gitbook serve起不来

修改~/.gitbook/versions/3.2.3/lib/output/website/copyPluginAssets.js这个文件中的confirm 的值为false