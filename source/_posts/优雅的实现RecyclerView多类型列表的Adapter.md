---
title: 优雅的实现RecyclerView多类型列表的Adapter
date: 2017-4-10 12:28:37
categories:
 - Android
tags: 
 - RecyclerView
---

### 引言

在开发中经常会遇到，一个列表(RecyclerView)中有多种布局类型的情况。前段时间，看到了这篇文章

> [关于 Android Adapter，你的实现方式可能一直都有问题](http://www.jianshu.com/p/c6a44e18badb)

文中主要从设计的角度阐释如何更合理的实现多种布局类型的Adapter，本文主要从实践的角度出发，站在巨人的肩膀上，结合我个人的理解进行阐述，如果有纰漏，欢迎留言指出。

<!-- more -->

### 有多种布局类型

有时候，由于应用场景的需要，列表(RecyclerView)中需要存在一种以上的布局类型。为了阐述的方便，我们先假设一种应用场景

`列表中含有若干个常规的布局，在列表的中的第一个位置与第二个位置中分别为两个不同的布局，其余为常规的布局`

针对这样的需求，笔者一直以来的实现方式如下

```java
private final int ITEM_TYPE_ONE = 1;
private final int ITEM_TYPE_TWO = 2;

@Override
public int getItemViewType(int position) {
    if(0 == position){
       return  ITEM_TYPE_ONE;
    }else if(1 == position){
        return ITEM_TYPE_TWO;
    }
    return super.getItemViewType(position);
}
@Override
public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
    if(ITEM_TYPE_ONE == viewType){
        return new OneViewHolder();
    }else if(ITEM_TYPE_TWO == viewType){
        return new TwoViewHolder();
    }
    return new NormalViewHolder()；            
}
@Override
//伪代码
public void onBindViewHolder(ViewHolder holder, int position) {
   if(holder instanceof OneViewHolder ){
         ...
    }else if(holder instanceof TwoViewHolder){
         ...
    }else{
        ...
    }
}
```
- 在Adapter的getItemViewType方法中返回特定位置的特定标识（根据前文需求，就是position0与position1）
- 在onCreateViewHolder中根据viewType参数，也就是getItemViewType的返回值来判断需要创建的ViewHolder类型
- 在onBindViewHolder方法中对ViewHolder的具体类型进行判断，分别为不同类型的ViewHolder进行绑定数据与逻辑处理

通过以上就能实现多类型列表的Adapter，但这样的代码写多了总会觉得别扭。

结合文章与我个人的理解，这种实现方式所存在弊端可以总结为以下几点：

- *类型检查与类型转型*，由于在onCreateViewHolder根据不同类型创建了不同的ViewHolder，所以在onBindViewHolder需要针对不同类型的ViewHolder进行数据绑定与逻辑处理，这导致需要通过instanceof对ViewHolder进行类型检查与类型转型。
[[译]关于 Android Adapter，你的实现方式可能一直都有问题中是这样说的](http://www.jianshu.com/p/c6a44e18badb)

> 许多年前，我在我的显示器上贴了许多的名言。其中的一个来自 Scott Meyers 写的《Effective C++》 这本书（最好的IT书籍之一），它是这么说的：
不管什么时候，只要你发现自己写的代码类似于 “ if the object is of type T1, then do something, but if it’s of type T2, then do something else ”，就给自己一耳光。

- 不利于扩展，目前的需求是列表中存在三种布局类类型，那么如果需求变动，极端一点的情况就是数据源是从服务器获取的，数据中的model决定列表中的布局类型。这种情况下，每当model改变或model类型增加，我们都要去改变adapter中很多的代码，同时Adapter还必须知道特定的model在列表中的位置（position）除非跟服务端约定好，model（位置）不变，很显然，这是不现实的。
[译]关于 Android Adapter，你的实现方式可能一直都有问题中是这样说的

> 另外，我们实行那些 adapter 的方法违背了 SOLID 原则中的“开闭准则” 。它是这样说的：“对扩展开放，对修改封闭。” 当我们添加另一个类型或者 model 到我们的类中时，比如叫 Rabbit 和 RabbitViewHolder，我们不得不在 Adapter 里改变许多的方法。 这是对开闭原则明显的违背。添加新对象不应该修改已存在的方法。

- 不利于维护，这点应该是上一点的延伸，随着列表中布局类型的增加与变更，getItemViewType、onCreateViewHolder、onBindViewHolder中的代码都需要变更或增加，Adapter 中的代码会变得臃肿与混乱，增加了代码的维护成本。

首先让我摸摸自己的脸，然后结合[[译]关于 Android Adapter，你的实现方式可能一直都有问题](http://www.jianshu.com/p/c6a44e18badb)，看看如何优雅的实现多类型列表的Adapter

### 优雅的实现

结合上文，我们的核心目的就是三个

- 避免类的类型检查与类型转型
- 增强Adapter的扩展性
- 增强Adapter的可维护性

前文提到了，当列表中类型增加或减少时Adapter中主要改动的就是getItemViewType、onCreateViewHolder、onBindViewHolder这三个方法，因此，我们就从这三个方法中开始着手。

Talk is cheap. Show me the code，围绕以上几点，开始码代码

getItemViewType

原本的代码是这样

```java
@Override
public int getItemViewType(int position) {
    if(0 == position){
       return  ITEM_TYPE_ONE;
    }else if(1 == position){
        return ITEM_TYPE_TWO;
    }
    return super.getItemViewType(position);
}
```
在这段代码中，我们必须知道特定的布局类型在列表中的位置，而布局类型在列表中的位置是由数据源决定的，为了解决这个问题并且减少if之类的逻辑判断简化代码，我们可以简单粗暴的在Model中增加type标识,优化之后getItemViewType的实现大致如下

```java
@Override
 public int getItemViewType(int position) {
    return modelList.get(position).getType();
 }
 ```
 
这样的方式有很大的局限性（谁用谁知道），这里就不展开了，直接看正确的姿势，先看代码

```java
public interface Visitable {
    int type(TypeFactory typeFactory);
}

public class One implements Visitable {
    ...
    ...
    @Override
    public int type(TypeFactory typeFactory) {
        return typeFactory.type(this);
    }
}

public class Two implements Visitable {
    ...
    ...
    @Override
    public int type(TypeFactory typeFactory) {
        return typeFactory.type(this);
    }
}

public class Normal implements Visitable{
    ...
    ...
    @Override
    public int type(TypeFactory typeFactory) {
        return typeFactory.type(this);
    }
}

public interface TypeFactory {
    int type(One one);

    int type(Two two);
}

public class TypeFactoryForList implements TypeFactory {
    private final int TYPE_RESOURCE_ONE = R.layout.layout_item_one;
    private final int TYPE_RESOURCE_TWO = R.layout.layout_item_two;
    private final int TYPE_RESOURCE_NORMAL = R.layout.layout_item_normal;
    @Override
    public int type(One one) {
        return TYPE_RESOURCE_ONE;
    }

    @Override
    public int type(Two one) {
        return TYPE_RESOURCE_TWO;
    }

    @Override
    public int type(Normal normal) {
        return TYPE_RESOURCE_NORMAL;
    }
    ...
}
```

针对getItemViewType可以进行如下实现

```java
private List<Visitable> modelList;
@Override
public int getItemViewType(int position) {
    return modelList.get(position).type(typeFactory);
}
 ```
 
### 小结：

- 通过接口抽象，将所有与列表相关的Model抽象为Visitable，当我们在初始化数据源时就能以List<Visitable>的形式将不同类型的Model集合在列表中；

- 通过访问者模式，将列表类型判断的相关代码抽取到TypeFactoryForList 中，同时所有列表类型对应的布局资源都在这个类中进行管理与维护，以这样的方式巧妙的增强了扩展性与可维护性；

- getItemViewType中不再需要进行if判断，通过数据源控制列表的布局类型，同时返回的不再是简单的布局类型标识，而是布局的资源ID（通过modelList.get(position).type()获取），进一步简化代码（在onCreateViewHolder中会体现出来）；

#### onCreateViewHolder

结合上文可以了解到，getItemViewType返回的是布局资源ID，也就是onCreateViewHolder(ViewGroup parent, int viewType)参数中的viewType，我们可以直接用viewType创建itemView，但是，问题来了，itemView创建之后，还是需要进行类型判断，创建不同的ViewHolder，针对这个问题可以分以下几个步骤解决
首先为了增强ViewHolder的灵活性，可以继承RecyclerView.ViewHolder派生出BaseViewHolder抽象类如下

```java
public abstract class BaseViewHolder<T> extends RecyclerView.ViewHolder {
    private SparseArray<View> views;
    private View mItemView;
    public BaseViewHolder(View itemView) {
        super(itemView);
        views = new SparseArray<>();
        this.mItemView = itemView;
    }

    public View getView(int resID) {
        View view = views.get(resID);

        if (view == null) {
            view = mItemView.findViewById(resID);
            views.put(resID,view);
        }

        return view;
    }

    public abstract void setUpView(T model, int position, MultiTypeAdapter adapter);
}
```

不同的ViewHolder继承BaseViewHolder并实现setUpView方法即可。

然后对TypeFactory 与TypeFactoryForList 增加如下代码

```java
public interface TypeFactory {
  ...
  BaseViewHolder createViewHolder(int type, View itemView);
}

public class TypeFactoryForList implements TypeFactory {
  private final int TYPE_RESOURCE_ONE = R.layout.layout_item_one;
  private final int TYPE_RESOURCE_TWO = R.layout.layout_item_two;
  private final int TYPE_RESOURCE_NORMAL = R.layout.layout_item_normal;
  ...
  @Override
  public BaseViewHolder createViewHolder(int type, View itemView) {

        if(TYPE_RESOURCE_ONE == type){
            return new OneViewHolder(itemView);
        }else if (TYPE_RESOURCE_TWO == type){
            return new TwoViewHolder(itemView);
        }else if (TYPE_RESOURCE_NORMAL == type){
            return new NormalViewHolder(itemView);
        }

        return null;
    }
}
```

最后对onCreateViewHolder方法进行如下实现

```java
@Override
public BaseViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
    Context context = parent.getContext();

    View itemView = View.inflate(context,viewType,null);
    return typeFactory.createViewHolder(viewType,itemView);
}
```

### 小结：

- 在onCreateViewHolder中以BaseViewHolder作为返回值类型。因为BaseViewHolder作为不同类型的ViewHolder的基类，可以避免在onBindViewHolder中对ViewHolder进行类型检查与类型转换，同时也可以简化onBindViewHolder方法中的代码（具体会在下文阐述）；

- 创建不同类型的ViewHolder的相关代码被抽取到了TypeFactoryForList 中，简化了onCreateViewHolder中的代码，同时与类型相关的代码都集中在TypeFactoryForList 中，方便后期维护与拓展；

#### onBindViewHolder

经过以上实现，onBindViewHolder中的代码就非常的轻盈了，如下

```java
@Override
public void onBindViewHolder(BaseViewHolder holder, int position) {
    holder.setUpView(models.get(position),position,this);
}
```

可以看到，在onBindViewHolder中不需要对ViewHolder进行类型检查与转换，也不需要针对不同类型的ViewHoler执行不同绑定操作，不同的列表布局类型的数据绑定（逻辑代码）都交给了与其自身对应的ViewHolder处理，如下(setUpView中的代码可根据实际情况修改)

```java
public class NormalViewHolder extends BaseViewHolder<Normal> {
    public NormalViewHolder(View itemView) {
        super(itemView);
    }

    @Override
    public void setUpView(final Normal model, int position, MultiTypeAdapter adapter) {
        final TextView textView = (TextView) getView(R.id.normal_title);
        textView.setText(model.getText());

        textView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Toast.makeText(textView.getContext(),model.getText(),Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

### 小结

onBindViewHolder中不需要进行类型检查与转换，对ItemView的数据绑定与逻辑处理都交由各自的ViewHolder进行处理。通过这样方式，让代码更整洁，更易于维护，同时也增强了扩展性。
总结

经过如上优化之后，Adapter中的代码如下

```java
public class MultiTypeAdapter extends RecyclerView.Adapter<BaseViewHolder> {
    private TypeFactory typeFactory;
    private List<Visitable> models;

    public MultiTypeAdapter(List<Visitable> models) {
        this.models = models;
        this.typeFactory = new TypeFactoryForList();

    }

    @Override
    public BaseViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        Context context = parent.getContext();

        View itemView = View.inflate(context,viewType,null);
        return typeFactory.createViewHolder(viewType,itemView);
    }

    @Override
    public void onBindViewHolder(BaseViewHolder holder, int position) {
        holder.setUpView(models.get(position),position,this);
    }

    @Override
    public int getItemCount() {
        if（null == models）{
            return 0； 
        }
        return models.size();
    }


    @Override
    public int getItemViewType(int position) {
        return models.get(position).type(typeFactory);
    }

}
```

当列表中增加类型时：

- 为该类型创建实现了Visitable接口的Model类

- 创建继承于BaseViewHolder的ViewHolder（与Model类对应）

- 为TypeFactory增加type方法（与Model类对应），同时TypeFactoryForList 实现该方法

- 为TypeFactoryForList增加与列表类型对应的资源ID参数

- 修改TypeFactoryForList 中的createViewHolder方法

可以看到，虽然Adapter中的代码量减少，但总体的代码量并没减少（可能还增多了），但是和好处比起来，增加一点代码量还是值得的

- 拓展性——Adapter并不关心不同的列表类型在列表中的位置，因此对于Adapter来说列表类型可以随意增加或减少，我们只需要维护好数据源即可。

- 可维护性——不同的列表类型由不同的ViewHolder维护，相互之间互不干扰；对类型的管理都在TypeFactoryForList 中，TypeFactoryForList 中的代码量少，代码简洁，维护成本低。
避免了类的类型检查与类型转型，这点看源码就可以知道

> 转载自http://www.jianshu.com/p/1297d2e4d27a?utm_source=desktop&utm_medium=timeline