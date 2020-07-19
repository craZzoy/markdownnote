
```
function getTestData() {
	var returnValue;
	if(window.showModalDialog == undefined){
		//谷歌
        var iWidth = 520;
        var iHeight = 600;
        var iTop = (window.screen.availHeight - 30 - iHeight) / 2;
        var iLeft = (window.screen.availWidth - 10 - iWidth) / 2;
        returnValue = window.open("/test/get", "弹出窗口", "width=" + iWidth + ", height=" + iHeight + ",top=" + iTop + ",left=" + iLeft + ",toolbar=no, menubar=no, scrollbars=no, resizable=no,location=no, status=no,alwaysRaised=yes,depended=yes");
	}else{
        returnValue = window.showModalDialog('/test/get',window,"dialogWidth:400px;dialogHeight:400px");
	}
    console.log(returnValue);
	//$("#test").val(returnValue);
	if(window.opener){
        null;
	}else{
        $("#test").val(returnValue);
	}
}
```
新版的谷歌浏览器不支持window.showModalDialog，window.open()方法返回得是一个window对象。
打开的窗口可通过window.opener调用父窗口。