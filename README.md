# Практическая работа №10. Использование аппаратных возможностей устройства. Разрешения, уведомления, вибрация, камера.

Выполнила: Тристан Владислава Дмитриевна ИНС-б-о-24-2 

# Цель работы 

Изучить механизм работы с разрешениями в Android, научиться создавать уведомления, управлять вибрацией устройства, а также получать доступ к камере для предварительного просмотра изображения.

# Ход работы 

1.Создание проекта и подготовка манифеста 

1.1. Создан новый проект с шаблоном Empty Views Activity с названием HardwareLab

1.2. В файл AndroidManifest.xml добавлены разрешения, необходимые для варианта 9

```java
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.VIBRATE" />
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" android:maxSdkVersion="28" />

    <uses-feature android:name="android.hardware.camera" android:required="true" />

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="HardwareLab"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.HardwareLab"
        tools:targetApi="31">
        
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
    </application>

</manifest>
```

2. Запрос разрешений во время выполнения

Реализован метод для проверки и запроса опасных решений 

```java
package com.example.hardwarelab;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Bundle;
import android.widget.Toast;
import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

public class MainActivity extends AppCompatActivity {

    private static final int REQUEST_CODE_CAMERA = 100;
    private static final int REQUEST_CODE_NOTIFICATIONS = 101;
    private static final int REQUEST_CODE_STORAGE = 102;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        checkAllPermissions();
    }

    private void checkAllPermissions() {
        checkCameraPermission();
        checkNotificationPermission();
        checkStoragePermission();
    }

    // Проверка разрешения на камеру
    private void checkCameraPermission() {
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
                == PackageManager.PERMISSION_GRANTED) {
            // Разрешение уже есть, можно использовать камеру
            Toast.makeText(this, "Доступ к камере разрешен", Toast.LENGTH_SHORT).show();
        } else {
            // Объясняем, зачем нужно разрешение
            if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.CAMERA)) {
                Toast.makeText(this, "Разрешение необходимо для сканирования QR-кодов", Toast.LENGTH_LONG).show();
            }
            // Запрашиваем разрешение
            ActivityCompat.requestPermissions(this,
                    new String[]{Manifest.permission.CAMERA},
                    REQUEST_CODE_CAMERA);
        }
    }

    // Проверка разрешения на уведомления (для Android 13+)
    private void checkNotificationPermission() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            if (ContextCompat.checkSelfPermission(this, Manifest.permission.POST_NOTIFICATIONS)
                    == PackageManager.PERMISSION_GRANTED) {
                Toast.makeText(this, "Доступ к уведомлениям разрешен", Toast.LENGTH_SHORT).show();
            } else {
                if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.POST_NOTIFICATIONS)) {
                    Toast.makeText(this, "Разрешение необходимо для уведомлений о низком остатке", Toast.LENGTH_LONG).show();
                }
                ActivityCompat.requestPermissions(this,
                        new String[]{Manifest.permission.POST_NOTIFICATIONS},
                        REQUEST_CODE_NOTIFICATIONS);
            }
        }
    }

    // Проверка разрешения на запись в хранилище (для Android 10 и ниже)
    private void checkStoragePermission() {
        if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.P) {
            if (ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE)
                    == PackageManager.PERMISSION_GRANTED) {
                Toast.makeText(this, "Доступ к хранилищу разрешен", Toast.LENGTH_SHORT).show();
            } else {
                if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.WRITE_EXTERNAL_STORAGE)) {
                    Toast.makeText(this, "Разрешение необходимо для сохранения данных", Toast.LENGTH_LONG).show();
                }
                ActivityCompat.requestPermissions(this,
                        new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE},
                        REQUEST_CODE_STORAGE);
            }
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);

        if (requestCode == REQUEST_CODE_CAMERA) {
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                Toast.makeText(this, "Доступ к камере получен", Toast.LENGTH_SHORT).show();
            } else {
                Toast.makeText(this, "Нет доступа к камере. Сканирование QR-кодов недоступно.", Toast.LENGTH_LONG).show();
            }
        }

        if (requestCode == REQUEST_CODE_NOTIFICATIONS) {
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                Toast.makeText(this, "Доступ к уведомлениям получен", Toast.LENGTH_SHORT).show();
            } else {
                Toast.makeText(this, "Нет доступа к уведомлениям", Toast.LENGTH_LONG).show();
            }
        }

        if (requestCode == REQUEST_CODE_STORAGE) {
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                Toast.makeText(this, "Доступ к хранилищу получен", Toast.LENGTH_SHORT).show();
            } else {
                Toast.makeText(this, "Нет доступа к хранилищу", Toast.LENGTH_LONG).show();
            }
        }
    }
}
```

3. Создание уведомления

3.1. Создан метод для создания канала уведомлений

3.2. Создан метод для отправки простого уведомления понажатию кнопки

