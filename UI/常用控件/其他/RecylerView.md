存在的问题：自定义分隔符里面有点小问题地址http://blog.csdn.net/lmj623565791/article/details/45059587

特点:
想要控制其显示的方式，请通过布局管理器LayoutManager
想要控制Item间的间隔（可绘制），请通过ItemDecoration
想要控制Item增删的动画，请通过ItemAnimator
想要控制点击、长按事件,可以自己实现。
recyclerview自带ViewHolder。
要理解recyclerview，它的意思是只管Recycler View，也就是说RecyclerView只管回收与复用View，其他的你可以自己去设置。可以看出其高度的解耦，给予你充分的定制自由（所以你才可以轻松的通过这个控件实现ListView,GirdView，瀑布流等效果）。

以下的代码设置了一个线性布局的Recycleview.代码中给出注释。
package com.heihei.hehe.recyclerview_demo.divider;

import android.graphics.Color;
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.ViewGroup;
import android.widget.TextView;

import com.heihei.hehe.recyclerview_demo.R;
import com.heihei.hehe.recyclerview_demo.divider.widgt.ItemDivider;


public class LinearManagerActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_recycler);

	//RecyclerView布局可以在v7包中找到
        RecyclerView re = (RecyclerView) findViewById(R.id.re);

	//设置线性管理器并设置其布局方向为垂直
        LinearLayoutManager lay= new  LinearLayoutManager(this);
        lay.setOrientation(LinearLayoutManager.VERTICAL);

	//设置为水平方向
	//lay.setOrientation(LinearLayoutManager.HORIZONTAL);

        re.setLayoutManager(lay);
	//设置动画
	re.setItemAnimator(new DefaultItemAnimator());

	//添加分割线 
        re.addItemDecoration(new DividerItemDecoration(this,DividerItemDecoration.VERTICAL));

	//item固定高度时，可以设置这个属性提高性能
        //re.setHasFixedSize(true);

	//设置适配器
        re.setAdapter(new MyAdapter());
    }

    private class MyAdapter extends RecyclerView.Adapter{

        @Override
        public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
            return new RecyclerView.ViewHolder(LayoutInflater.from(parent.getContext()).inflate(android.R.layout.simple_list_item_1,parent,false)) {
            };
        }

        @Override
        public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
            TextView tv = (TextView) holder.itemView.findViewById(android.R.id.text1);
            tv.setText(String.valueOf(position));
//            holder.itemView.setBackgroundColor(Color.BLUE);
        }

        @Override
        public int getItemCount() {
            return 100;
        }
    }
}
系统提供了以下几个布局管理器：
LinearLayoutManager 现行管理器，支持横向、纵向。
GridLayoutManager 网格布局管理器
StaggeredGridLayoutManager 瀑布就式布局管理器

//设置一个GridLayoutManager。
GridLayoutManager manager= new GridLayoutManager( this,4,GridLayoutManager.VERTICAL,false);

//瀑布流式布局
 mRecyclerView.setLayoutManager(new StaggeredGridLayoutManager(4,StaggeredGridLayoutManager.VERTICAL));
可以看到，固定为4行，变成了左右滑动。有一点需要注意，如果是横向的时候，item的宽度需要注意去设置，毕竟横向的宽度没有约束了，应为控件可以横向滚动了。


ItemAnimator

// 设置item动画 系统提供了一个默认的实现
mRecyclerView.setItemAnimator(new DefaultItemAnimator());

更新数据的时候使用notifyItemInserted(position)与notifyItemRemoved(position) 而不是adapter.notifyDataSetChanged()否则没有动画效果。

点击事件（需要自定义）

方式：通过adapter中自己去提供回调接口
public static class  RecycleAdapter extends RecyclerView.Adapter<RecycleAdapter.MViewHolder>{
        public  interface OnItemClickListener{
            void OnItemClick(View view,int position);
        }
        private OnItemClickListener listener;

        public void setOnItemClickLitener(OnItemClickListener listener){
            this.listener=listener;
        }
        private ArrayList<String> list=new ArrayList<>();
        public RecycleAdapter(ArrayList<String> list) {
            this.list=list;
        }



        @Override
        public RecycleAdapter.MViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
            View view= LayoutInflater.from(parent.getContext()).inflate(R.layout.item,parent,false);
            MViewHolder holder=new MViewHolder(view);
            return holder;
        }

        @Override
        public void onBindViewHolder(final RecycleAdapter.MViewHolder holder, int position) {
                holder.tv.setText(list.get(position));
            if(listener!=null){
                holder.itemView.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        int pos=holder.getLayoutPosition();
                       listener.OnItemClick(v,pos);
                    }
                });
            }

        }

        @Override
        public int getItemCount() {
            return list.size();
        }
         class MViewHolder extends RecyclerView.ViewHolder{
             TextView tv;
            public MViewHolder(View itemView) {
                super(itemView);
                tv=(TextView)itemView.findViewById(R.id.textview);
            }
        }

    }
Activity中代码：
mAdapter.setOnItemClickLitener(new OnItemClickLitener()
        {

            @Override
            public void onItemClick(View view, int position)
            {
                Toast.makeText(HomeActivity.this, position + " click",
                        Toast.LENGTH_SHORT).show();
            }

        });

//设置占据的空间
        manager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
            @Override
            public int getSpanSize(int position) {
                if(position % 5 == 0){
                    return 4;
                }else if(position % 5 == 1){
                    return 3;
                }else if(position % 5 == 2){
                    return 1;
                }else{
                    return 2;
                }

            }
        });

拖拽：实现ItemTouchHelper.Callback 具体实现见代码
