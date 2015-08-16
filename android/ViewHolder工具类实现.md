&emsp;&emsp;在开发APP的过程中，攻城狮少不了要跟ListView、GridView这些组件眉来眼去，暗送几波秋波。自然原生态美人BaseAdapter更是程序员的最爱，有了它，我们想怎么干就能怎么干，嘿嘿，你懂的哈哈~
##前言
&emsp;&emsp;在开发APP的过程中，攻城狮少不了要跟ListView、GridView这些组件眉来眼去，暗送几波秋波。自然原生态美人BaseAdapter更是程序员的最爱，有了它，我们想怎么干就能怎么干，嘿嘿，你懂的哈哈~

&emsp;&emsp;但是，每次写一个BaseAdapter，我们都很自觉的给他写一个ViewHolder，一两个还好，万一应用程序中有数不清的ListView，呵呵~你妹！千篇一律，看得都审美疲劳。作为最伟大的第二十二世纪的程序员们，脱掉、搞上永远是我们最真挚的追求，所以我们要怎么将ViewHolder从BaseAdapter中脱掉呢？绝非不是不用，而是要将其搞成一个华丽丽的工具类实现，收入角落那个寂寞得tools类中。
##ViewHolder的实现

&emsp;&emsp;我觉得应该简略的介绍下ViewHolder的实现，谷歌很聪明的在Adapter中运用了复用View的思想，自然让我们的屌丝机也能泡上一些白富美应用多了一点点可能。ViewHolder的具体实现基本体现在BaseAdapter的 getView(int position, View convertView, ViewGroup parent) 这个方法里面，参见下面的代码：

```java
@Override 
public View getView(int position, View convertView, ViewGroup parent)
{    
          ViewHolder holder;    
          if (convertView == null) 
          {        
            convertView = inflater.inflate(R.layout.listview_item_layout, parent, false); 
            holder = new ViewHolder();        
            holder.studentName = (TextView) convertView.findViewById(R.id.student_name);
            holder.studentAge = (TextView) convertView.findViewById(R.id.student_age);
            convertView.setTag(holder);     
          }     
          else
          {  
            holder = (ViewHolder) convertView.getTag();     
          }     
            Student data = (Student) getItem(position);     
            holder.studentName.setText(data.getName());     
            holder.studentAge.setText(data.getAge());     
            return convertView;
} 
          
  class ViewHolder
  {     
      public TextView studentName;     
      public TextView studentAge; 
  } 
```
&emsp;&emsp;很明显，大家不要问我ViewHolder在哪里，稍微把目光往上扶一扶就看到那个大大的 class ViewHolder 。这里的ViewHolder用法主要有两个地方，一是 convertView 的复用，二是 ViewHolder 也就是 convertView 里面的索引的复用。具体的用法不熟悉的话可以百度一下，再往下说就对不起我今天这篇博文了，因为在这里写这个代码的目的，肯定不是介绍你怎么用ViewHolder，只是想告诉你：传统的ViewHolder的写法，是多么的臃肿！而且对于每一个新的BaseAdapter，你都得无聊的实现一次又一次，OH~
##ViewHolder的工具类实现
&emsp;&emsp;自然，脱光要从小，行动要趁早。既然我们烦了，就把它写成一个工具类咯。参见下面的代码
```java
static class ViewHolder {     
    public static <T extends View> T get(View view, int id) {         
          SparseArray<View> viewHolder = (SparseArray<View>) view.getTag();         
          if (viewHolder == null) {             
          viewHolder = new SparseArray<View>();             
          view.setTag(viewHolder);        
          }         
          View childView = viewHolder.get(id);         
          if (childView == null) {            
              childView = view.findViewById(id);             
              viewHolder.put(id, childView);         
          }         
      return (T) childView;     
      }
}
```
这是工具类的实现，稍微说下实现的原理：

1、ViewHolder既然是依赖View的Tag存放，但是以一个 SparseArray 集合存放。

2、判断View里的Tag是否存在viewHolder，不存在，赶紧叫她生一个。

3、然后在viewholder（也就是SparseArray）寻找View的索引，如果没有，赶紧findViewById一个put进去顺便return出来，如果已经存在，皆大欢喜，直接用呗。

贴个BaseAdapter里面使用的代码：

```java
@Override 
public View getView(int position, View convertView, ViewGroup parent)
{     
        if (convertView == null)
        {        
          convertView = inflater.inflate(R.layout.listview_item_layout, parent, false);    
        }     
        TextView name = Tools.ViewHolder.get(convertView, R.id.student_name);   
        TextView age = Tools.ViewHolder.get(convertView, R.id.student_age);     
        Student data = (Student) getItem(position);     
        name.setText(data.getName());     
        age.setText(data.getAge());          
        return convertView;
} 
```
&emsp;&emsp;简洁明了，不用多说~~~嘿嘿，后面如果要写ViewHolder，直接Tools工具类调用，省心不废脑。。
##分析可行性
既然要作为工具类使用，我们有必要先评估这个工具值不值得我们使用。

一般来说，我们可以从以下几个方面进行评估：易用性? 内存泄露？ 性能提升？ 健壮性？等等等。。。。。。

易用性：工具类的最大特性就是易用简约，这个ViewHolder的写法就是典型的拿来就用的主义，根本不用我们操心写些适配的代码，直接传入View和id，高内聚松耦合。并且采用了<T extends View> T的泛型模板的方法，自动与外部的View子类适配，不用我们手动去强制装换。

内存泄露：有些初学者，看到static方法就回固执的认为 SparseArray<View> viewHolder 这个变量会存在内存泄露，但是java告诉我们，这个变量的小命仅仅在方法执行之中，方法完毕，GC回收；存在ViewHolder一如既往放在View的Tag中，一旦View被回收，ViewHolder自然消失。不信，打开DDMS，用你28青年的手速不停刷listView试试，保证对象基本稳定在一个值。

性能提升：在这里我们发现用了 SparseArray 这个集合而不是 HashMap ，我们知道 SparseArray 是Android的一个工具类，是官方推荐用来代替 HashMap<Integer,E> 的一个类，它的内部采用了二分查找的实现提高了查找效率，而且不是一点两点的哦，谁用谁知道；具体内容想要了解，可以度娘谷哥或者左转源码。

所以，作为一个工具类，它是完全合格的，赶紧把它拷贝到你的tools、util里面，然后我们就可以开心加愉悦的优雅用起ViewHolder了。。

资料来源http://mobile.51cto.com/aprogram-475335.htm
