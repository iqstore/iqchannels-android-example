IQChannels Android SDK
==================
SDK для Андроида сделано как обычная андроид-библиотека, которую можно поставить с помощью Gradle.
Библиотека легко интегрируется как обычная зависимость в существующее приложение.

Требования:
* minSdkVersion 11 (3.0).
* targetSdkVersion 25.


Установка
---------
Добавить репозиторий с библиотекой в `build.gradle` всего проекта в раздел `allProjects`. 
Это временное требование, в ближайшем будущем библиотека появится в центральном репозитории jCenter.

```build.gradle
allprojects {
    repositories {
        jcenter()
        maven {
            url "https://dl.bintray.com/iqstore/maven/"
        }
    }
}
```

Добавить зависимосить `compile 'ru.iqstore:iqchannels-sdk:0.9.0'` в `build.gradle` модуля приложнеия.
```build.gradle
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])   
    compile 'ru.iqstore:iqchannels-sdk:0.9.0'
    // etc...
}
```

Собрать проект, `gradle` должен успешно скачать все зависимости. 


Конфигурация
------------
Приложение разделено на два основных класса: `IQChannels` и `ChatFragment`. Первый представляет собой библиотеку, 
которая реализуюет бизнес-логику. Второй - это фрагмент, который реализует пользовательский интерфейс чата.

Для использования SDK его нужно сконфигурировать. Конфигурировать можно в любом месте приложения, 
где доступен контекст андроида. `ChatFragment` можно спокойно использовать до конфигурации.
Для конфигурации нужно передать адрес чат-сервера и английское название канала в `IQChannels.instance().configure()`.

Пример конфигурации в `Activity.onCreate`.
```java
public class MainActivity extends AppCompatActivity
        implements NavigationView.OnNavigationItemSelectedListener {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        setupIQChannels();
    }

    private void setupIQChannels() {
        // Настраиваем сервер и канал iqchannels.
        IQChannels iqchannels = IQChannels.instance();
        iqchannels.configure(this, new IQChannelsConfig("http://192.168.31.158:3001/", "support"));
    }
}
```


Логин
-----
Логин/логаут пользователя осуществляется по внешнему токену, специфичному для конкретного приложения.
Для логина требуется вызвать в любом месте приложения:

```java
IQChannels.instance().login("isimple chat token");
```

Для логаута:
```java
IQChannels.instance().logout();
```

После логина внутри SDK авторизуется сессия пользователя и начинается бесконечный цикл, который подключается
к серверу и начинает слушать события о новых сообщения, при обрыве соединения или любой другой ошибке
сессия переподключается к серверу. При отсутствии сети, сессия ждет, когда последняя станет доступна.


Интерфейс чата
--------------
Интерфес чата построен как отдельный фрагмент, который можно использовать в любом месте в приложении.
Интерфейс чата корректно обрабатывает логины/логаут, обнуляет сообщения.

Пример использования в стандартном боковом меню:
```java
public class MainActivity extends AppCompatActivity
        implements NavigationView.OnNavigationItemSelectedListener {
    
    @Override
    public boolean onNavigationItemSelected(MenuItem item) {
        Fragment fragment;
        switch (item.getItemId()) {
            case R.id.nav_index:
                fragment = PlusOneFragment.newInstance();
                break;

            case R.id.nav_chat:
                // Пользователь выбрал чат в меню.
                fragment = ChatFragment.newInstance();
                break;

            default:
                fragment = PlusOneFragment.newInstance();
                break;
        }

        // Заменяем фрагмент в нашей активити.
        FragmentManager fragmentManager = getSupportFragmentManager();
        fragmentManager.beginTransaction().replace(R.id.content, fragment).commit();

        // Сворачиваем меню.
        DrawerLayout drawer = (DrawerLayout) findViewById(R.id.drawer_layout);
        drawer.closeDrawer(GravityCompat.START);
        return true;
    }
}
```


Отображение непрочитанных сообщений
-----------------------------------
Для отображения и обновления непрочитанных сообщений нужно добавить слушателя в IQChannels.
Слушателя можно добавлять в любой момент времени, он корректно обрабатывает переподключения,
логин, логаут.

Пример добавления слушателя в `Activity.onCreate`:
```java
public class MainActivity extends AppCompatActivity
        implements NavigationView.OnNavigationItemSelectedListener,
                   UnreadListener {

    private Cancellable unreadSubscription;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        listenToUnread();
    }

    // Добавляем слушателя непрочитанных сообщений.
    private void listenToUnread() {
        unreadSubscription = IQChannels.instance().addUnreadListener(this);
    }

    // Показывает текущие количество непрочитанных сообщений.
    @Override
    public void unreadChanged(int unread) {

    }
}
```
