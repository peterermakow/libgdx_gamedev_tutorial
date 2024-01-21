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
*Пояснение по перегруженному методу draw:*

Для создания **эффекта мерцания звезд**  в этом же цикле рисуем с малой вероятности одну и ту же звезду поверх со scale*2. Оптимальная вероятность **MathItils.random(300) < 1**

### Создадим космический корабль

Реализуем управляемый космический корабль (далее КК), который будет двигаться с все возрастающим ускорением и плавно замедляться, если не придавать ему никакого ускорения. Создадим класс **Hero**. У КК будет *position*, скорость *velocity*, мощность двигателя *enginePower*, а также угол *angle*, показывающий куда направлен КК. Позже прикрутим другие поля, в том числе касающиеся стрельбы. 

В конструкторе проинициализируем указанные поля следующим образом: КК должен появлятиься в середине экрана, изначальная скорость нулевая, мощность двигателя 500.0f, а angle равен 0.0f. Также проинициализируем текстуру и gameController. Метод *render(SpriteBatch batch)* также как и в прошлом классе отрисует текстуру КК на SpriteBatch. Метод *update(float dt)* отрабатывает каждый фрейм. В нем отслеживаются нажатия на кнопки. Например, ниже приведен пример нажатий на клавиши A и D.
```java
public void update(float dt) {
   if(Gdx.input.isKeyPressed(Input.Keys.A)) {
      angle += 180*dt;
   }
   if(Gdx.input.isKeyPressed(Input.Keys.D)) {
      angle -= 180*dt;
   }
}
```

**Как сделать ускорение и замедление космического корабля?**

Допустим у нас есть вектор местоположения объекта (10,4) и вектор скорости (1,1), при их сложении получается движение объекта. Получается, что если если каждый фрейм пока нажата клавиша W мы будем суммировать векторы позиции и скорости, мы реализуем движение космического корабля. Если же при этом на каждом шаге мы будем увеличивать вектор скорости например на 0.1, то скорость будет расти. Также реализуем замедление. Движение назад реализуется аналогичным способом что и вперед, единственное рекомендуется сделать мощность двигателя поменьше(помножить на 0.7f). Также не забудем про метод checkBorders().

При нажатии на клавишу W будем увеличивать скорость КК. 
```java
if(Gdx.input.isKeyPressed(Input.Keys.W)) {
   velocity.x += MathUtils.cosdeg(angle) * enginePower * dt;
   velocity.y += MathUtils.sinDeg(angle) * enginePower * dt;
} else {
   float stopKoef = 1.0 - dt;
   if(stopKoef < 0.0f) {
      stopKoef = 0.0f;
   }
   velocity.scl(stopKoef);
}
position.mulAdd(velocity, dt);
checkBorders();
```
В методе *checkBorders* добавим фичу, согласно которой при наскоке на стенки КК будет отскакивать от них. Приведённый ниже код сильно сокращен.
```java
private void checkBorders() {
   if (position.x < 32) {
      position.x = 32;
      velocity.x *= -0.25f;
   }
}
```

**Основные методы для работы с векторами в LibGDX**
+ add() сложение векторов
+ sub() вычитание векторов
+ scl() умножение вектора на скаляр
+ len() получение длины вектора (длина вектора |a| есть корень квадратный из суммы квадратов величин x и y) -> длина вектора (3,4) = 5
+ nor() нормирование вектора (есть вычисление такого вектора, кот направлен туда же, куда и исходный, но при этом его длина равна 1)
+ cpy() копирование вектора
+ dot() скалярное прозиведение