```java
package com.example.hardwarelab;

import android.app.NotificationChannel;
import android.app.NotificationManager;
import android.content.Context;
import android.os.Build;
import android.os.Bundle;
import androidx.core.app.NotificationCompat;
import androidx.core.app.NotificationManagerCompat;

public class MainActivity extends AppCompatActivity {

    private static final String CHANNEL_ID_LOW_STOCK = "low_stock_channel";
    private static final String CHANNEL_ID_REMINDER = "reminder_channel";
    private static final int NOTIFICATION_ID_LOW_STOCK = 100;
    private static final int NOTIFICATION_ID_REMINDER = 101;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        createNotificationChannels();
    }

    // 1. Создание каналов уведомлений (для Android 8+)
    private void createNotificationChannels() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            
            // Канал для уведомлений о низком остатке
            NotificationChannel lowStockChannel = new NotificationChannel(
                    CHANNEL_ID_LOW_STOCK,
                    "Низкий остаток товаров",
                    NotificationManager.IMPORTANCE_HIGH
            );
            lowStockChannel.setDescription("Уведомления о товарах с низким остатком");
            lowStockChannel.enableVibration(true);
            lowStockChannel.setVibrationPattern(new long[]{0, 500, 200, 500});
            
            // Канал для обычных напоминаний
            NotificationChannel reminderChannel = new NotificationChannel(
                    CHANNEL_ID_REMINDER,
                    "Напоминания",
                    NotificationManager.IMPORTANCE_DEFAULT
            );
            reminderChannel.setDescription("Обычные напоминания");
            reminderChannel.enableVibration(false);
            
            NotificationManager notificationManager = getSystemService(NotificationManager.class);
            notificationManager.createNotificationChannel(lowStockChannel);
            notificationManager.createNotificationChannel(reminderChannel);
        }
    }

    // 2. Уведомление о низком остатке (вызывается автоматически)
    private void sendLowStockNotification(String productName, int quantity) {
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this, CHANNEL_ID_LOW_STOCK)
                .setSmallIcon(android.R.drawable.ic_dialog_alert)
                .setContentTitle("Низкий остаток товара")
                .setContentText(productName + " осталось " + quantity + " шт. Пополните запас!")
                .setPriority(NotificationCompat.PRIORITY_HIGH)
                .setAutoCancel(true)
                .setVibrate(new long[]{0, 500, 200, 500});
        
        NotificationManagerCompat notificationManager = NotificationManagerCompat.from(this);
        notificationManager.notify(NOTIFICATION_ID_LOW_STOCK, builder.build());
    }

    // 3. Уведомление по нажатию кнопки (ручное создание)
    private void sendManualNotification() {
        createNotificationChannels();
        
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this, CHANNEL_ID_REMINDER)
                .setSmallIcon(android.R.drawable.ic_dialog_info)
                .setContentTitle("Напоминание")
                .setContentText("Проверьте остатки товаров на складе")
                .setPriority(NotificationCompat.PRIORITY_DEFAULT)
                .setAutoCancel(true);
        
        NotificationManagerCompat notificationManager = NotificationManagerCompat.from(this);
        notificationManager.notify(NOTIFICATION_ID_REMINDER, builder.build());
    }

    // 4. Тестовый метод для отправки уведомления по кнопке
    private void setupNotificationButton() {
        Button btnTestNotification = findViewById(R.id.btnTestNotification);
        if (btnTestNotification != null) {
            btnTestNotification.setOnClickListener(v -> sendManualNotification());
        }
    }
}
```
4. Управление вибрацией

Добавлена кнопка "Вибрация", при нажатии на которую устройство вибрирует по заданному паттерну

```java
import android.os.Build;
import android.os.VibrationEffect;
import android.os.Vibrator;
import android.content.Context;

private void vibrate() {
    Vibrator vibrator = (Vibrator) getSystemService(Context.VIBRATOR_SERVICE);
    
    if (vibrator != null && vibrator.hasVibrator()) {
        long[] pattern = {0, 500, 200, 500, 200, 1000}; // пауза, вибрация, пауза, вибрация, пауза, вибрация
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            vibrator.vibrate(VibrationEffect.createWaveform(pattern, -1));
        } else {
            vibrator.vibrate(pattern, -1);
        }
    }
}

// Паттерн с повторением (вибрация будет повторяться)
private void vibrateRepeating() {
    Vibrator vibrator = (Vibrator) getSystemService(Context.VIBRATOR_SERVICE);
    
    if (vibrator != null && vibrator.hasVibrator()) {
        long[] pattern = {0, 500, 300, 500};
        int repeatIndex = 0; // повторять с первого элемента паттерна
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            vibrator.vibrate(VibrationEffect.createWaveform(pattern, repeatIndex));
        } else {
            vibrator.vibrate(pattern, repeatIndex);
        }
    }
}

// Остановка вибрации
private void stopVibration() {
    Vibrator vibrator = (Vibrator) getSystemService(Context.VIBRATOR_SERVICE);
    if (vibrator != null) {
        vibrator.cancel();
    }
}
```

