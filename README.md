# Andrioid_Mid_NodePad
nomal NodePad

1.主页面

![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/1.png)


通过showNotesList()来显示整个记事的列表，setOnItemLongClickListener设置长按弹出的功能，setOnItemClickListener设置单击弹出的功能
```
protected  void onStart( ){
      super.onStart();
        //显示记事列表
        showNotesList();
       //为记事列表添加长按事件
        lv_notes.setOnItemLongClickListener(new ItemLongClickEvent());
        lv_notes.setOnItemClickListener(new ItemClickEvent());
    }
```

创建或打开数据库，并将数据库每一条记录放入到adapter当中
```
private void showNotesList(){
        //创建或打开数据库
        dop.create_db();
        cursor = dop.query_db();
        adapter = new SimpleCursorAdapter(this,
                R.layout.note_item,
                cursor,
                new String[]{"_id","title","time","context"}, new int[]{R.id.tv_note_id,R.id.tv_note_title,R.id.tv_note_time,R.id.tv_note_content});
        lv_notes.setAdapter(adapter);
    }
```	


主页面的XML文件(LinearLayout下包含多个ListView）
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    >
    <ListView
        android:id="@+id/lv_notes"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        ></ListView>
</LinearLayout>
```

2.增加笔记

![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/5.png)

ClickEvent是对主页面事件的监听，当事件为bt_back时，就返回到主页面；当事件为bt_save时，就会进入判断是为update还是newAdd

```
    class ClickEvent implements OnClickListener{

        @Override
        public void onClick(View v) {
            switch(v.getId()){
                case R.id.bt_back :
                    AddActivity.this.finish();
                    break;

                case R.id.bt_save :
                    String context = et_Notes.getText().toString();
                    Log.e("调试1",context);
                    if(context.isEmpty()){
                        Toast.makeText(AddActivity.this, "记事为空!", Toast.LENGTH_LONG).show();
                    }
                    else{

                        SimpleDateFormat   formatter   =   new   SimpleDateFormat   ("yyyy-MM-dd HH:mm");
                        Date   curDate   =   new   Date(System.currentTimeMillis());
                        String   time   =   formatter.format(curDate);
                        String title = getTitle(context);

                        dop.create_db();
                        if(editModel.equals("newAdd")){
                            dop.insert_db(title,context,time);
                        }
                        else if(editModel.equals("update")){
                            dop.update_db(title,context,time,item_Id);
                        }
                        dop.close_db();
                        AddActivity.this.finish();
                    }
                    break;
            }
        }
    }
```
其中此处为通过Date来获取当前的时间，SimpleDateFormat将Date转换为yyyy-MM-dd HH:mm的格式
```
    SimpleDateFormat   formatter   =   new   SimpleDateFormat   ("yyyy-MM-dd HH:mm");
                        Date   curDate   =   new   Date(System.currentTimeMillis());
                        String   time   =   formatter.format(curDate);
                        String title = getTitle(context);
```
当事件为newAdd时，执行数据库中insert_db的方法，把数据插入到数据库当中
```
    public void insert_db(String title,String text,String time){


		if(text.isEmpty()){
			Toast.makeText(context, "各字段不能为空", Toast.LENGTH_LONG).show();
		}
		else{
			db.execSQL("insert into notes(title,context,time) values('"+ title+"','"+ text+ "','"+time+"');");
			Toast.makeText(context, "插入成功", Toast.LENGTH_LONG).show();
		}

	}
```
3.编辑笔记

![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/3.png)
![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/2.png)
事件判断是newAdd还是update
```
        //加载数据
    private void loadData(){

        //如果是新增记事模式，则将editText清空
        if(editModel.equals("newAdd")){
            et_Notes.setText("");
        }
        //如果编辑的是已存在的记事，则将数据库的保存的数据取出，并显示在EditText中
        else if(editModel.equals("update")){
            tv_title.setText("编辑记事");
            dop.create_db();
            Cursor cursor = dop.query_db(item_Id);
            cursor.moveToFirst();
            //取出数据库中相应的字段内容
            String context = cursor.getString(cursor.getColumnIndex("context"));
            et_Notes.setText(context);
            dop.close_db();
        }
    }
```
在主页面当中长按一个记事，就弹出编辑还是删除的选项
```
       class ItemLongClickEvent implements OnItemLongClickListener{

        @Override
        public boolean onItemLongClick(AdapterView<?> parent, View view,
                                       int position, long id) {
            tv_note_id = (TextView)view.findViewById(R.id.tv_note_id);
            int item_id = Integer.parseInt(tv_note_id.getText().toString());
            simpleList(item_id);
            return true;
        }
    }
```
当选择编辑时，就会调用到db里的update_db，当成功更新纪事以后，通过toast范围一个信息
```
public void update_db(String title,String text,String time,int item_ID){
		if( text.isEmpty()){
			Toast.makeText(context, "各字段不能为空", Toast.LENGTH_LONG).show();
		}
		else{
			//String sql = "update main set class1='" + class1 + "',class2='" + class2 + "',class3='" + class4 + "',class4='" + class4 + "'where days='" + days + "';";
			db.execSQL("update notes set context='"+text+ "',title='"+title+"',time='"+time+"'where _id='" + item_ID+"'");
			Toast.makeText(context, "修改成功", Toast.LENGTH_LONG).show();
		}
	}
```

4.删除笔记
![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/4.png)
在主页面当中长按一个记事，就弹出编辑还是删除的选项
```
  @Override
        public boolean onItemLongClick(AdapterView<?> parent, View view,
                                       int position, long id) {
            tv_note_id = (TextView)view.findViewById(R.id.tv_note_id);
            int item_id = Integer.parseInt(tv_note_id.getText().toString());
            simpleList(item_id);
            return true;
        }
```
选择删除选项时，会打开数据库，调用dop.delete_db(item_id)，传入id值，并关闭数据库
```
 public void simpleList(final int item_id){
        AlertDialog.Builder alertDialogBuilder = new AlertDialog.Builder(this,R.style.custom_dialog);
        alertDialogBuilder.setTitle("选择操作");
        alertDialogBuilder.setIcon(R.drawable.ic_launcher);
        alertDialogBuilder.setItems(R.array.itemOperation, new DialogInterface.OnClickListener() {

            @Override
            public void onClick(DialogInterface dialog, int which) {

                switch(which){
                    //编辑
                    case 0 :
                        Intent intent = new Intent(MainActivity.this,AddActivity.class);
                        intent.putExtra("editModel", "update");
                        intent.putExtra("noteId", item_id);
                        startActivity(intent);
                        break;
                    //删除
                    case 1 :
                        dop.create_db();
                        dop.delete_db(item_id);
                        dop.close_db();
                        //刷新列表显示
                        lv_notes.invalidate();
                        showNotesList();
                        break;
                }
            }
        });
        alertDialogBuilder.create();
        alertDialogBuilder.show();
    }
```
通过ID来删除数据库内的数据，Toast返回结果
```
public void delete_db(int item_ID){
		db.execSQL("delete from notes where _id='" + item_ID+"'");
		Toast.makeText(context, "删除成功", Toast.LENGTH_LONG).show();
	}
```
5.搜索笔记
![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/9.png)

主页面搜索菜单的事件，通过动态搜索调用Search_query_db来返回数据库内相关title的记事，通过swapCursor来找到这些数据
```
   mSearchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String query) {
                Toast.makeText(MainActivity.this, "Submit---提交", Toast.LENGTH_SHORT).show();
                return true;
            }

            @Override
            public boolean onQueryTextChange(String newText) {
                Toast.makeText(MainActivity.this, "改变", Toast.LENGTH_SHORT).show();
                cursor = dop.Search_query_db(newText);
                adapter.swapCursor(cursor);
                return true;
            }
        });
