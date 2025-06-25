# دليل المطور - لعبة محاكاة الحافلة

## نظرة عامة على البنية

### الهيكل العام للمشروع:
```
bus_simulator_game/
├── app/
│   ├── src/main/
│   │   ├── java/com/example/bussimulator/
│   │   │   ├── MainActivity.java          # النشاط الرئيسي
│   │   │   ├── SplashActivity.java        # شاشة التحميل
│   │   │   ├── MenuActivity.java          # القائمة الرئيسية
│   │   │   ├── GameActivity.java          # نشاط اللعبة
│   │   │   ├── SettingsActivity.java      # شاشة الإعدادات
│   │   │   ├── Bus.java                   # كلاس الحافلة
│   │   │   ├── GameEngine.java            # محرك اللعبة
│   │   │   ├── GameThread.java            # خيط اللعبة
│   │   │   ├── Camera.java                # نظام الكاميرا
│   │   │   ├── SimpleMap.java             # الخريطة
│   │   │   ├── Passenger.java             # كلاس الراكب
│   │   │   ├── PassengerManager.java      # إدارة الركاب
│   │   │   ├── AIVehicle.java             # المركبات الذكية
│   │   │   ├── TrafficManager.java        # إدارة المرور
│   │   │   ├── GameSettings.java          # إعدادات اللعبة
│   │   │   └── SoundManager.java          # إدارة الأصوات
│   │   ├── res/
│   │   │   ├── layout/                    # ملفات التخطيط
│   │   │   ├── values/                    # القيم والألوان
│   │   │   ├── drawable/                  # الرسوم والخلفيات
│   │   │   ├── anim/                      # الرسوم المتحركة
│   │   │   └── raw/                       # الموسيقى
│   │   ├── assets/sounds/                 # ملفات الصوت
│   │   └── AndroidManifest.xml            # ملف البيان
│   ├── build.gradle                       # إعدادات البناء
│   └── CMakeLists.txt                     # إعدادات C++
├── build.gradle                           # إعدادات المشروع
└── README.md                              # دليل المشروع
```

## الكلاسات الرئيسية:

### 1. Bus.java - كلاس الحافلة
```java
public class Bus {
    // الخصائص الفيزيائية
    private float mass = 12000.0f;          // الكتلة بالكيلوغرام
    private float enginePower = 250.0f;     // قوة المحرك بالحصان
    private float maxSpeed = 80.0f;         // السرعة القصوى
    
    // نظام القير
    private TransmissionType transmissionType;
    private int currentGear = 1;
    private float currentRPM = 800.0f;
    
    // نظام الركاب
    private int maxPassengerCapacity = 40;
    private int currentPassengerCount = 0;
    
    // الدوال الرئيسية
    public void updatePhysics(float deltaTime);
    public void setThrottle(float throttle);
    public void setBrake(float brake);
    public void setSteering(float steering);
}
```

### 2. GameEngine.java - محرك اللعبة
```java
public class GameEngine extends SurfaceView {
    // المكونات الرئيسية
    private Bus bus;
    private SimpleMap gameMap;
    private Camera camera;
    private PassengerManager passengerManager;
    private TrafficManager trafficManager;
    
    // دورة اللعبة
    public void update(float deltaTime);
    public void draw(Canvas canvas);
    
    // إدارة الأحداث
    private void checkBusStops();
    private void handlePassengerExchange(BusStop stop);
}
```

### 3. PassengerManager.java - إدارة الركاب
```java
public class PassengerManager {
    private List<Passenger> waitingPassengers;
    private List<Passenger> onBusPassengers;
    
    // إدارة الركاب
    public void update(float deltaTime);
    public void boardPassengers(List<Passenger> passengers, int maxCapacity);
    public void exitPassengers(List<Passenger> passengers);
    
    // الإحصائيات
    public float getTotalRevenue();
    public float getAverageSatisfaction();
}
```

