---
title: Android指东：组件通信
date: 2019-08-30 10:59:08
tags:
- android
categories:
- Android指南
---

组件间通信全方法介绍，高效的组件间通信可以让开发事半功倍。

<!-- more -->

这里我们会先略过一些通用的方法，比如广播、数据存储SharePreferences等，把注意力放在组件内部。

## 1. 向下一个Activity传递数据

`Intent` 是在相互独立的组件（如两个 `Activity`）之间提供运行时绑定功能的对象。`Intent` 表示某个应用“执行某项操作的意图”。新建一个`Intent`对象，传入希望启用的`Activity`作为参数，调用`startActivity`，即可向下一个Activity传递数据。

```
Intent intent = new Intent(this, TargetActivity.class);
// 其中value支持多种类型
intent.putExtra("key",value);
startActivity(intent);
```


## 2. 向上一个Activity返回数据

`Activity`中还有一个`startActivityForResult()`方法也是用于启动`Activity`的，但是这个方法期望在下一个`Activity`销毁的时候能够返回一个结果给上一个`Activity`。

`startActivityForResult`方法接收两个参数，第一个参数还是`Intent`，第二个参数是请求码(`requestCode`)，用于在之后的回调函数中判断数据的来源。

```
// 启动
Intent intent = new Intent(FirstActivity.this, SecondActivity.class);
startActivityForResult(intent, 1);
```

```
// 下一个Activity销毁之前
Intent intent = new Intent();
intent.putExtra("data","123");
setResult(RESULT_OK, intent);
finish()
```

在下一个`Activity`被销毁之后会调用上一个`Activity`的`onActivityResult()`方法，因此我们需要在上一个`Activity`中重写这个方法来得到返回的数据结果。

```
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data){
    switch(requestCode){
        case 1:
            if(resultCode == RESULT_OK) {
                String string = data.getStringExtra("data");
            }
            break;
        case 2:
            ....
    }
}
```

## 3. Activity与内嵌Fragment通信

### 3.1 Activity向Fragment发送数据

常见的就是使用`Bundle`来传递参数。

#### 3.1.1 基础用法

`Activity`中，创建`Fragment`对象的时候，通过`setArguments`放入数据。

```
// 通过Bundle和setArguments向MessageFragment传递初始化数据
MessageFragment messageFragment = new MessageFragment();
Bundle bundle = new Bundle();
bundle.putString("name", "My name is Jack");
messageFragment.setArguments(bundle);

DisplayFragment displayFragment = new DisplayFragment();
fragmentTransaction = getSupportFragmentManager()
        .beginTransaction()
        .add(R.id.fragment_1, messageFragment, null);
fragmentTransaction.commit();
```

在`Fragment`初始化时通过`getArguments`取出数据。

```
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fragment_message, container, false);

    // 通过getArguments获取数据
    String initStr = (String) getArguments().get("name");
    ...
    return view;
}
```

#### 3.1.2 扩展用法

```java
// Activity代码
public class MainActivity extends AppCompatActivity implements MessageFragment.Listener {
    private FragmentTransaction fragmentTransaction;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        if (findViewById(R.id.fragment_1) != null) {
            if (savedInstanceState != null) {
                return;
            }
            fragmentTransaction = getSupportFragmentManager()
                    .beginTransaction()
                    .replace(R.id.fragment_1, MessageFragment.getInstance("test"), null);
            fragmentTransaction.commit();
        }
    }
}
```

```java
// Fragment代码
public class MessageFragment extends Fragment {
    // 定义getInstance()静态方法
    public static MessageFragment getInstance(String s){
        MessageFragment messageFragment = new MessageFragment();
        Bundle bundle = new Bundle();
        bundle.putString("data",s);
        messageFragment.setArguments(bundle);
        return messageFragment;
    }


    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_message, container, false);
        String initStr = (String) getArguments().get("data");
        return view;
    }
}
```


### 3.2 Fragment向Activity返回数据

#### 3.2.1 接口回调

在`Fragment`中定义一个内部回调接口，再让包含该`Fragment`的`Activity`实现该回调接口，这样`Fragment`即可调用该回调方法将数据传给`Activity`。

当`Fragment`添加到`Activity`中时，会调用`Fragment`的方法`onAttach()`，这个方法中适合检查`Activity`是否实现了`Listener`接口，检查方法就是对传入的`Activity`的实例进行类型转换，然后赋值给我们在`fragment`中定义的接口。

```java
// Fragment代码
public class MessageFragment extends Fragment {
    private Listener listener;
    private EditText editText;
    private Button bFragment;
    private Button bActivity;
    private FragmentManager manager;

    public MessageFragment() {
        // Required empty public constructor
    }

    public interface Listener {
        void show(String string);
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_message, container, false);

        // 向Activity返回数据
        bActivity = view.findViewById(R.id.button2);
        bActivity.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                listener.show(editText.getText().toString());
            }
        });
        return view;
    }

    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        if(context instanceof Listener) {
            listener = ((Listener) context);
        } else {
            throw new IllegalArgumentException("Activity must implements Listener");
        }
    }

    @Override
    public void onDetach() {
        super.onDetach();
        listener = null;
    }
}
```

在一个`Fragment`从`Activity`中剥离的时候，就会调用`onDetach`方法，这个时候要把传递进来的`Activity`对象释放掉，不然会影响`Activity`的销毁，产生不必要的错误。

