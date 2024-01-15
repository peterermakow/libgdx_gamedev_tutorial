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

Класс **DesktopLauncher** запускает приложение на экране. Gdx-Setup автоматически создаст **Lwjgl3ApplicationConfiguration**, в него необходимо передать параметры окна приложения(fps, title, width, height, resizable и проч.). 
Конфигурация и главный класс игры(в нашем случае StarGame) передаются в качестве параметров в **Lwjgl3Application**. Укажите следующие размера окна и максимальный FPS:
```java
config.setForegroundFPS(60);
config.setWindowedMode(1280, 720);
```

Изначально главный класс игры наследуется от *ApplicationAdapter*, сразу же поменяем это так, чтобы он наследовался от класса *Game*. Класс StarGame является "запускателем" игры. В настоящий момент в классе StarGame нам необходимы следующие методы и поля:
+ **SpriteBatch** является областью, в которой происходит отрисовка элементов
+ метод **create** (он запускается при старте приложения)
+ метод **render** отрабатывает каждый фрейм(в идеальных условиях 60 раз в секунду)
+ метод **dispose** при закрытии программы очищает ресурсы (LibGDX использует GPU, а Garbage Collectors не умеют чистить видеопамять)
+ также нам необходимо поле игрового экрана **GameScreen**
  
Игровой экран (GameScreen) работает непосредственно с игрой. Помимо игрового экрана позднее мы реализуем экран игрового меню и прочие экраны. Все они будут наследоваться от абстрактного класса **AbstractScreen**, который, в свою очередь, реализует интерфейс **Screen**. Создадим абстрактный класс AbstractScreen, имплеметируем у него интерфейс Screen. Интерфейс имеет набор методов: *show*, *hide*, *pause*, *resume*, *render(float dt)* и прочие.
```java
public abstract class AbstractScreen implements Screen {...}
```

Далее создадим класс игрового экрана **GameScreen**, который принимает в конструкторе SpriteBatch




