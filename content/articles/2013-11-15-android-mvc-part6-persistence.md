---
title: Android MVC Part6 Persistence
date: 2013-11-15
category: android
tags: mvc
Parts: mvc
---

最难的 MVC 已经讲解过了。如果你可以在 android 上实现 MVC, 这系列剩下的部分将会很容易。奖赏一下自己吧，接下来我们要讨论一下数据持久化。
<!-- excerpt -->

我会默认你已经知道 sqlite 和其他 android 中持久化的方法。如果不知道，现在是时候去补补了。我会建议你了解 Data Access Objects 或者叫 DAOs 的概念。这个概念非常简单：我们传递一个模型的实例给 DAOs ，由它来保存，更新或者删除数据。再也不用编写一大堆 sql 语句了。妈妈再也不用担心我调用数据库了。废话不说，看看代码：

    :::java
    public class CounterDao {
     
        protected static final String TABLE = "Counter";
        protected static final String _ID = "_id";
        protected static final String LABEL = "label";
        protected static final String COUNT = "count";
        protected static final String LOCKED = "locked";
     
        public CounterDao() {
            //
        }
     
        public ArrayList<CounterVo> getAll() {
            // ... truncated.... returns all CounterVos
        }
     
        public CounterVo get(int id) {
            SQLiteDatabase db = new DatabaseHelper().getWritableDatabase();
            Cursor cursor = db.query(TABLE, null, _ID+"=?", new String[] {Integer.toString(id)}, null, null, null);
            CounterVo vo = null;
            if (cursor.moveToFirst()) {
                vo = new CounterVo();
                vo.setId(cursor.getInt(cursor.getColumnIndex(_ID)));
                vo.setLabel(cursor.getString(cursor.getColumnIndex(LABEL)));
                vo.setCount(cursor.getInt(cursor.getColumnIndex(COUNT)));
                vo.setLocked(cursor.getInt(cursor.getColumnIndex(LOCKED)) == 1);
            }
     
            cursor.close();
            db.close();
            return vo;
        }
     
        public long insert(CounterVo counterVo) {
            SQLiteDatabase db = new DatabaseHelper().getWritableDatabase();
            ContentValues values = new ContentValues();
            if (counterVo.getId() > 0) values.put(_ID, counterVo.getId());
            values.put(LABEL, counterVo.getLabel());
            values.put(COUNT, counterVo.getCount());
            values.put(LOCKED, counterVo.isLocked());
     
            long num = db.insert(TABLE, null, values);
            db.close();
            return num;
        }
     
        public int update(CounterVo counterVo) {
            // ... truncated....
        }
     
        public void delete(int id) {
            SQLiteDatabase db = new DatabaseHelper().getWritableDatabase();
            db.delete(TABLE, _ID+"=?", new String[]{Integer.toString(id)});
            db.close();
        }
    }

`CounterDao` 就是用来持久化 `CounterVo` 的。里面的持久化实现可以使用 sqlite, json, xml 或者 shared preferences ，也就是外部不用知道内部是如何实现持久化的，调用接口就可以了。

> 假如我想使用更多的方式获取数据，我不就需要在视图中编写特定的 sql 语句吗？

首先，你的视图不应该有逻辑。第二，如果有需要的话，在 DAO 中增加方法就可以了。DAO 中最重要的概念是：你只要发送需要持久化的模型或数据给我就好了，至于我怎么实现，你不用管。

来看看我们是如何使用 DAOs 的：

    :::java
    private void populateModel(final int id) {
        if (id < 0) return;
        workerHandler.post(new Runnable() {
            @Override
            public void run() {
                synchronized (model) {
                    CounterDao dao = new CounterDao();
                    CounterVo vo = dao.get(id);
                    if (vo == null) vo = new CounterVo();
                    model.consume(vo);
                }
            }
        });
    }

当视图请求填入模型的时候这个方法会被调用。控制器把任务放在线程上执行，然后调用 DAO 来获得模型对象。

>为什么不把保存操作放到 Vo 里面，那样不是更合理吗？

无错，我看过其他开发者这样做（只是我要写这个系列的原因）。但这不是一个好的做法。记得我们之前讲的单一职责原则吗？ 数据对象应该只持有状态。它不应该做其他更多的事，因为这违反了单一职责原则。

最后的提示，我用常量来指定了 DAO 数据库中的表的名字。你应该避免直接在 sql 中写 strings ，因为你很有可能会拼错。接着 我使用了 protected 来修饰这些常量，表示这些常量包内可见，因为 DatabaseHelper 需要用到。

##目前为止

理清一下我们程序现在做的事情：

1. 视图询问控制器要新数据。
2. 控制器把获取数据的任务代理给 DAO。
3. 控制器把数据添加到模型中。
4. 模型发消息说数据有更新。
5. 视图更新界面。

把各个部分解耦得很好，不是吗？