```java
// Activity代码
public class MainActivity extends AppCompatActivity implements MessageFragment.Listener {
    private FragmentTransaction fragmentTransaction;
    private TextView textView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        textView = findViewById(R.id.top);

        if (findViewById(R.id.fragment_1) != null) {
            if (savedInstanceState != null) {
                return;
            }

            MessageFragment messageFragment = new MessageFragment();
            fragmentTransaction = getSupportFragmentManager()
                    .beginTransaction()
                    .add(R.id.fragment_1, messageFragment, null);

            fragmentTransaction.commit();
        }
    }

    @Override
    public void show(String string) {
        textView.setText(string);
    }
}
```


#### 3.2.2 Handler方案

这种方法会使得`Fragment`对具体的`Activity`存在耦合，不利于`Fragment`复用不利于维护，若想删除相应的`Activity`，`Fragment`也得改动

```java
public class MainActivity extends AppCompatActivity{ 
      //声明一个Handler 
      public Handler mHandler = new Handler(){       
          @Override
           public void handleMessage(Message msg) { 
                super.handleMessage(msg);
                // ...相应的处理代码
           }
     };
   } 

public class MainFragment extends Fragment{
    // 保存Activity传递的handler
    private Handler mHandler;

    @Override
    public void onAttach(Context context) { 
        super.onAttach(context);
        // 这个地方已经产生了耦合，若还有其他的activity，这个地方就得修改 
        mHandler = ((MainActivity)context).mHandler; 
    }      
}
```

## 4. 同宿主的Fragment间通信

由于 `Activity` 持有所有内嵌的 `Fragment` 对象实例。（创建实例时保存的 `Fragment` 对象，或者通过 `FragmentManager` 类提供的`findFragmentById()`和`findFragmentByTag()` 方法也能获取到 `Fragment` 对象），所以可以直接操作`Fragment`。`Fragment` 通过 `getActivity()` 方法可以获取到宿主 `Activity` 对象（强制转换类型即可），进而可以操作宿主 `Activity`；那么很自然的，获取到宿主 `Activity` 对象的 `Fragment` 便可以操作其他 `Fragment` 对象。

```java
public class MessageFragment extends Fragment {
    private Listener listener;
    private Button bFragment;
    private FragmentManager manager;

    public MessageFragment() {
        // Required empty public constructor
    }

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // onCreate时获取宿主的FragmentManager
        manager = getFragmentManager();
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_message, container, false);

        // 向另一个Fragment传递数据
        bFragment = view.findViewById(R.id.button);
        bFragment.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // 通过manager.findFragmentByTag
                DisplayFragment displayFragment = (DisplayFragment) manager.findFragmentByTag("bottom");
                TextView textView = displayFragment.getView().findViewById(R.id.display);
                textView.setText(editText.getText());
            }
        });

        return view;
    }
}
```

虽然上述操作已经能够解决 `Activity` 与 `Fragment` 的通信问题，但会造成代码逻辑紊乱的结果，极度不符合 `“高内聚，低耦合”` 这一编程思想。`Fragment` 做好自己的事情即可，所有涉及到 `Fragment` 之间的控制显示等操作，都应交由宿主 `Activity` 来统一管理。

## 5. EventBus

`EventBus`是用反射机制实现的，性能上会有问题。使用`EventBus`实现组件间的通信主要分为3个步骤：

（1）定义事件模型对象；

```java
// 定义事件对象
public class MessageEvent {
    public String key;
    public String value;

    public MessageEvent(String key, String value) {
        this.key = key;
        this.value = value;
    }
}
```

（2）添加事件订阅者；

```
EventBus.getDefault().register(DisplayFragment.this);
```

（3）当事件触发时，进行事件发布。

```
EventBus.getDefault().post(new MessageEvent("data", editText.getText().toString()));
```

详细例子如下：

```java
// 添加订阅者
public class DisplayFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        // 订阅事件
        EventBus.getDefault().register(DisplayFragment.this);
        return inflater.inflate(R.layout.fragment_display, container, false);
    }
    
    // 定义事件的处理函数
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void messageEventBus(MessageEvent messageEvent) {
        TextView textView = getView().findViewById(R.id.display);
        textView.setText(messageEvent.value);
    }

}
```

```java
// 添加事件发布者
public class MessageFragment extends Fragment {
    private Button bBus;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_message, container, false);
        bBus = view.findViewById(R.id.button3);
        return view;
    }

    @Override
    public void onStart() {
        super.onStart();

        // 使用Event Bus
        bBus.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // 发布事件
                EventBus.getDefault().post(new MessageEvent("key", "value"));
            }
        });
    }

}
```

## 6. 更多

本文demo地址：[>>>点我进入](https://github.com/Nirvana-cn/Learning-android/tree/master/demo04-FragmentCommunication)

Activity与Fragment通信方式：[>>>点我进入](https://pangrongxian.github.io/2017/07/20/Activity%20%E4%B8%8E%20Fragment%20%E9%80%9A%E4%BF%A1%E6%96%B9%E5%BC%8F/)

Android框架之路—EventBus的使用：[>>>点我进入](https://blog.csdn.net/bskfnvjtlyzmv867/article/details/71480647)

EventBus官方Github地址：[>>>点我进入](https://github.com/greenrobot/EventBus)