### 4. TrafficManager.java - إدارة المرور
```java
public class TrafficManager {
    private List<AIVehicle> vehicles;
    private List<SpawnPoint> spawnPoints;
    
    // إدارة المركبات
    public void update(float deltaTime, Bus playerBus);
    private void spawnRandomVehicle();
    private void removeDistantVehicles(Bus playerBus);
    
    // التحكم في الكثافة
    public void setTrafficDensity(float density);
}
```

## أنظمة اللعبة:

### 1. نظام الفيزياء
```java
// حساب القوى المؤثرة على الحافلة
private void calculateForces() {
    // قوة المحرك
    float engineForce = throttleInput * getEngineForce();
    
    // قوة الفرامل
    float brakeForce = brakeInput * maxBrakeForce;
    
    // مقاومة الهواء
    float dragForce = 0.5f * airDensity * dragCoefficient * 
                     frontalArea * speed * speed;
    
    // مقاومة التدحرج
    float rollingForce = mass * 9.81f * rollingResistance;
    
    // القوة الصافية
    float netForce = engineForce - brakeForce - dragForce - rollingForce;
    
    // التسارع
    float acceleration = netForce / mass;
}
```

### 2. نظام القير
```java
// تحديث نظام القير
private void updateTransmission() {
    switch (transmissionType) {
        case AUTOMATIC:
            updateAutomaticTransmission();
            break;
        case MANUAL:
            updateManualTransmission();
            break;
        case SEMI_AUTO:
            updateSemiAutoTransmission();
            break;
    }
}

// القير الأوتوماتيكي
private void updateAutomaticTransmission() {
    if (currentRPM > maxRPM * 0.85f && currentGear < maxGears) {
        shiftUp();
    } else if (currentRPM < maxRPM * 0.25f && currentGear > 1) {
        shiftDown();
    }
}
```

### 3. نظام الذكاء الاصطناعي
```java
// تحديث سلوك المركبة الذكية
public void update(float deltaTime, SimpleMap map, 
                  List<AIVehicle> otherVehicles, Bus playerBus) {
    
    // تحديث الحالة
    updateState(map, otherVehicles, playerBus);
    
    // تنفيذ السلوك حسب الحالة
    switch (state) {
        case DRIVING:
            updateDriving(deltaTime);
            break;
        case FOLLOWING:
            updateFollowing(deltaTime);
            break;
        case STOPPING:
            updateStopping(deltaTime);
            break;
    }
    
    // تحديث الموقع
    updatePosition(deltaTime);
}
```

## إرشادات التطوير:

### 1. إضافة ميزات جديدة
```java
// مثال: إضافة نظام الطقس
public class WeatherSystem {
    private WeatherType currentWeather;
    private float intensity;
    
    public void update(float deltaTime) {
        // تحديث حالة الطقس
        updateWeatherConditions(deltaTime);
        
        // تأثير على الفيزياء
        applyWeatherEffects();
        
        // تأثير على الرؤية
        updateVisibility();
    }
    
    private void applyWeatherEffects() {
        if (currentWeather == WeatherType.RAIN) {
            // تقليل الاحتكاك
            bus.setTractionMultiplier(0.7f);
            
            // تقليل الرؤية
            camera.setVisibilityRange(0.5f);
        }
    }
}
```

### 2. تحسين الأداء
```java
// استخدام Object Pooling
public class ObjectPool<T> {
    private Queue<T> pool = new LinkedList<>();
    private Supplier<T> factory;
    
    public T acquire() {
        return pool.isEmpty() ? factory.get() : pool.poll();
    }
    
    public void release(T object) {
        // إعادة تعيين الكائن
        if (object instanceof Poolable) {
            ((Poolable) object).reset();
        }
        pool.offer(object);
    }
}

// تطبيق على الركاب
private ObjectPool<Passenger> passengerPool = 
    new ObjectPool<>(() -> new Passenger());
```