5. Предварительный просмотр камеры

5.1. Создана новая Activity CameraActivity с разметкой activity_camera.xml, содержащей SurfaceView.

5.2. При создании поверхности камера открывается, ей передаётся держатель поверхности.

5.3. Добавлена кнопка на главный экран для перехода к CameraActivity.

CameraActivity

```java
package com.example.hardwarelab;

import android.hardware.Camera;
import android.os.Bundle;
import android.util.Log;
import android.view.SurfaceHolder;
import android.view.SurfaceView;
import androidx.appcompat.app.AppCompatActivity;
import java.io.IOException;

public class CameraActivity extends AppCompatActivity implements SurfaceHolder.Callback {
    private Camera camera;
    private SurfaceHolder surfaceHolder;

    @SuppressWarnings("deprecation")
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_camera);

        SurfaceView surfaceView = findViewById(R.id.surfaceView);
        surfaceHolder = surfaceView.getHolder();
        surfaceHolder.addCallback(this);
    }

    @SuppressWarnings("deprecation")
    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        try {
            camera = Camera.open(0);
            camera.setPreviewDisplay(holder);
        } catch (IOException e) {
            Log.e("Camera", "Ошибка настройки превью", e);
        }
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
        if (camera != null) camera.startPreview();
    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        if (camera != null) {
            camera.stopPreview();
            camera.release();
            camera = null;
        }
    }
}
```
<img width="378" height="833" alt="image_2 (1)" src="https://github.com/user-attachments/assets/5357697f-07d6-45de-9951-b774ca9a31bf" />

<img width="371" height="830" alt="image_4" src="https://github.com/user-attachments/assets/67e3ea56-12a3-4269-94e7-22f5768fb7be" />

# Контрольные вопросы

1. В чём разница между нормальными и опасными разрешениями? Приведите примеры.
   
Нормальные разрешения (Normal permissions) — не представляют угрозы для приватности пользователя, предоставляются автоматически при установке приложения. Примеры: INTERNET, ACCESS_NETWORK_STATE.

Опасные разрешения (Dangerous permissions) — предоставляют доступ к конфиденциальным данным пользователя, требуют запроса во время выполнения приложения. Примеры: CAMERA, RECORD_AUDIO, READ_CONTACTS, ACCESS_FINE_LOCATION, READ_EXTERNAL_STORAGE.

2.Как запросить опасное разрешение во время выполнения приложения? Опишите последовательность действий.

Проверить наличие разрешения: checkSelfPermission()

Если разрешения нет, запросить его: requestPermissions() (опционально предварительно объяснить пользователю необходимость через shouldShowRequestPermissionRationale())

Обработать результат в методе onRequestPermissionsResult()

3. Для чего нужен NotificationChannel в Android 8.0 и выше?
   
NotificationChannel (канал уведомлений) необходим для группировки и управления уведомлениями в Android 8.0 (API 26) и выше. Без создания канала уведомления не будут отображаться. Канал определяет важность уведомлений, звук, вибрацию и другие параметры.

4. Как создать простое уведомление и отобразить его?

```java
NotificationCompat.Builder builder = new NotificationCompat.Builder(this, "CHANNEL_ID")
    .setSmallIcon(R.drawable.ic_notification)
    .setContentTitle("Заголовок уведомления")
    .setContentText("Текст уведомления")
    .setPriority(NotificationCompat.PRIORITY_DEFAULT);

NotificationManagerCompat notificationManager = NotificationManagerCompat.from(this);
notificationManager.notify(1, builder.build()); // 1 — уникальный ID
```

5. Какие методы класса Vibrator используются для создания вибрации? Как создать вибрацию с заданным паттерном?
   
Простая вибрация: vibrator.vibrate(500) — вибрация 500 мс

6.Как получить доступ к камере для предварительного просмотра? Какие классы для этого используются?

Необходимые шаги:

Добавить разрешение CAMERA в манифест
Запросить разрешение во время выполнения
Создать SurfaceView для отображения превью
Получить доступ к камере через Camera.open() и передать SurfaceHolder
Используемые классы: Camera (или Camera2), SurfaceView, SurfaceHolder, SurfaceHolder.Callback

7. Что произойдёт, если попытаться использовать опасное разрешение без его запроса во время выполнения на Android 6.0+?
   
Приложение завершит работу с исключением SecurityException. Разрешение не будет предоставлено, и операция, требующая доступа, не выполнится.

8. Как проверить, есть ли у приложения определённое разрешение в данный момент?

```java
if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) 
    == PackageManager.PERMISSION_GRANTED) {
    // Разрешение есть
} else {
    // Разрешения нет
}
```
