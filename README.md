# 普通高等学校招生录取自动查询脚本

这个人很懒，还没有写详细文档。  

目前仅支持托管在Cloudflare Worker，如有本地化需求请自行修改。  
本脚本仅适用于海南省普通高等学校招生查询系统，其他地区系统请自行适配。  
请合理配置Corn执行时间，建议在15分钟以上。  
本脚本及其作者不对任何不当使用本脚本而造成的服务器拥堵等不良后果负责。

指路：[邮件API](https://developers.cloudflare.com/workers/tutorials/send-emails-with-resend)  
温馨提示：部署在cloudflare前要先使用Webpack将Resend打包
