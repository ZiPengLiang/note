Http常见状态码



200 OK -[GET]:服务器成功返回用户请求的数据

201 GREATED - [POST/PUT/PATCH]:用户新建或修改数据成功

204 NO CONTENT - [DELETE]：用户删除数据成功



401 Unauthorized -[*]:表示用户没有权限 （令牌、用户、密码错误）

403 Forbidden -[*]:表示用户获得授权(与401错误相对)，但是访问是被禁止的

404 NOT FOUND -[*]:用户发出的请求针对的是不存在的记录，服务器没有进行操作

