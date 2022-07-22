# ml-game-framework
java课作业的附属产物

内容非常的粗糙，属于我学习过程中瞎鼓捣的玩意，算是半个2D java游戏框架叭。具体例程：[坦克大战](https://github.com/Masterlong-Dragon/scu-java-battlecity

资源管理目前只能读取一张大的精灵图集。

## 基本使用

- 加载资源，创建游戏物体，场景和窗体

  - 加载资源

    - ```java
      public class Resource extends ResourceHandler {
      
          //基于枚举的安全单例 (枚举类型可以避免多线程问题)
          private enum Singleton {
              INSTANCE;
              private final Resource instance;
              private final TileMapSys mapInstance;
      
              Singleton() {
                  instance = new Resource("sprite.png");
              }
      
              private ResourceHandler getResourceHandlerInstance() {
                  return instance;
              }
      
              private TileMapSys getMapInstance() {
                  return mapInstance;
              }
          }
      
          public static ResourceHandler getResourceHandlerInstance() {
              return Singleton.INSTANCE.getResourceHandlerInstance();
          }
      
          public static TileMapSys getMapInstance() {
              return Singleton.INSTANCE.getMapInstance();
          }
      
          public Resource(String path) {
              super(path);
              File f = new File(path);
              try {
                  res = ImageIO.read(f);
              } catch (IOException e) {
                  e.printStackTrace();
              }
              staticResources.put("texture", res);
              // 贴图...
              staticResources.put("myTexture", new ImgArea(170, 204, 10, 10, 170 + 34, 204 + 34));
          }
      }
      ```

      

  - 创建游戏物体

    - 继承GameItem，实现方法即可

    - 注意几个关键属性，例如pos代表了显示位置，imgArea为默认使用的显示区域，详见坦克大战例程

    - ```java
      public class MyItem extends GameItem{
      
          public MyItem(Direction pos) {
              super(pos, Resource.getResourceHandlerInstance());
              // 显示层
              layer = 3;
              initImgArea();
          }
      
          public MyItem(int x, int y) {
              super(x, y, Resource.getResourceHandlerInstance());
              // 显示层
              layer = 3;
              initImgArea();
          }
      
          void initImgArea() {
              imgArea = (ImgArea) resourceHandler.getResource("myTexture");
          }
      
          @Override
          public void init() {
              super.init();
              scene.attachComponent(this, "xxx");//添加到组件管理器
          }
      
          @Override
          public void update(long clock) {
              super.update(clock);
              // 操作
              pos.x++;
          }
      
          @Override
          public void draw(Graphics g) {
              // 默认绘制imgArea属性范围
              // 可重写实现其它效果
              super.draw(g);
          }
      
          @Override
          public void bind(DrawManager d) {
              super.bind(d);
          }
      
          @Override
          public void onDestroy() {
              super.onDestroy();
              System.out.println(getID() + " destroyed.");
              // 如果需要的话，从添加过的组件管理器移除自身
              scene.detachComponent(this, "xxx");
          }
      
      }
      
      ```

      

  - 创建场景

    - ```java
      public class MainScene extends Scene {
          public MainScene(GameView gameView) {
              super(gameView);
          }
      	
          @Override
          public Scene registerEvents() {
              EventRegister eventRegister = SingletonManager.getEventRegister();
              eventRegister.unregisterAll();
              // 基础事件注册，没有就略去
              IGameEvent event = new IGameEvent() {
                  @Override
                  public void run(Object... args) {
                  	// do sth
                  }
      
                  @Override
                  public Object getInfo(Object... args) {
                      return false;
                  }
              };
              // 添加到事件注册器
              eventRegister.register("event", event);
              return super.registerEvents();
          }
      
          // 初始化场景
          @Override
          public void init(Object... args) {
              super.init();
              ResourceHandler resourceHandler = Resource.getResourceHandlerInstance();
              // 初始化场景组件
              addComponent(new CollisionManager(width, height))
                      .addComponent(new InputManager(gameView.getGameFrame()));
              // 初始化游戏物体
              addItem(new MyItem());
              // Debug帮助组件  /////////////////////////////////////////////////////////////////////////////////////////////////
              GameDebug gameDebug = new GameDebug(this);
              findComponent("input").addItem(gameDebug);
              drawManager.addItem(gameDebug);
              ////////////////////////////////////////////////////////////////////////////////////////////////
              //对象池分配资源（如果需要的话）
              IGameEvent releaseToPool = eventRegister.getEvent("releaseToPool");
              resourceHandler.initPoolByName("items", new IPoolCreate() {
                          @Override
                          public GameItem createItem() {
                              MyItem item = new MyItem();
                              item.setPoolName("items");
                              return bullet.addEvent(GameItem.ON_DESTROY, releaseToPool);
                          }
                      }
                      , 100);
          }
      }
      ```

  - 创建窗体

    - 继承GameFrame实现MyGameFrame

      - 初始化并添加场景到场景管理器

      - ```java
        public class MyGameFrame extends GameFrame {
        
            public myGameFrame(String str, int w, int h) {
                super(str, w, h);
                setBackground(Color.BLACK);
                gameView = new GameView(w, h, Resource.getResourceHandlerInstance());
                setGameView(gameView);
                Scene scene1 = new MainScene(gameView);
                SceneManager sceneManager = SingletonManager.getSceneManager();
                sceneManager.addScene(scene1, "1");
                sceneManager.setActiveScene("1", false, true);
            }
        
            @Override
            public void paint(Graphics g) {
                super.paint(g);
                //只是用来抹平底下莫名其妙的白线
                g.drawLine(0, getHeight() - 7, getWidth(), getHeight() - 7);
            }
        }
        ```

        

    - ```java
      public static void main(String[] args) {
              // write your code here
              // 主游戏视图
              GameFrame frame = new MyGameFrame("我的游戏", 800, 600 + 20);
          }
      ```

      

## 类名解释

- essentials
  - resources
    - ImgArea：定义了在唯一的大资源图集上的资源矩形
    - IPoolCreate：对象池创建方法接口
    - ResourceHandler：全局资源管理器，读取精灵图集，必须被继承以实现具体内容，必须为单例；实现了一个游戏物体的对象池管理
  - IGameEvent：游戏中需要被操作的事件
  - IResDraw：需要被绘制物体的接口
  - IUpdate：需要逻辑更新物体的接口
  - DrawManager：管理可绘制物体（IResDraw）
  - LogicManager：管理逻辑更新物体（IUpdate）
  - AliveMonitor：管理当前被暂时挂起的游戏组件，继承自GameComponentSystem
  - EventRegister：注册全局事件
  - SingletonManager：管理框架定义的单例对象
  - GameFrame：游戏主窗口，继承自JFrame
  - GameView：游戏绘制部分，继承自JPanel
  - GameItem：游戏物体，具有基本的一些属性，实现IResDraw和IUpdate
  - GameDebug：实现IInput和IResDraw，可显示一些框架内组件等运行管理状态
  - Scene：场景
  - SceneManager：场景切换管理
- components
  - AABB：aabb包围盒
  - IColider：碰撞物体接口
  - CollidableGameItem：简单封装后支持碰撞的游戏物体，继承GameItem
  - IInput：接受输入的物体接口
  - CollisionManager：碰撞对象管理组件，继承自GameComponentSystem
  - InputManager：输入响应管理组件，继承自GameComponentSystem
  - Animation：帧动画存储（只存储大图集上的ImgArea位置）
  - AnimationItem：简单封装后的动画物体，继承GameItem
  - AnimationPlayer：操作Animation对象，决定播放内容
  - PeriodicTimer：存储并管理一系列IGameEvent的定时调用，需要由继承IUpdate的组件自行管理传入时钟更新
- maths
  - Direction：实现了基本的向量运算操作
  - Rigid：和刚体模拟无关，通过计算AABB盒相对位置判断阻拦关系，有bug