### 3. إضافة أصوات جديدة
```java
// إضافة صوت جديد
public void addNewSound(String soundName, String fileName) {
    try {
        int soundId = soundPool.load(context.getAssets()
            .openFd("sounds/" + fileName), 1);
        soundIds.put(soundName, soundId);
    } catch (IOException e) {
        Log.e("SoundManager", "Failed to load sound: " + fileName);
    }
}

// تشغيل الصوت الجديد
public void playNewSound(String soundName, float volume) {
    Integer soundId = soundIds.get(soundName);
    if (soundId != null) {
        soundPool.play(soundId, volume, volume, 1, 0, 1.0f);
    }
}
```

## نصائح للتحسين:

### 1. تحسين الرسم
```java
// تجميع العمليات المتشابهة
private void drawOptimized(Canvas canvas) {
    // رسم الخلفية مرة واحدة
    drawBackground(canvas);
    
    // تجميع رسم المركبات
    drawVehiclesBatched(canvas);
    
    // رسم الواجهة في النهاية
    drawUI(canvas);
}

// استخدام Dirty Rectangles
private void updateDirtyRegions() {
    for (GameObject obj : gameObjects) {
        if (obj.hasChanged()) {
            dirtyRegions.add(obj.getBounds());
        }
    }
}
```

### 2. إدارة الذاكرة
```java
// تنظيف الموارد
@Override
protected void onDestroy() {
    super.onDestroy();
    
    // تحرير الأصوات
    soundManager.release();
    
    // تنظيف الكائنات
    gameEngine.cleanup();
    
    // إيقاف الخيوط
    gameThread.stopThread();
}

// مراقبة استهلاك الذاكرة
private void monitorMemoryUsage() {
    Runtime runtime = Runtime.getRuntime();
    long usedMemory = runtime.totalMemory() - runtime.freeMemory();
    
    if (usedMemory > MEMORY_THRESHOLD) {
        // تنظيف الذاكرة
        performGarbageCollection();
    }
}
```

### 3. التعامل مع الأخطاء
```java
// معالجة الأخطاء بشكل صحيح
public void safeOperation() {
    try {
        // العملية المحتملة للخطأ
        performRiskyOperation();
    } catch (Exception e) {
        // تسجيل الخطأ
        Log.e("GameEngine", "Error in operation", e);
        
        // التعافي من الخطأ
        recoverFromError();
        
        // إشعار المستخدم إذا لزم الأمر
        notifyUserIfNecessary(e);
    }
}
```

## اختبار اللعبة:

### 1. اختبارات الوحدة
```java
@Test
public void testBusPhysics() {
    Bus bus = new Bus();
    bus.setThrottle(1.0f);
    
    float initialSpeed = bus.getSpeed();
    bus.updatePhysics(1.0f); // ثانية واحدة
    
    assertTrue("Bus should accelerate", bus.getSpeed() > initialSpeed);
}

@Test
public void testPassengerBoarding() {
    PassengerManager manager = new PassengerManager(busStops);
    int initialCount = manager.getOnBusPassengersCount();
    
    // محاكاة صعود الركاب
    manager.boardPassengers(waitingPassengers, 40);
    
    assertTrue("Passengers should board", 
               manager.getOnBusPassengersCount() > initialCount);
}
```

### 2. اختبارات الأداء
```java
@Test
public void testFrameRate() {
    GameEngine engine = new GameEngine(context);
    long startTime = System.currentTimeMillis();
    int frameCount = 0;
    
    while (frameCount < 60) { // 60 إطار
        engine.update(0.016f); // 60 FPS
        engine.draw(mockCanvas);
        frameCount++;
    }
    
    long duration = System.currentTimeMillis() - startTime;
    assertTrue("Should maintain 60 FPS", duration < 1100); // مع هامش خطأ
}
```

## الخلاصة:

هذا الدليل يوفر نظرة شاملة على بنية اللعبة وكيفية تطويرها وتحسينها. استخدم هذه المعلومات كنقطة انطلاق لإضافة ميزات جديدة أو تحسين الأداء.

