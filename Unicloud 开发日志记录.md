Unicloud 开发日志记录

1. 创建数据库
    uniCloud.database()
    db.collection("user");
2.  新增记录let rest = await usercollection.add()		 
3. 删除
   collection.doc(id).remove()
4. 更新表
   collection.doc(id).update(更新内容) -- 只能更新已经存在的元素
   collection.doc(id).set(更新内容) -- 更新已经存在的元素,不存在則新增

​           5.查詢

​            collection.doc(id).get();

​          collection.where("查詢條件").get()