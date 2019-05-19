# Andrioid_Mid_NodePad
nomal NodePad

1.主页面

![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/1.png)

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


3.编辑笔记

![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/3.png)
![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/2.png)

```

    
        if(editModel.equals("newAdd")){
            et_Notes.setText("");
        }
        else if(editModel.equals("update")){
            tv_title.setText("编辑记事");
            dop.create_db();
            Cursor cursor = dop.query_db(item_Id);
            cursor.moveToFirst();
            String context = cursor.getString(cursor.getColumnIndex("context"));
            et_Notes.setText(context);
            dop.close_db();
        }
```


4.删除笔记
![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/4.png)

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

5.搜索笔记
![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/9.png)

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

6.添加图片

![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/6.png)

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


7.添加语音

![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/7.png)
![](https://raw.githubusercontent.com/BarongDog/Andrioid_Mid_NodePad/master/pic/8.png)

```
   extras = data.getExtras();
                String audioPath = extras.getString("audio");
                bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.record_microphone_icon);
                InsertBitmap(bitmap,80);
```

```
    
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
```
