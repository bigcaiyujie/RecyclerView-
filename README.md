####先说使用RecyclerView要实现的必要步骤：
1.用**LayoutManager**管理它的布局：横向或竖向，不同的布局风格；
2.实现Adapter类继承**RecyclerView.Adapter**类；
3.**RecyclerView.addItemDecoration**(newDividerItemDecoration()):
制作item之间的分割线；
4.**RecyclerView.setItemAnimator**（new MyItemAnimator()）:制作删除添加item等操作的动画效果；
5.实现RecyclerView的**事件监听器**：点击，长按等
><code>mRecyclerView = findView(R.id.id_recyclerview);
//设置布局管理器
mRecyclerView.setLayoutManager(layout);
//设置adapter
mRecyclerView.setAdapter(adapter)
//设置Item增加、移除动画
mRecyclerView.setItemAnimator(new DefaultItemAnimator());
//添加分割线
mRecyclerView.addItemDecoration(new DividerItemDecoration(
                getActivity(), DividerItemDecoration.HORIZONTAL_LIST));
</code>

####实现步骤
使用RecyclerView之前我们要导入相对应的包：  
><code>compile 'com.android.support:recyclerview-v7:+'</code>

布局文件：主布局
><pre><code>
 <android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>  
</code></pre>

item子布局：
<pre><code>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:background="@drawable/item_bg"
    android:layout_margin="3dp"
    android:layout_height="72dp">    
<TextView
        android:layout_width="72dp"
        android:layout_height="match_parent"
        android:id="@+id/Item_text"
        android:gravity="center" />
</LinearLayout>
</code></pre>
   
##一.设置它的布局管理：
>     LinearLayoutManager linearLayoutManager  = new LinearLayoutManager(this,LinearLayoutManager.VERTICAL,false);        
>      recyclerView.setLayoutManager(linearLayoutManager);

####三个布局：直接上源代码
1 .**线性布局**：
      
      /**  * @param context       Current context, will be used to access resources.
     * @param orientation   Layout orientation. Should be {@link #HORIZONTAL} or{@link#VERTICAL}.
     * @param reverseLayout When set to true, layouts from end to start.
     */
    public LinearLayoutManager(Context context, int orientation, boolean reverseLayout) {
        mAnchorInfo = new AnchorInfo();
        setOrientation(orientation);
        setReverseLayout(reverseLayout);
    }

2.**格子布局**：
  
      /** * @param context Current context, will be used to access resources.
     * @param spanCount The number of columns or rows in the grid
     * @param orientation Layout orientation. Should be {@link #HORIZONTAL} or {@link
     *                      #VERTICAL}.
     * @param reverseLayout When set to true, layouts from end to start.
     */
    public GridLayoutManager(Context context, int spanCount, int orientation,
            boolean reverseLayout) {
        super(context, orientation, reverseLayout);
        setSpanCount(spanCount);
    }
3.**瀑布流布局**：
  
     /**  * Creates a StaggeredGridLayoutManager with given parameters.
     *
     * @param spanCount   If orientation is vertical, spanCount is number of columns. If
     *                    orientation is horizontal, spanCount is number of rows.
     * @param orientation {@link #VERTICAL} or {@link #HORIZONTAL}
     */
    public StaggeredGridLayoutManager(int spanCount, int orientation) {
        mOrientation = orientation;
        setSpanCount(spanCount);
    }

##二.创建适配器  
<code>

    public class MyRecyclerViewAdatper extends RecyclerView.Adapter<RecyclerView.ViewHolder>  {
      private LayoutInflater inflater;
      private Context context;
      private List<String> datas;
    private OnitemClickListener onitemClickListener;

    public  interface  OnitemClickListener{
        void onItemClick(View view,int positon);
        void onItemLongClick(View view,int position);
    }

    public void setOntermClickListener(OnitemClickListener ontermClickListener){
        this.onitemClickListener = ontermClickListener;
    }
       public MyAdapter(Context context,List<String> datas){
        this.context = context;
        this.datas = datas;
        inflater = LayoutInflater.from(context);

    }
    @Override
    public long getItemId(int position) {//返回数据的id
        return super.getItemId(position);
    }

    @Override
    public int getItemCount() { //返回数据的数量
        return datas.size();
    }

    @Override
    public int getItemViewType(int position) { //放回item的类型（制定不同风格的item要用到）
        return position;
    }

    @Override
    public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
     //这个函数是初始化视图，返回一个ViewHolder，方法是把View直接封装在ViewHolder中，然后我们面向的是ViewHolder这个实例
        View view  = inflater.inflate(R.layout.item,parent,false);
        MyViewHolder myViewHolder = new MyViewHolder(view);
        return myViewHolder;
    }

    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
          //对视图的操作主要放在这里
          holder.textView.setText(datas.get(position));
       if(onitemClickListener!=null){
            holder.itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    onitemClickListener.onItemClick(holder.itemView, position);
                }
            });
            holder.itemView.setOnLongClickListener(new View.OnLongClickListener() {
                @Override
                public boolean onLongClick(View v) {
                    onitemClickListener.onItemLongClick(holder.itemView, position);
                    return false;
                }
            });
        }
    }
     public void addData(int pos){
        datas.add(pos,"Insert one");
        notifyItemInserted(pos);
    }

    public void deletData(int pos){
        datas.remove(pos);
        notifyItemRemoved(pos);
    }
      //就是一个持有者的类，他里面一般没有方法，只有属性，作用就是一个临时的储存器，
      //把你getView方法中每次返回的View存起来，可以下次再用。这样做的好处就是不必每次都到布局文件中去拿到你的View，提高了效率。
    class MyViewHolder extends RecyclerView.ViewHolder{
    TextView textView;
    public MyViewHolder(View itemView) {
        super(itemView);
        textView = (TextView)itemView.findViewById(R.id.Item_text);
    }
    }</code>

####说明： 
> notifyItemInserted(pos);//增加item  
> notifyItemRemoved(pos);//删除item
