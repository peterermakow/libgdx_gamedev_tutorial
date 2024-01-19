## Разработка игры на LibGDX

В данном Tutorial мы поэтапно, шаг за шагом напишем игру на космическую тематику. В процессе разработки проекта постоянно будут меняться реализация и архитектура проекта от простых к более сложным, 
это сделано специально для лучшего понимания того, что мы делаем. 

Для запуска проекта требуется зайти на сайт [LibGDX](https://libgdx.com/ "LibGDX"), перейти в  раздел Documentation, далее Tools, и скачать **gdx-setup**.
В *Sub Projects* оставить только *Desktop*. Также лучше убрать чекбокс с Android, иначе проект желательно запускать в Android Studio. 
В *Extensions* оставить библиотеку **Freetype**

В результате проект получит следующую структуру из 3 модулей:
в **assets** хранятся ресурсы проекта(изображения, звуки, шрифты и пр.),
в **core** пишется основная логика приложения,
а в **desktop** указывается все, что касается экрана.

### Приступим к реализации проекта!

Класс **DesktopLauncher** запускает приложение на экране. Gdx-Setup автоматически создаст структуру проекта. В DesktopLauncher прописывается **Lwjgl3ApplicationConfiguration**, в нее необходимо передать параметры окна приложения(fps, title, width, height, resizable и проч.). 
Конфигурация и главный класс игры(в нашем случае **StarGame**) передаются в качестве параметров в **Lwjgl3Application**. Укажите следующие размера окна и максимальный FPS:
```java
config.setForegroundFPS(60);
config.setWindowedMode(1280, 720);
```

Изначально главный класс игры наследуется от *ApplicationAdapter*, сразу же поменяем это так, чтобы он наследовался от класса *Game*. Класс StarGame является "запускателем" игры. В настоящий момент в классе StarGame нам необходимы следующие методы и поля:
+ **SpriteBatch** является областью, в которой происходит отрисовка элементов
+ поле игрового экрана **GameScreen**
+ метод **create** (он запускается при старте приложения)

  в этом методе инициализируется SpriteBatch и GameScreen, а также устанавливается экран
  ```java
  setScreen(gameScreen);
  ``` 
+ метод **render** отрабатывает каждый фрейм.

  в нем высчитывается deltaTime, который далее передается в метод render у игрового экрана
  ```java
  float deltaTime = Gdx.graphics.getDeltaTime();
  getScreen().render(deltaTime);
  ```
+ метод **dispose** при закрытии программы очищает ресурсы (LibGDX использует GPU, а Garbage Collectors не умеют чистить видеопамять)

  
Игровой экран (GameScreen) работает непосредственно с игрой. Помимо игрового экрана позднее мы реализуем экран игрового меню и прочие экраны. Все они будут наследоваться от абстрактного класса **AbstractScreen**, который, в свою очередь, реализует интерфейс **Screen**. Создадим абстрактный класс AbstractScreen, имплеметируем у него интерфейс Screen. Интерфейс имеет набор методов: *show*, *hide*, *pause*, *resume*, *render(float dt)* и прочие.
```java
public abstract class AbstractScreen implements Screen {...}
```

Далее создадим класс игрового экрана **GameScreen**, который принимает в конструкторе SpriteBatch. Также в его задачи входит инициализация в методе *show* главного хаба игры (класс **GameController** хранит в себе ссылки на все игровые сущности) и класса-рисовальщика **WorldRenderer**. Последнему для своей работы необходим SpriteBatch и GameController. В методе *render* класса GameScreen, который отрабатывает каждый фрейм, происходит апдейт всех игровых сущеностей и рвызов метода *render* у WorldRenderer. 

Единственный метод класса **WorldRenderer** это *render*. Пример метода: 
```java
ScreenUtils.clear(0, 0, 0.5f, 1);
batch.begin();
hero.render(batch);
batch.end();
```
ScreenUtils.clear() каждый раз прежде чем что-либо рисовать на экран очищает его указанным в методе цветом. В данном случае цвет кодируется в так называемом float-представлении(не байтовое представление, где 0...255)
a в *rgba* означает alpha-канал, то есть прозрачность (1 прозрачность отсутствует, 0 полностью прозрачно).

Также нам понадобится утилитный класс ScreenManager, хранящий ширину и высоту экрана.

Итак, загрузим в *assets* текстуры игровых объектов.

### Создадим основной фон игры

Основной фон игры будет представлять из себя картинку космического неба и звезд, двигающихся по нему. Создадим класс **Background** и в нем же приватный класс **Star**. 
У звезды есть координаты позиции *position*, скорсоть *velocity* и размер *scale*. Проинициализируем их в конструкторе. Позиция реализуется при помощи класса **Vector2**, задается координата *x* и *y*. 
Координаты рандомные - от 0 до высоты/ширины экрана.
```java
Vector2 position = new Vector2(MathUtils.random(0, ScreenManager.WIDTH), MathUtils.random(0, ScreenManager.HEIGHT));
```
Скорость рекомендуется задать от -40 до -5 по *x*, в *y* указать 0. Для создания *эффекта глубины* сделаем быстрые звезды больше, а медленные меньше:
```java
scale = Math.abs(position.x/40f) * 0.8f;
```
В методе update(dt) каждый фрейм высчитываются новые координаты звезды, при этом если звезда ушла за пределы экрана (для верности -20), поставим её position.x в WIDTH + 20, а высоту высчитаем заново.

Вернемся в класс *Background*: объявим текстуры космического неба и звезды, объявим массив звезд *stars* и укажем что *STAR_COUNT* равен 600. В конструкторе проинициализируем текстуры, создадим массив звезд и заполним его звездами.
В методе update(dt) вызовем соответвующий метод у каждой звезды из массива *stars*, а в методе render отрисуем бэкграунд и звезды.
```java
public void render(Spritebatch batch) {
   batch.draw(SpaceTexture, 0, 0);
   for(Star star : stars) {
      batch.draw(StarTexture, star.position.x - 8, star.position.y - 8, 8, 8, 16, 16, star.scale, star.scale, 0, 0, 0, 16, 16, false, false)
   }
}
```
Для создания **эффекта мерцания звезд**  в этом же цикле рисуем с малой вероятности одну и ту же звезду еще раз поверх той же самой звезды со scale*2. Оптимальная вероятность **MathItils.random(300) < 1**

### Создадим космический корабль


