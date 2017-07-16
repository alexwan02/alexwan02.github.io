---
title: Android架构：MVP设计模式
date: 2016-01-10 00:17:32
tags:
---
[[译]Android开发中的MVP架构
](http://www.jianshu.com/p/7567ed0d1853?utm_campaign=maleskine&utm_content=note&utm_medium=writer_share&utm_source=weibo)

[用MVP架构开发Android应用](http://kymjs.com/code/2015/11/09/01/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

[一种在android中实现MVP模式的新思路](https://github.com/alexwan1989/android-tech-frontier/blob/master/androidweekly%2F%E4%B8%80%E7%A7%8D%E5%9C%A8android%E4%B8%AD%E5%AE%9E%E7%8E%B0MVP%E6%A8%A1%E5%BC%8F%E7%9A%84%E6%96%B0%E6%80%9D%E8%B7%AF%2Freadme.md)
### 一、MVP是什么
MVP代表Model，View和Presenter

1. View层负责处理用户事件和视图部分的展示。在Android中，它可能是Activity或者Fragment类。
2. Model层负责访问数据。数据可以是远端的Server API，本地数据库或者SharedPreference等。
3. Presenter层是连接（或适配）View和Model的桥梁。

**核心**
	
高层接口不能，不应该，并且必须不了解底层接口的细节，是（面向）抽象的，并且是细节隐藏的。

### 二、分层

#### 1.View/UI Thread
1. 所有的细节所在
2. 如数据库细节，Web框架细节，等等
	
#### 2.Presenter/Adapter Controllers
1. 将Use Case和Entity中的数据转换成格式最方便的数据
2. 外部系统，如数据库或网页能够方便的使用这些数据
3. 完全包含GUI的MVC架构

#### 3.UserCase/Business Logic
1. 包含特定于应用程序的业务规则
2. 精心编排流入Entity或从Entity流出的数据
3. 指挥Entity直接使用项目范围内的业务规则，从而实现Use Case的目标

#### 4.Entity/Data(eg. Json，SQL)
1. 可以是一个持有方法函数的对象
2. 可以是一组数据结构或方法函数
3. 它并不重要，能在项目中被不同应用程序使用即可

这些模式的动机都是一样的。那就是如何避免复杂混乱的代码，让执行单元测试变得容易，创造高质量应用程序。

### 三、实现
#### 1.[AndroidMVP](https://github.com/antoniolg/androidmvp)
(1).首先需要定义一个View层接口，让View实现类Activity(Fragment)实现
```java
public interface LoginView {
    public void showProgress();
    public void hideProgress();
    public void setUsernameError();
    public void setPasswordError();
}
```
(2).定义一个Presenter实现接口，让Presenter实现类实现；Presenter中通过构造时传入的视图层对象操作View

`LoginPresenter`
```java
public interface LoginPresenter {
    void validateCredentials(String username, String password);
    void onDestroy();
}
```
`LoginPresenterImpl`
```java
public class LoginPresenterImpl implements LoginPresenter, OnLoginFinishedListener {	
    private LoginView loginView;
    ...

    public LoginPresenterImpl(LoginView loginView) {
        this.loginView = loginView;
        this.loginInteractor = new LoginInteractorImpl();
    }
	
    @Override 
    public void validateCredentials(String username, String password) {
        if (loginView != null) {
            loginView.showProgress();
        }
        loginInteractor.login(username, password, this);
    }
	
    @Override 
    public void onDestroy() {
        loginView = null;
    }
}
```
(3).在View实现类Activity(Fragment)中包含Presenter对象，并在Presenter创建的时候传一个View对象
```java
public class A extends Activity implements LoginView, OnClickListener {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // ...
        // 省略初始化控件
        // ...
        presenter = new LoginPresenterImpl(this);
    }
    //...省略众多接口方法
}
```
#### 2. Activity和Fragment

Activity 有一个很复杂的生命周期(fragment的生命周期可能会更复杂), 而这些生命周期很有可能对你项目的业务逻辑有非常重大的影响. Activity 可以获取上下文环境和多种android系统服务. Activity中发送Intent，启动Service和执行FragmentTransisitons等。而这些特性在我看来绝不应该是视图层应该涉及的领域(视图的功能就是现实数据和从用户那里获取输入数据，在理想的情况下，视图应该避免业务逻辑).Activity和Fragment不适合作为View

1. Activity和Fragment作为presenters

(1). 去除所有的view

```java
public interface IBaseView {
    void init(LayoutInflater inflater , ViewGroup viewGroup);
    View getView();
}
```
(2). 创建一个presenter基类 (Activity)
```java
public abstract class BasePresenterActivity<V extends IBaseView> extends AppCompatActivity {
    protected V view;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        try {
            view = getViewClass().newInstance();
            view.init(getLayoutInflater(), null);
            setContentView(view.getView());
            onBindView();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }

    }

    @Override
    protected void onResume() {
        super.onResume();
    }

    @Override
    protected void onDestroy() {
        onDestroyView();
        view = null;
        super.onDestroy();
    }

    /**
     * 获取View对应的Class
     * @return View类
     */
    protected abstract Class<V> getViewClass();

    /**
     * 绑定View
     */
    protected void onBindView(){}

    /**
     * 清除View
     */
    protected void onDestroyView(){}
}
```

(3). 创建一个基本的presenter(Fragment)
```java
public abstract class BasePresenterFragment<V extends IBaseView> extends Fragment {
    protected V view;
    protected TabLayout tabLayout;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = null;
        try {
            this.view = getViewClass().newInstance();
            this.view.init(inflater , container);
            view = this.view.getView();
            onBindView();
        } catch (java.lang.InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        return view;
    }

    @Override
    public void onDestroyView() {
        onDestroyVU();
        this.view = null;
        super.onDestroyView();
    }

    protected void onDestroyVU(){}
    protected void onBindView(){}
    protected void setTabLayout(TabLayout tabLayout){};
    protected abstract Class<V> getViewClass();
}
```
(4). 使用
```java
/**
* MainActivityView
*/
public class IMainActivityView implements IBaseView {
    private View mView;
    @Bind(R.id.tool_bar)
    Toolbar mToolbar;
    @Bind(R.id.drawer_layout)
    DrawerLayout mDrawerLayout;
    @Bind(R.id.root_layout)
    CoordinatorLayout mRootLayout;
    ActionBarDrawerToggle drawerToggle;
    @Bind(R.id.navigation)
    NavigationView navigationView;
    @Bind(R.id.tab_layout)
    TabLayout tabLayout;
    @Bind(R.id.fab_btn)
    FloatingActionButton fabBtn;
    @Override
    public void init(LayoutInflater inflater, ViewGroup viewGroup) {
        mView = inflater.inflate(R.layout.activity_main, viewGroup, false);
        ButterKnife.bind(this, mView);
    }

    @Override
    public View getView() {
        return mView;
    }

    /**
     * 初始化界面
     *
     * @param activity activity
     */
    public void initViews(final MainActivity activity) {
        // toolbar
        activity.setSupportActionBar(mToolbar);
        ActionBar actionBar = activity.getSupportActionBar();
        if (actionBar != null) {
            actionBar.setDisplayShowHomeEnabled(true);
            actionBar.setDisplayShowTitleEnabled(true);
            actionBar.setDisplayHomeAsUpEnabled(true);
        }
        // drawer toggle
        this.drawerToggle = new ActionBarDrawerToggle(activity, mDrawerLayout, R.string.tool_name, R.string.tool_name);
        drawerToggle.setDrawerIndicatorEnabled(true);
        mDrawerLayout.setDrawerListener(drawerToggle);
        navigationView.setCheckedItem(R.id.main_frame);
        // fragment
        final HomeFragment homeFragment = new HomeFragment();
        homeFragment.setTabLayout(tabLayout);
        final SettingFragment settingFragment = new SettingFragment();
        final FragmentTransaction transaction = activity.getSupportFragmentManager().beginTransaction();
        transaction.replace(R.id.frame_layout , homeFragment).commit();
        // navigation
        navigationView.setNavigationItemSelectedListener(new OnNavigationItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(MenuItem item) {
                int id = item.getItemId();
                mDrawerLayout.closeDrawers();
                navigationView.setCheckedItem(id);
                FragmentTransaction transaction = activity.getSupportFragmentManager().beginTransaction();
                switch (id) {
                    case R.id.main_frame:
                        tabLayout.setVisibility(View.VISIBLE);
                        transaction.replace(R.id.frame_layout , homeFragment).commit();
                        break;
                    case R.id.message_frame:
                        tabLayout.setVisibility(View.GONE);
                        transaction.replace(R.id.frame_layout , settingFragment).commit();
                        break;
                    case R.id.search_frame:
                        tabLayout.setVisibility(View.GONE);
                        transaction.replace(R.id.frame_layout , settingFragment).commit();
                        break;
                    default:
                        break;
                }
                return false;
            }
        });
    }

    public void onPostCreate() {
        drawerToggle.syncState();
    }

    public void onConfigurationChanged(Configuration newConfig) {
        drawerToggle.onConfigurationChanged(newConfig);
    }

    public boolean onOptionsItemSelected(MenuItem item) {
        return drawerToggle.onOptionsItemSelected(item);
    }
}
```

MainActivity继承封装View接口的BasePresenterActivity，实现Presenter具体业务逻辑
```Java
public class MainActivity extends BasePresenterActivity<IMainActivityView> {

    private FragmentTransaction transaction;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        
        super.onCreate(savedInstanceState);
    }

    @Override
    protected void onBindView() {
        super.onBindView();
        view.initViews(this);
    }

    @Override
    protected void onDestroyView() {
        super.onDestroyView();
    }

    @Override
    public void onPostCreate(Bundle savedInstanceState) {
        super.onPostCreate(savedInstanceState);
        view.onPostCreate();
    }

    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
        view.onConfigurationChanged(newConfig);
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.navigation_menu , menu);
        return super.onCreateOptionsMenu(menu);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        if(view.onOptionsItemSelected(item))
            return true;
        int id = item.getItemId();
        switch (id){
            case R.id.main_frame:
                transaction.add(new HomeFragment() , "home_fragment");
                break;
            case R.id.oauth_frame:
                Intent intent = new Intent(MainActivity.this, WBAuthActivity.class);
                MainActivity.this.startActivity(intent);
                break;
            default:
                break;
        }
        return super.onOptionsItemSelected(item);
    }

    @Override
    protected Class<IMainActivityView> getViewClass() {
        return IMainActivityView.class;
    }
}
```

Code：[Github](https://github.com/wangwei1121/iweibo)
