---
title: 多个商品条目批量选中上架时判断售后模板是否为空
date: 2016-06-13 10:58:27
categories: [JQuery,checkbox]
tags: [JQuery,checkbox]
visitors:
---
### 解决
获取选中checkbox长度
获取选中条目对应非空售后模板长度
两者比较大小
{% codeblock lang:javascript %}
function getcheckAllSiblingsVal(name,cls){
	var prams = $("table input:checkbox[name='"+name+"']:checked").parent()
	            .parent().siblings("."+cls+"");
	var nameLength = $("table input:checkbox[name='"+name+"']:checked").length;
	var len = prams.length;
	var pramLength = 0;
	if(len>0){
		for(var i=0;i<len;i++){
			if(""!=prams.eq(i).text()){
				pramLength+=1;
			}
		}
		if(parseInt(pramLength)<parseInt(nameLength)){
			return 0;
		}else{
			return 1;
		}
	}else{
		return 0;
	}
}
{% endcodeblock %}

