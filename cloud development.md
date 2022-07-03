初始化云环境

app.js中

```javascript
App({
    onLaunch(){
        wx.cloud.init({
            env:'cloud1-3ge8571r470d2c49'
        })
    }
})
```



## 云数据库

### 1.添加数据add()

```javascript
 addData(){
         wx.cloud.database().collection('yun_users').add({
            data:{
                name:'spiderMan',
                num:'2222',
                gender:'男',
                age:'18',
            }.then(res=>{
                console.log(res);
                wx.showToast({
                  title: '添加成功！',
                })
            })
        })
    }
```

### 2.查询数据

>查询所有数据get()
>
>```javascript
>getAllData(){
>    console.log(getApp().globalData.openid);
>    wx.cloud.database().collection('yun_users').get()
>    .then(res=>{
>        console.log(res);
>        this.setData({
>            dataList:res.data
>        })
>    })
>}
>```
>
>
>
>查询单条数据doc()
>
>条件查询where()

### 3.删除数据remove()

```javascript
    delete(e){
        var id = e.currentTarget.dataset.id
        wx.cloud.database().collection('yun_users').doc(id).remove()
        .then(res=>{
            console.log(res);
            wx.showToast({
              title: '删除成功！',
            })
        })
    }
```



## 云函数

###1.云函数文件配置

project.config.json里

"cloudfunctionRoot" : "cloud/"

### 2.新建云函数

在cloud文件夹上点击右键，新建node.js云函数，自定义命名即可创建云函数

### 3.调用云函数

```javascript
//在app.js调用云函数
wx.cloud.callFunction({
    name:'getUserOpenid'
}).then(res=>{
    console.log(res.result.openid);
    this.globalData.openid = res.globalData.openid
}),

    //在页面中调用云函数
wx.cloud.callFunction({
    name:'getUserOpenid'
}).then(res=>{
    console.log(res.result.openid);
    getApp().globalData.openid = res.globalData.openid  //唯一区别
})
```

####1.用云函数向数据库添加数据

```javascript
//云函数入口
exports.main = async (event, context) => {
    const wxContext = cloud.getWXContext()

   return cloud.database().collection('yun_users').add({
       data:{
        name:event.name//通过event接受传递的参数
       }.then(res=>{
           console.log(res);
       })
   })
}


//调用云函数
	addUser(){
        wx.cloud.callFunction({
            name:'yun-add',
            data:{				// 向云函数传递数据
                 name:'spiderMan'
            }
        }).then(res =>{
            console.log(res);
        })
    }
```



#### 2.用云函数删除数据

```javascript
// 云函数入口函数
exports.main = async (event, context) => {
    const wxContext = cloud.getWXContext()
    
    return cloud.database().collection('yun_users').doc(event.id).remove()
}

//调用云函数
    deleteUser(){
        wx.cloud.callFunction({
            name:'yun-delete',
            data:{
                id:'d4107ab16246b06e039a62d82b4d8c48'
            }
        }).then(res => {
            console.log('删除成功！');
        })
    }
```



#### 3.用云函数更新数据

```javascript
//云函数入口

exports.main = async (event, context) => {
    const wxContext = cloud.getWXContext()
    
    return cloud.database().collection('yun_users').doc(event.id).update({
        data:{
            name:event.name,
            age:event.age
        }
    })
}
   
   //云函数调用
   updateUser(){
        wx.cloud.callFunction({
            name:'yun-update',
            data:{
                id:'efbc6d716246b85d0364b5e745bc15e9',
                name:'hawkEye',
                age:'30'
            }
        }).then(res => {
            console.log('更新成功！');
        })
    }
```



#### 4.查询

```javascript
// 云函数入口函数
exports.main = async (event, context) => {
    const wxContext = cloud.getWXContext()
   
    return cloud.database().collection('yun_users').where({
        name:event.name
    }).get()
    
}
//调用
    getUsers(){
        wx.cloud.callFunction({
            name:'yun-get',
            data:{
                name:'spiderMan'
            }
        }).then(res => {
            console.log(res);
        })
    }
```



## 云存储

#### 1.上传图片

```javascript
//选择图片
 wx.chooseImage({
     count: 1,//最多上传图片的个数
     sizeType:['original','compressed'],//所选的图片尺寸。原图/缩略图
     sourceType:['album','camera'],//选择图片的来源，从相册选图/使用相机
 }).then(res => {
     this.setData({
         img:res.tempFilePaths[0]
     })

     
     //生成随机数为图片命名
     var num = Math.random()
     var time = Date.now()

     wx.cloud.uploadFile({
         cloudPath:'user/' + num + time + 'abc.png',//上传到云端
         //cloudPath:`user/${num}-${time}-abc.png`   es6
         filePath:res.tempFilePaths[0]//小程序临时文件路径
     }).then(result => {
         // console.log(result);
         this.setData({
             yunImgUrl:result.fileID
         })
         wx.showToast({
             title: '上传成功！'
         })
     })
```

#### 2.给用户更新头像，并阅览

```javascript
 <image src="{{user.Faceimage}}" class="face"></image>


updateFaceImg(){
        
    //选择图片
    wx.chooseImage({
        count: 1,//最多上传图片的个数
        sizeType:['original','compressed'],//所选的图片尺寸。原图/缩略图
        sourceType:['album','camera'],//选择图片的来源，从相册选图/使用相机
    }).then(res => {
        this.setData({
            img:res.tempFilePaths[0]
        })


        //生成随机数为图片命名
        var num = Math.random()
        var time = Date.now()

        wx.cloud.uploadFile({
            cloudPath:'user/' + num + time + 'abc.png',//上传到云端
            //cloudPath:`user/${num}-${time}-abc.png`   es6
            filePath:res.tempFilePaths[0]//小程序临时文件路径
        }).then(result => {
            // console.log(result);
            this.setData({
                yunImgUrl:result.fileID
            })
            //给数据库添加数据
            this.updateFace()

            wx.showToast({
                title: '上传成功！'
            })
        })
    })

},    
    //给数据库加一条图像数据
    updateFace(){
        wx.cloud.database().collection('yun_users').doc(this.data.user._id)
            .update({
            data:{
                Faceimage:this.data.yunImgUrl
            }
        }).then(res=>{
            console.log(res);
            this.getuserInfo()
            wx.showToast({
                title: '更新成功！',
            })
        })
    }
```

#### 3.





