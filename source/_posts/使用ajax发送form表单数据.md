---
title: 使用ajax发送form表单数据
tags:
  - ajax
  - jQuery
categories: ajax
abbrlink: ca78ff6a
date: 2017-04-08 11:41:12
---

### 使用ajax向后台发送json格式的form表单数据 

#### 使用场景是在vue的一个组件之中，前端通过form表单组合数据，而后台需要 将form表单 进行json格式的转化

1. processData：processData 默认为true，当设置为false的时候,jQuery ajax 提交的时候不会序列化 data，而是直接使用data
  举例说明；设置为true 会进行序列化。{ width:1680, height:1050 }参数对象序列化为width=1680&height=1050这样的字符串。
2. contentType：默认值: "application/x-www-form-urlencoded"。发送信息至服务器时内容编码类型 适用于大多数情况。
```javascript
s = jQuery.ajaxSetup( {}, options ),
// Convert data if not already a string
if ( s.data && s.processData && typeof s.data !== "string" ) {
    s.data = jQuery.param( s.data, s.traditional );
}
序列化一个 key/value 对象：
var params = { width:1900, height:1200 };
var str = jQuery.param(params);
$("#results").text(str);
结果:  width=1680&height=1050
``` 



```javascript
                var _this = this;
                this.$message.modal({
                    header: '导出到生成平台',
                    body: ` <form action="https://10.10.11.105:9001/genplatform-rest/api/upload/projectmeta" id='exportli' method='post' enctype='multipart/form-data'>
                    <button>用户名</button> <input type="text" name="userName" required><br>
                    <button>密码</button> <input type="text" name="passWord" required><br>
                    <button>用户组列表</button> <input type="text" name="groupName" required><br>
                    <button>项目列表</button> <input type="text" name="projectName" required><br>
                    <input type="text" name="projectMeta"  hidden id='file'>
                </form>`,
                    primaryClick: () => {
                        
                        $.ajax({
                            url: `${_this.appConfig.apiBase}project/download/${_this.$route.params.id}`,
                            type: 'get',
                            cache: false,
                            processData: false,
                            contentType: false,
                            success: (data) => {
                                var fileInput = document.getElementById('file');
                                fileInput.value = data;
                                let formData = $('#exportli').serialize();
                                console.log(formData);
                                console.log(JSON.stringify(formData));
                                $.ajax({
                                    url: 'https://10.10.11.105:9001/genplatform-rest/api/upload/projectmeta',
                                    type: "POST",
                                    data: formData,
                                    cache: false,
                                    processData: false,
                                    contentType: 'application/x-www-form-urlencoded',
                                    success: (data) => {
                                        if (data.code === 0) {
                                            this.$message.hide();
                                            this.$message.alert('导出到生成平台成功！');
                                        } else {
                                            this.appConfig.showErrorAlert(data.code);
                                        }
                                    },
                                    error: (data) => {
                                        this.appConfig.showErrorAlert();
                                    }
                                })
                            },
                            error: (data) => {
                                this.appConfig.showErrorAlert();
                            }
                        })

                    }
                })
            }
``` 