```
返回一个cursor类型的数据
```
public Cursor  Search_query_db(String search_content){
		Cursor cursor = db.rawQuery("select * from notes where  context LIKE '%" + search_content + "%'" ,null);
		return cursor;
	}
```
关闭搜索框
```
mSearchView.setOnCloseListener(new SearchView.OnCloseListener() {
            @Override
            public boolean onClose() {
                Toast.makeText(MainActivity.this, "关闭搜索框", Toast.LENGTH_SHORT).show();
                return false;
            }
        });
```


6.添加图片

![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/6.png)
在新增或者编辑记事的页面，选择图片功能时候，会打开系统的相册页面
```
case 3:
                    //添加图片的主要代码
                    intent = new Intent();
                    //设定类型为image
                    intent.setType("image/*");
                    //设置action
                    intent.setAction(Intent.ACTION_GET_CONTENT);
                    //选中相片后返回本Activity
                    startActivityForResult(intent, 1);
                    break;
```
当选择类型为照片时，将通过bitmap的形式来获取数据，然后通过InsertBitmap插入数据
```
   
                String[] proj = { MediaStore.Images.Media.DATA };
                Cursor actualimagecursor = managedQuery(uri,proj,null,null,null);
                int actual_image_column_index = actualimagecursor.getColumnIndexOrThrow(MediaStore.Images.Media.DATA);
                actualimagecursor.moveToFirst();
                String path = actualimagecursor.getString(actual_image_column_index);
                try {

                    
                    bitmap = BitmapFactory.decodeStream(cr.openInputStream(uri));
                } catch (FileNotFoundException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
               
                InsertBitmap(bitmap,480,path);
```
这是将图片等比例缩放到合适的大小并添加在EditText中的代码
```
 void InsertBitmap(Bitmap bitmap,int S,String imgPath){


        final ImageSpan imageSpan = new ImageSpan(this,bitmap);
        SpannableString spannableString = new SpannableString(imgPath);
        spannableString.setSpan(imageSpan, 0, spannableString.length(), SpannableString.SPAN_MARK_MARK);
        Editable editable = et_Notes.getEditableText();
        int selectionIndex = et_Notes.getSelectionStart();
        spannableString.getSpans(0, spannableString.length(), ImageSpan.class);
        editable.insert(selectionIndex, spannableString);
        et_Notes.append("\n");
        Map<String,String> map = new HashMap<String,String>();
        map.put("location", selectionIndex+"-"+(selectionIndex+spannableString.length()));
        map.put("path", imgPath);
        imgList.add(map);

    }
```

7.添加语音

![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/7.png)
![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/8.png)
当插入的类型为语音形式的时候，用bitmap的形式来保存语音信息，并调用InsertBitmap
```
   extras = data.getExtras();
                String audioPath = extras.getString("audio");
                bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.record_microphone_icon);
                InsertBitmap(bitmap,80);
```
将插入的语音规格进行调整，放入到edittext当中
```
     void InsertBitmap(Bitmap bitmap,int S){

        int imgWidth = bitmap.getWidth();
        int imgHeight = bitmap.getHeight();
        double partion = imgWidth*1.0/imgHeight;
        double sqrtLength = Math.sqrt(partion*partion + 1);
        //新的缩略图大小
        double newImgW = S*(partion / sqrtLength);
        double newImgH = S*(1 / sqrtLength);
        float scaleW = (float) (newImgW/imgWidth);
        float scaleH = (float) (newImgH/imgHeight);

        Matrix mx = new Matrix();
        //对原图片进行缩放
        mx.postScale(scaleW, scaleH);
        bitmap = Bitmap.createBitmap(bitmap, 0, 0, imgWidth, imgHeight, mx, true);
        final ImageSpan imageSpan = new ImageSpan(this,bitmap);
        SpannableString spannableString = new SpannableString("test");
        spannableString.setSpan(imageSpan, 0, spannableString.length(), SpannableString.SPAN_MARK_MARK);
        //光标移到下一行
        //et_Notes.append("\n");
        Editable editable = et_Notes.getEditableText();
        int selectionIndex = et_Notes.getSelectionStart();
        spannableString.getSpans(0, spannableString.length(), ImageSpan.class);

        //将图片添加进EditText中
        editable.insert(selectionIndex, spannableString);
        //添加图片后自动空出两行
        et_Notes.append("\n");
    }
```
在已经授予权限的情况下，本虚拟机还是无法打开语音系统。
