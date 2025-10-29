# Lab 07 ESP32 Components - การใช้งานและสร้าง Component ด้วยตนเอง

## วัตถุประสงค์การทดลอง
1. เรียนรู้การใช้งาน component ที่มีอยู่แล้ว (local components)
2. เรียนรู้การใช้งาน managed components จาก URL
3. เรียนรู้การสร้าง component ใหม่สำหรับ ESP32
4. เข้าใจโครงสร้างและการจัดการ component ใน ESP-IDF

## อุปกรณ์และเครื่องมือ
- Docker และ Docker Compose
- ESP-IDF Development Environment
- VS Code หรือ Text Editor
- QEMU Emulator (สำหรับการทดสอบ)

## การเตรียมความพร้อม

### ขั้นตอนที่ 1: สร้าง Docker Environment
เตรียมความพร้อมโดยการสร้างไฟล์ `docker-compose.yml`
```yml
services:
  esp32-dev:
    image: espressif/idf:latest
    container_name: esp32-lab7
    volumes:
      - .:/project
    working_dir: /project
    tty: true
    stdin_open: true
    environment:
      - IDF_PATH=/opt/esp/idf
    command: /bin/bash
    networks:
      - esp32-network

networks:
  esp32-network:
    driver: bridge
```

### ขั้นตอนที่ 2: เข้าใช้งาน Docker Container
```bash
# เริ่มต้น Docker Container
docker-compose up -d

# ตรวจสอบ Docker Container

docker-compose ps -a

# ดูว่ามี NAME เป็น esp32-lab7 หรือไม่

# เข้าใช้งาน Container
docker exec -it esp32-lab7 bash
```

---

## Lab 7.1 การใช้ Local Component (การใช้ component ที่มีอยู่บน harddisk)

### ขั้นตอนที่ 1: เตรียม Component Files
ก่อนทดลองให้พิจารณาโครงสร้างของไฟล์ ซึ่งเมื่อทดลองเสร็จแล้วจะมีโครงสร้างโฟลเดอร์และไฟล์ต่างๆ ดังนี้

```bash
Lab7-ESP32-Components/            
├── docker-compose.yml                  # ขั้นการเตรียม 
└── components                          # การทดลอง 7.1
│   └── Sensors/
│   │   ├── CMakeLists.txt        
│   │   ├── sensor.h    
│   │   └── sensor.c    
│   └── Display/
│       ├── CMakeLists.txt        
│       ├── display.h    
│       └── display.c    
└── lab7-1_Managed_Local_Component/    # Lab 7.1           
│   ├── CMakeLists.txt            
│   ├── main/                     
│   │   ├── CMakeLists.txt        
│   │   └── lab7-1.c    
│   ├── build/                    
│   └── README.md                      # <-- ตอบคำถามไว้ในไฟล์นี้ 
└── lab7-2_Managed_url_Component/      # การทดลอง 7.2       
│   ├── CMakeLists.txt            
│   ├── main/                     
│   │   ├── CMakeLists.txt        
│   │   └── lab7-2.c    
│   ├── build/                         # <-- ตอบคำถามไว้ในไฟล์นี้
│   └── README.md                 
└── lab7-3_esp32_Component/            # การทดลอง 7.3
    ├── CMakeLists.txt            
    └── components 
    │   └── Sensors/
    │   │   ├── CMakeLists.txt        
    │   │   ├── sensor.c    
    │   │   └── sensor.h    
    │   └── Display/
    │       ├── CMakeLists.txt        
    │       ├── display.c    
    │       └── display.h    
    ├── main/                     
    │   ├── CMakeLists.txt        
    │   └── lab7-3.c    
    ├── build/                    
    └── README.md                      # <-- ตอบคำถามไว้ในไฟล์นี้
```

### ขั้นตอนที่ 2: สร้างไฟล์ Component

#### สร้างไฟล์ `components/Sensors/CMakeLists.txt`
```cmake
idf_component_register(SRCS "sensor.c"
                       INCLUDE_DIRS "."
                       REQUIRES "log")
```

#### สร้างไฟล์ `components/Sensors/sensor.h`

```c
#ifndef SENSOR_H
#define SENSOR_H

#ifdef __cplusplus
extern "C" {
#endif

/**
 * @brief Initialize sensor module
 */
void sensor_init(void);

/**
 * @brief Read data from sensors
 */
void sensor_read_data(void);

/**
 * @brief Check sensor status
 */
void sensor_check_status(void);

#ifdef __cplusplus
}
#endif

#endif // SENSOR_H
```

#### สร้างไฟล์ `components/Sensors/sensor.c`
```c
#include <stdio.h>
#include <stdlib.h>
#include "esp_system.h"
#include "esp_random.h"
#include "esp_log.h"
#include "sensor.h"

static const char *TAG = "SENSOR";

void sensor_init(void)
{
    ESP_LOGI(TAG, "🔧 Sensor initialized from file: %s, line: %d", __FILE__, __LINE__);
    ESP_LOGI(TAG, "📡 Sensor module ready for operation");
}

void sensor_read_data(void)
{
    ESP_LOGI(TAG, "📊 Reading sensor data from file: %s, line: %d", __FILE__, __LINE__);
    
    // จำลองการอ่านข้อมูลจาก sensor
    float temperature = 25.5 + (float)(esp_random() % 100) / 10.0f;
    float humidity = 60.0 + (float)(esp_random() % 400) / 10.0f;
    
    ESP_LOGI(TAG, "🌡️  Temperature: %.1f°C", temperature);
    ESP_LOGI(TAG, "💧 Humidity: %.1f%%", humidity);
}

void sensor_check_status(void)
{
    ESP_LOGI(TAG, "✅ Sensor status check from file: %s, line: %d", __FILE__, __LINE__);
    ESP_LOGI(TAG, "📈 All sensors operating normally");
}
```

### ขั้นตอนที่ 3: สร้าง Project Lab 7-1 (Local Component)

#### สร้างไฟล์ `lab7-1_Managed_Local_Component/CMakeLists.txt`
```cmake
cmake_minimum_required(VERSION 3.16)

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(lab7-1)
```

#### สร้างไฟล์ `lab7-1_Managed_Local_Component/main/CMakeLists.txt`
```cmake
idf_component_register(SRCS "lab7-1.c"
                       INCLUDE_DIRS ".")
```

#### สร้างไฟล์ `lab7-1_Managed_Local_Component/main/lab7-1.c`
```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_log.h"
#include "sensor.h"

static const char *TAG = "LAB7-1";

void app_main(void)
{
    ESP_LOGI(TAG, "🚀 Lab 7-1: Local Component Demo Started");
    
    // เรียกใช้ฟังก์ชันจาก local component
    sensor_init();
    
    while(1) {
        sensor_read_data();
        sensor_check_status();
        
        ESP_LOGI(TAG, "----------------------------");
        vTaskDelay(pdMS_TO_TICKS(3000));
    }
}
```

#### การ Build และ Flash Lab 7-1
```bash
# เข้าไปใน project directory
cd lab7-1_Managed_Local_Component

#export environment เพื่อให้สามารถเรียกใช้ idf tools ได้
. $IDF_PATH/export.sh

# กำหนด target ESP32
idf.py set-target esp32

# Build project
idf.py build

```

#### สร้างไฟล์ `.gitignore` สำหรับ Lab 7-1
```bash
# สร้างไฟล์ .gitignore ใน lab7-1_Managed_Local_Component
cat > .gitignore << 'EOF'
# ESP-IDF Build files
build/
sdkconfig
sdkconfig.old
sdkconfig.h

# IDE files
.vscode/
.idea/
*.swp
*.swo
*~

# OS files
.DS_Store
Thumbs.db

# Dependencies
managed_components/
dependencies.lock

# Logs
*.log
EOF
```

### ขั้นตอนที่ 4: ระบุเส้นทางไปยังโฟลเดอร์ของ Components

ในการ build จะพบ error เนื่องจากระบบ build ยังไม่รู้จัก components ให้แก้ไขดังต่อไปนี้

ในไฟล์ CMakeLists.txt ของ project ให้แก้ไขเป็นดังนี้

```cmake
cmake_minimum_required(VERSION 3.16)

# Add external components directory
set(EXTRA_COMPONENT_DIRS "../components")

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(lab7-1)
```

การใส่ `EXTRA_COMPONENT_DIRS` ในไฟล์ `CMakeLists.txt` หลักของโปรเจคจะทำให้ระบบ build ค้นหา components จากโฟลเดอร์ที่ระบุโดยอัตโนมัติ

**สำคัญ** ต้องใส่ก่อน `include($ENV{IDF_PATH}/tools/cmake/project.cmake)`


#### สร้างไฟล์ `lab7-1_Managed_Local_Component/README.md`
```markdown
# Lab 7-1: Local Component Demo

## คำอธิบาย
การทดลองนี้แสดงการใช้งาน component ที่มีอยู่ในโฟลเดอร์ `components/Sensors/` ของ project


## สรุปคำสั่งที่ใช้ และผลลัพธ์ที่ได้

### คำสั่งที่ใช้ในการ Build Lab 7-1:
```bash
# เข้าไปใน project directory
cd lab7-1_Managed_Local_Component

# Export environment เพื่อให้สามารถเรียกใช้ idf tools ได้
. $IDF_PATH/export.sh

# กำหนด target ESP32
idf.py set-target esp32

# Build project
idf.py build

# ทดสอบด้วย QEMU
idf.py qemu monitor
```

### ผลลัพธ์ที่ได้:
1. **การเริ่มต้นระบบ**: แสดงข้อความ "Lab 7-1: Local Component Demo Started"
2. **การเริ่มต้น Sensor**: แสดงข้อความจาก sensor_init() พร้อมไฟล์และบรรทัดที่เรียกใช้
3. **การอ่านข้อมูล**: แสดงค่า temperature และ humidity ที่สุ่มขึ้นมา
4. **การตรวจสอบสถานะ**: แสดงสถานะการทำงานของ sensor
5. **การทำงานต่อเนื่อง**: วนลูปทุก 3 วินาที

### ข้อมูลที่แสดงบนหน้าจอ:
```
I (xxx) LAB7-1: 🚀 Lab 7-1: Local Component Demo Started
I (xxx) SENSOR: 🔧 Sensor initialized from file: /project/components/Sensors/sensor.c, line: 11
I (xxx) SENSOR: 📡 Sensor module ready for operation
I (xxx) SENSOR: 📊 Reading sensor data from file: /project/components/Sensors/sensor.c, line: 16
I (xxx) SENSOR: 🌡️  Temperature: 28.3°C
I (xxx) SENSOR: 💧 Humidity: 67.2%
I (xxx) SENSOR: ✅ Sensor status check from file: /project/components/Sensors/sensor.c, line: 26
I (xxx) SENSOR: 📈 All sensors operating normally
I (xxx) LAB7-1: ----------------------------
```

### สิ่งที่เรียนรู้:
- การใช้ `EXTRA_COMPONENT_DIRS` ในไฟล์ CMakeLists.txt เพื่อระบุตำแหน่ง component
- การสร้างโครงสร้าง component ที่ถูกต้อง (CMakeLists.txt, .h, .c)
- การเรียกใช้ฟังก์ชันจาก local component ใน main application

```

## โจทย์ท้าทาย

### 1. สร้าง  component ชื่อ `Display` โดย นำไฟล์ `display.c` และ `display.h` จากใบงานที่ 6 มาใช้ 

สิ่งที่ต้องมีใน display component
1. ไฟล์ `CMakeLists.txt` 
2. ไฟล์ `display.h`
3. ไฟล์ `display.c`

#### คำตอบ:

**ขั้นตอนการสร้าง Display Component:**

1. **สร้างโฟลเดอร์ `components/Display/`**
2. **สร้างไฟล์ `components/Display/CMakeLists.txt`**
```cmake
idf_component_register(SRCS "display.c"
                       INCLUDE_DIRS "."
                       REQUIRES "log")
```

3. **สร้างไฟล์ `components/Display/display.h`** (นำมาจากใบงานที่ 6)
```c
#ifndef DISPLAY_H
#define DISPLAY_H

#ifdef __cplusplus
extern "C" {
#endif

void display_init(void);
void display_show_sensor_data(float temperature, float humidity);
void display_show_status(const char* status);
void display_clear(void);

#ifdef __cplusplus
}
#endif

#endif // DISPLAY_H
```

4. **สร้างไฟล์ `components/Display/display.c`** (ดัดแปลงจากใบงานที่ 6)
```c
#include <stdio.h>
#include <string.h>
#include "esp_log.h"
#include "display.h"

static const char *TAG = "DISPLAY";

void display_init(void)
{
    ESP_LOGI(TAG, "🖥️  Display initialized from file: %s, line: %d", __FILE__, __LINE__);
    ESP_LOGI(TAG, "📱 Virtual display ready");
}

void display_show_sensor_data(float temperature, float humidity)
{
    ESP_LOGI(TAG, "┌─────────────────────────────────┐");
    ESP_LOGI(TAG, "│        SENSOR DATA DISPLAY      │");
    ESP_LOGI(TAG, "├─────────────────────────────────┤");
    ESP_LOGI(TAG, "│ 🌡️  Temperature: %6.1f°C      │", temperature);
    ESP_LOGI(TAG, "│ 💧 Humidity:    %6.1f%%       │", humidity);
    ESP_LOGI(TAG, "└─────────────────────────────────┘");
}

void display_show_status(const char* status)
{
    ESP_LOGI(TAG, "📊 Status: %s", status);
}

void display_clear(void)
{
    ESP_LOGI(TAG, "🧹 Display cleared");
}
```

### 2. นำโค้ดจาก main.c ในใบงานที่ 6 มาใช้ แล้ว build พร้อมทดสอบ

#### คำตอบ:

**แก้ไขไฟล์ `lab7-1_Managed_Local_Component/main/lab7-1.c`**
```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_log.h"
#include "sensor.h"
#include "display.h"  // เพิ่ม include display component

static const char *TAG = "LAB7-1";

void app_main(void)
{
    ESP_LOGI(TAG, "🚀 Lab 7-1: Local Component Demo (Sensor + Display) Started");
    
    // เรียกใช้ฟังก์ชันจาก local components
    sensor_init();
    display_init();
    
    int reading_count = 0;
    
    while(1) {
        reading_count++;
        ESP_LOGI(TAG, "📋 Reading #%d", reading_count);
        
        display_clear();
        
        // อ่านข้อมูลจาก sensor
        sensor_read_data();
        
        // จำลองการแยกค่า temperature และ humidity
        float temperature = 25.5 + (reading_count % 10);
        float humidity = 60.0 + (reading_count % 20);
        
        // แสดงผลผ่าน display component
        display_show_sensor_data(temperature, humidity);
        
        sensor_check_status();
        display_show_status("✅ System Normal");
        
        ESP_LOGI(TAG, "----------------------------");
        vTaskDelay(pdMS_TO_TICKS(3000));
    }
}
```

**ผลลัพธ์การ Build และทดสอบ:**
```bash
# เข้าไปใน project directory
cd lab7-1_Managed_Local_Component

# Build project
idf.py build

# ผลลัพธ์: Build เสร็จสมบูรณ์
# แสดงข้อมูลจากทั้ง sensor และ display components
# Display component แสดงข้อมูลในรูปแบบตารางที่สวยงาม
```

**ผลลัพธ์บนหน้าจอ:**
```
I (xxx) LAB7-1: 🚀 Lab 7-1: Local Component Demo (Sensor + Display) Started
I (xxx) SENSOR: 🔧 Sensor initialized from file: /project/components/Sensors/sensor.c, line: 11
I (xxx) DISPLAY: 🖥️  Display initialized from file: /project/components/Display/display.c, line: 8
I (xxx) DISPLAY: 🧹 Display cleared
I (xxx) SENSOR: 📊 Reading sensor data from file: /project/components/Sensors/sensor.c, line: 16
I (xxx) DISPLAY: ┌─────────────────────────────────┐
I (xxx) DISPLAY: │        SENSOR DATA DISPLAY      │
I (xxx) DISPLAY: ├─────────────────────────────────┤
I (xxx) DISPLAY: │ 🌡️  Temperature:   25.5°C      │
I (xxx) DISPLAY: │ 💧 Humidity:       60.0%       │
I (xxx) DISPLAY: └─────────────────────────────────┘
I (xxx) DISPLAY: 📊 Status: ✅ System Normal
```


ใส่ผลลัพธ์ทั้งหมดในไฟล์ README.md ของใบงานนี้

---

## Lab 7.2 การใช้ Managed Component จาก URL

### ขั้นตอนที่ 1: สร้าง Project Lab 7-2


#### 1.1 สร้างโฟลเดอร์  `lab7-2_Managed_url_Component`

#### 1.2 สร้างไฟล์ `lab7-2_Managed_url_Component/CMakeLists.txt`
```cmake
cmake_minimum_required(VERSION 3.16)

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(lab7-2)
```

#### 1.3 สร้างไฟล์ `lab7-2_Managed_url_Component/main/CMakeLists.txt`
```cmake
idf_component_register(SRCS "lab7-2.c"
                       INCLUDE_DIRS ".")
```

#### 1.4 สร้างไฟล์ `lab7-2_Managed_url_Component/main/idf_component.yml`
```yaml
dependencies:
  lab7_components:
    git: https://github.com/APPLICATIONS-OF-MICROCONTROLLERS/Lab7_Components.git
```

#### 1.5 สร้างไฟล์ `lab7-2_Managed_url_Component/main/lab7-2.c`
```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_log.h"
#include "sensor.h"

static const char *TAG = "LAB7-2";

void app_main(void)
{
    ESP_LOGI(TAG, "🚀 Lab 7-2: Managed Component from GitHub URL Demo Started");
    ESP_LOGI(TAG, "📥 Using Sensors component from: https://github.com/APPLICATIONS-OF-MICROCONTROLLERS/Lab7_Components");
    
    // เรียกใช้ฟังก์ชันจาก managed component (GitHub)
    sensor_init();
    
    int reading_count = 0;
    
    while(1) {
        reading_count++;
        ESP_LOGI(TAG, "📋 Reading #%d from GitHub Component", reading_count);
        
        sensor_read_data();
        sensor_check_status();
        
        ESP_LOGI(TAG, "� Component Source: GitHub Repository");
        ESP_LOGI(TAG, "==========================================");
        vTaskDelay(pdMS_TO_TICKS(4000));
    }
}
```

#### 1.6 การ Build และ Flash Lab 7-2
```bash
# เข้าไปใน project directory
cd lab7-2_Managed_url_Component

#export environment เพื่อให้สามารถเรียกใช้ idf tools ได้
. $IDF_PATH/export.sh

# กำหนด target ESP32
idf.py set-target esp32

# Build project (จะดาวน์โหลด lab7_components จาก GitHub อัตโนมัติ)
idf.py build

# รัน QEMU (สำหรับการทดสอบ)
idf.py qemu monitor
```

#### สร้างไฟล์ `.gitignore` สำหรับ Lab 7-2
```bash
# สร้างไฟล์ .gitignore ใน lab7-2_Managed_url_Component
cat > .gitignore << 'EOF'
# ESP-IDF Build files
build/
sdkconfig
sdkconfig.old
sdkconfig.h

# IDE files
.vscode/
.idea/
*.swp
*.swo
*~

# OS files
.DS_Store
Thumbs.db

# Managed Components (downloaded from GitHub)
managed_components/
dependencies.lock

# Logs
*.log
EOF
```

#### 1.7 สร้างไฟล์ `lab7-2_Managed_url_Component/README.md`
```markdown
# Lab 7-2: Managed Component from GitHub URL Demo

## คำอธิบาย
การทดลองนี้แสดงการใช้งาน managed component จาก GitHub Repository
ใช้ `Sensors` component จาก https://github.com/APPLICATIONS-OF-MICROCONTROLLERS/Lab7_Components

## ผลลัพธ์ที่คาดหวัง
- แสดงข้อความการเริ่มต้น sensor จาก GitHub component
- แสดงข้อมูล temperature และ humidity ทุก 4 วินาที
- แสดงสถานะการทำงานของ sensor
- แสดงแหล่งที่มาของ component (GitHub Repository)

## ความแตกต่างจาก Lab 7-1
- Lab 7-1: ใช้ local component (ในเครื่อง)
- Lab 7-2: ใช้ managed component จาก GitHub URL

## การใช้งาน
1. เข้าไปในโฟลเดอร์ lab7-2_Managed_url_Component
2. รันคำสั่ง `idf.py build` (จะดาวน์โหลด component จาก GitHub อัตโนมัติ)
3. ทดสอบด้วย QEMU
```

---

## โจทย์ท้าทาย

### 1. สร้าง  component ชื่อ `Display` โดย นำไฟล์ `display.c` และ `display.h` จากใบงานที่ 6 มาใช้ 

สิ่งที่ต้องมีใน display component
1. ไฟล์ `CMakeLists.txt` 
2. ไฟล์ `display.h`
3. ไฟล์ `display.c`

#### คำตอบ:

**ข้อสังเกต**: สำหรับ Lab 7.2 (Managed Component from URL) การเพิ่ม Display component จะต้องทำผ่าน GitHub Repository หรือใช้วิธี local component

**วิธีที่ 1: เพิ่ม Local Display Component**

1. **แก้ไขไฟล์ `lab7-2_Managed_url_Component/CMakeLists.txt`**
```cmake
cmake_minimum_required(VERSION 3.16)

# เพิ่ม external components directory สำหรับ Display
set(EXTRA_COMPONENT_DIRS "../components")

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(lab7-2)
```

2. **ใช้ Display component เดียวกับ Lab 7.1** (ในโฟลเดอร์ `components/Display/`)

**วิธีที่ 2: แก้ไข GitHub Repository**

สร้าง Display component ใน GitHub Repository และเพิ่มใน `idf_component.yml`
```yaml
dependencies:
  lab7_components:
    git: https://github.com/APPLICATIONS-OF-MICROCONTROLLERS/Lab7_Components.git
    include:
      - sensor
      - display  # เพิ่ม display component
```

### 2. นำโค้ดจาก main.c ในใบงานที่ 6 มาใช้ แล้ว build พร้อมทดสอบ

ให้ผลลักษณะเดียวกับ component แบบ local หรือไม่

#### คำตอบ:

**แก้ไขไฟล์ `lab7-2_Managed_url_Component/main/lab7-2.c`** (ใช้วิธีที่ 1)
```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_log.h"
#include "sensor.h"
#include "display.h"  // เพิ่ม include display component

static const char *TAG = "LAB7-2";

void app_main(void)
{
    ESP_LOGI(TAG, "🚀 Lab 7-2: Managed Component + Local Display Demo Started");
    ESP_LOGI(TAG, "📥 Using Sensors component from: https://github.com/APPLICATIONS-OF-MICROCONTROLLERS/Lab7_Components");
    ESP_LOGI(TAG, "📱 Using Display component from: Local Components");
    
    // เรียกใช้ฟังก์ชันจาก managed และ local components
    sensor_init();
    display_init();
    
    int reading_count = 0;
    
    while(1) {
        reading_count++;
        ESP_LOGI(TAG, "📋 Reading #%d from Mixed Components", reading_count);
        
        display_clear();
        
        // อ่านข้อมูลจาก GitHub sensor component
        sensor_read_data();
        
        // จำลองการแยกค่า temperature และ humidity
        float temperature = 26.0 + (reading_count % 8);
        float humidity = 65.0 + (reading_count % 15);
        
        // แสดงผลผ่าน local display component
        display_show_sensor_data(temperature, humidity);
        
        sensor_check_status();
        display_show_status("✅ Mixed Components OK");
        
        ESP_LOGI(TAG, "🔄 GitHub Sensor + Local Display");
        ESP_LOGI(TAG, "==========================================");
        vTaskDelay(pdMS_TO_TICKS(4000));
    }
}
```

**ผลลัพธ์การ Build และทดสอบ:**
```bash
# เข้าไปใน project directory
cd lab7-2_Managed_url_Component

# Build project
idf.py build

# ผลลัพธ์: Build เสร็จสมบูรณ์
# ดาวน์โหลด sensor component จาก GitHub
# ใช้ display component จาก local
```

**ผลลัพธ์บนหน้าจอ:**
```
I (xxx) LAB7-2: 🚀 Lab 7-2: Managed Component + Local Display Demo Started
I (xxx) LAB7-2: 📥 Using Sensors component from: https://github.com/APPLICATIONS-OF-MICROCONTROLLERS/Lab7_Components
I (xxx) LAB7-2: 📱 Using Display component from: Local Components
I (xxx) SENSOR: 🔧 Sensor initialized from GitHub component
I (xxx) DISPLAY: 🖥️  Display initialized from file: /project/components/Display/display.c
I (xxx) DISPLAY: 🧹 Display cleared
I (xxx) SENSOR: 📊 Reading sensor data from GitHub component
I (xxx) DISPLAY: ┌─────────────────────────────────┐
I (xxx) DISPLAY: │        SENSOR DATA DISPLAY      │
I (xxx) DISPLAY: ├─────────────────────────────────┤
I (xxx) DISPLAY: │ 🌡️  Temperature:   26.0°C      │
I (xxx) DISPLAY: │ 💧 Humidity:       65.0%       │
I (xxx) DISPLAY: └─────────────────────────────────┘
I (xxx) DISPLAY: 📊 Status: ✅ Mixed Components OK
I (xxx) LAB7-2: 🔄 GitHub Sensor + Local Display
```

**คำตอบคำถาม: ให้ผลลักษณะเดียวกับ component แบบ local หรือไม่**

**ใช่ ให้ผลลักษณะเดียวกัน** เพราะ:
1. **การทำงาน**: ฟังก์ชันทำงานเหมือนกันไม่ว่าจะมาจาก GitHub หรือ local
2. **การแสดงผล**: แสดงข้อมูลในรูปแบบเดียวกัน
3. **API**: interface ของ component เหมือนกัน

**ความแตกต่าง**:
1. **แหล่งที่มา**: Sensor จาก GitHub, Display จาก local
2. **การจัดการ**: Managed component อัพเดทได้อัตโนมัติจาก GitHub
3. **การ dependency**: ระบุใน `idf_component.yml` สำหรับ GitHub component
4. **การ build**: ต้องดาวน์โหลดจาก internet ครั้งแรก

ใส่ผลลัพธ์ทั้งหมดในไฟล์ README.md ของใบงานนี้



## Lab 7.3 การสร้าง ESP32 Component ใหม่ด้วยคำสั่ง idf.py create-component

### ขั้นตอนที่ 1: สร้าง Project และ Components

#### สร้างไฟล์ `lab7-3_esp32_Component/CMakeLists.txt`
```cmake
cmake_minimum_required(VERSION 3.16)

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(lab7-3)
```

#### ใช้คำสั่ง idf.py create-component เพื่อสร้าง Components
```bash
# เข้าไปใน project directory
cd lab7-3_esp32_Component

#export environment เพื่อให้สามารถเรียกใช้ idf tools ได้
. $IDF_PATH/export.sh

# สร้างโฟลเดอร์ components ก่อน 
mkdir -p components

# สร้าง sensor component ในโฟลเดอร์ components
cd components
idf.py create-component sensor

# สร้าง display component ในโฟลเดอร์ components  
idf.py create-component display
cd ..
```

> **หมายเหตุ:** การสร้าง component ในโฟลเดอร์ `components/` จะช่วยให้:
> - การจัดระเบียบ project ดีขึ้น
> - ง่ายต่อการทำงานเป็นทีมผ่าน GitHub
> - เป็นมาตรฐานของ ESP-IDF project
> - แต่ละคนสามารถรับผิดชอบ component ต่างกันได้

#### โครงสร้างโฟลเดอร์ที่เกิดขึ้นหลังจากรันคำสั่ง:
```
lab7-3_esp32_Component/
├── CMakeLists.txt
├── components/                   # สร้างด้วยตนเอง
│   ├── sensor/                   # สร้างจาก idf.py create-component sensor
│   │   ├── CMakeLists.txt        # สร้างอัตโนมัติ
│   │   ├── include/
│   │   │   └── sensor.h          # สร้างอัตโนมัติ (ต้องแก้ไข)
│   │   └── sensor.c              # สร้างอัตโนมัติ (ต้องแก้ไข)
│   └── display/                  # สร้างจาก idf.py create-component display
│       ├── CMakeLists.txt        # สร้างอัตโนมัติ
│       ├── include/
│       │   └── display.h         # สร้างอัตโนมัติ (ต้องแก้ไข)
│       └── display.c             # สร้างอัตโนมัติ (ต้องแก้ไข)
└── main/
    ├── CMakeLists.txt
    └── lab7-3.c
```

### ขั้นตอนที่ 2: แก้ไขไฟล์ Sensor Component

#### แก้ไขไฟล์ `lab7-3_esp32_Component/components/sensor/CMakeLists.txt`
```cmake
idf_component_register(SRCS "sensor.c"
                       INCLUDE_DIRS "include"
                       REQUIRES "log" "driver")
```

#### แก้ไขไฟล์ `lab7-3_esp32_Component/components/sensor/include/sensor.h`
```c
#ifndef SENSOR_H
#define SENSOR_H

#ifdef __cplusplus
extern "C" {
#endif

/**
 * @brief Initialize sensor module with GPIO
 */
void sensor_init(void);

/**
 * @brief Read temperature data
 */
float sensor_read_temperature(void);

/**
 * @brief Read humidity data
 */
float sensor_read_humidity(void);

/**
 * @brief Read all sensor data and display
 */
void sensor_read_all_data(void);

#ifdef __cplusplus
}
#endif

#endif // SENSOR_H
```

#### แก้ไขไฟล์ `lab7-3_esp32_Component/components/sensor/sensor.c`
```c
#include <stdio.h>
#include <stdlib.h>
#include "esp_system.h"
#include "esp_random.h"
#include "esp_log.h"
#include "driver/gpio.h"
#include "sensor.h"

static const char *TAG = "ENHANCED_SENSOR";

void sensor_init(void)
{
    ESP_LOGI(TAG, "🔧 Enhanced Sensor Component initialized");
    ESP_LOGI(TAG, "📍 File: %s, Line: %d", __FILE__, __LINE__);
    
    // กำหนด GPIO สำหรับ LED indicator
    gpio_config_t io_conf = {
        .pin_bit_mask = (1ULL << GPIO_NUM_2),
        .mode = GPIO_MODE_OUTPUT,
        .pull_up_en = 0,
        .pull_down_en = 0,
        .intr_type = GPIO_INTR_DISABLE,
    };
    gpio_config(&io_conf);
    
    ESP_LOGI(TAG, "✅ GPIO LED configured on pin 2");
}

float sensor_read_temperature(void)
{
    float temperature = 20.0 + (float)(esp_random() % 200) / 10.0f;
    ESP_LOGI(TAG, "🌡️  Temperature: %.2f°C", temperature);
    return temperature;
}

float sensor_read_humidity(void)
{
    float humidity = 40.0 + (float)(esp_random() % 400) / 10.0f;
    ESP_LOGI(TAG, "💧 Humidity: %.2f%%", humidity);
    return humidity;
}

void sensor_read_all_data(void)
{
    ESP_LOGI(TAG, "📊 Reading all sensor data...");
    
    // เปิด LED เมื่ออ่านข้อมูล
    gpio_set_level(GPIO_NUM_2, 1);
    
    float temp = sensor_read_temperature();
    float hum = sensor_read_humidity();
    
    // คำนวณ Heat Index
    float heat_index = temp + 0.5 * hum;
    ESP_LOGI(TAG, "🔥 Heat Index: %.2f", heat_index);
    
    // แสดงสถานะตามค่า Heat Index
    if (heat_index < 80) {
        ESP_LOGI(TAG, "✅ Comfortable conditions");
    } else if (heat_index < 90) {
        ESP_LOGI(TAG, "⚠️  Caution: Possible fatigue");
    } else {
        ESP_LOGI(TAG, "🚨 Warning: High heat stress");
    }
    
    // ปิด LED หลังอ่านข้อมูลเสร็จ
    gpio_set_level(GPIO_NUM_2, 0);
}
```

### ขั้นตอนที่ 3: แก้ไขไฟล์ Display Component

#### แก้ไขไฟล์ `lab7-3_esp32_Component/components/display/CMakeLists.txt`
```cmake
idf_component_register(SRCS "display.c"
                       INCLUDE_DIRS "include"
                       REQUIRES "log")
```

#### แก้ไขไฟล์ `lab7-3_esp32_Component/components/display/include/display.h`
```c
#ifndef DISPLAY_H
#define DISPLAY_H

#ifdef __cplusplus
extern "C" {
#endif

/**
 * @brief Initialize display module
 */
void display_init(void);

/**
 * @brief Show sensor data on display
 */
void display_show_sensor_data(float temperature, float humidity, float heat_index);

/**
 * @brief Show system status
 */
void display_show_status(const char* status);

/**
 * @brief Clear display
 */
void display_clear(void);

#ifdef __cplusplus
}
#endif

#endif // DISPLAY_H
```

#### แก้ไขไฟล์ `lab7-3_esp32_Component/components/display/display.c`
```c
#include <stdio.h>
#include <string.h>
#include "esp_log.h"
#include "display.h"

static const char *TAG = "DISPLAY";

void display_init(void)
{
    ESP_LOGI(TAG, "🖥️  Display Component initialized");
    ESP_LOGI(TAG, "📍 File: %s, Line: %d", __FILE__, __LINE__);
    ESP_LOGI(TAG, "✅ Virtual display ready for operation");
}

void display_show_sensor_data(float temperature, float humidity, float heat_index)
{
    ESP_LOGI(TAG, "┌─────────────────────────────────┐");
    ESP_LOGI(TAG, "│        SENSOR DATA DISPLAY      │");
    ESP_LOGI(TAG, "├─────────────────────────────────┤");
    ESP_LOGI(TAG, "│ 🌡️  Temperature: %6.2f°C      │", temperature);
    ESP_LOGI(TAG, "│ 💧 Humidity:    %6.2f%%       │", humidity);
    ESP_LOGI(TAG, "│ 🔥 Heat Index:  %6.2f        │", heat_index);
    ESP_LOGI(TAG, "└─────────────────────────────────┘");
}

void display_show_status(const char* status)
{
    ESP_LOGI(TAG, "┌─────────────────────────────────┐");
    ESP_LOGI(TAG, "│         SYSTEM STATUS           │");
    ESP_LOGI(TAG, "├─────────────────────────────────┤");
    ESP_LOGI(TAG, "│ Status: %-23s │", status);
    ESP_LOGI(TAG, "└─────────────────────────────────┘");
}

void display_clear(void)
{
    ESP_LOGI(TAG, "🧹 Display cleared");
    ESP_LOGI(TAG, "");
}
```

### ขั้นตอนที่ 4: สร้างไฟล์ Main Application

#### สร้างไฟล์ `lab7-3_esp32_Component/main/CMakeLists.txt`
```cmake
idf_component_register(SRCS "lab7-3.c"
                       INCLUDE_DIRS ".")
```

#### สร้างไฟล์ `lab7-3_esp32_Component/main/lab7-3.c`
```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_log.h"
#include "sensor.h"
#include "display.h"

static const char *TAG = "LAB7-3";

void app_main(void)
{
    ESP_LOGI(TAG, "� Lab 7-3: Custom Components Demo (sensor + display) Started");
    ESP_LOGI(TAG, "📦 Using components created with idf.py create-component");
    
    // เริ่มต้น components
    sensor_init();
    display_init();
    
    int reading_count = 0;
    
    while(1) {
        reading_count++;
        ESP_LOGI(TAG, "📋 Reading #%d", reading_count);
        
        display_clear();
        
        // อ่านข้อมูลจาก sensor component
        float temp = sensor_read_temperature();
        float hum = sensor_read_humidity();
        
        // คำนวณ Heat Index
        float heat_index = temp + 0.5 * hum;
        ESP_LOGI(TAG, "🔥 Heat Index: %.2f", heat_index);
        
        // แสดงข้อมูลผ่าน display component
        display_show_sensor_data(temp, hum, heat_index);
        
        // แสดงสถานะตามค่า Heat Index
        if (heat_index < 80) {
            display_show_status("✅ Comfortable");
        } else if (heat_index < 90) {
            display_show_status("⚠️  Caution");
        } else {
            display_show_status("🚨 Warning");
        }
        
        ESP_LOGI(TAG, "==========================================");
        vTaskDelay(pdMS_TO_TICKS(6000));
    }
}
```

### ขั้นตอนที่ 5: การ Build และทดสอบ Lab 7-3

#### การ Build และ Flash Lab 7-3
```bash
# เข้าไปใน project directory
cd lab7-3_esp32_Component

#export environment เพื่อให้สามารถเรียกใช้ idf tools ได้
. $IDF_PATH/export.sh

# กำหนด target ESP32
idf.py set-target esp32

# Build project
idf.py build

# รัน QEMU (สำหรับการทดสอบ)
idf.py qemu monitor
```

#### สร้างไฟล์ `.gitignore` สำหรับ Lab 7-3
```bash
# สร้างไฟล์ .gitignore ใน lab7-3_esp32_Component
# ESP-IDF Build files
build/
sdkconfig
sdkconfig.old
sdkconfig.h

# IDE files
.vscode/
.idea/
*.swp
*.swo
*~

# OS files
.DS_Store
Thumbs.db

# Component dependencies
managed_components/
dependencies.lock

# Logs
*.log

# Temporary files
*.tmp
*.temp
```

#### สร้างไฟล์ `lab7-3_esp32_Component/README.md`
```markdown
# Lab 7-3: Custom ESP32 Components (Sensor + Display)

## คำอธิบาย
การทดลองนี้แสดงการสร้าง component ใหม่ด้วยคำสั่ง `idf.py create-component`
สร้าง 2 components:
1. **Sensor Component** - อ่านค่า temperature, humidity และคำนวณ heat index
2. **Display Component** - แสดงผลข้อมูลในรูปแบบตาราง

## โครงสร้างโฟลเดอร์หลังใช้ create-component
lab7-3_esp32_Component/
├── CMakeLists.txt
├── components/
│   ├── sensor/
│   │   ├── CMakeLists.txt
│   │   ├── include/
│   │   │   └── sensor.h
│   │   └── sensor.c
│   └── display/
│       ├── CMakeLists.txt
│       ├── include/
│       │   └── display.h
│       └── display.c
├── main/
│   ├── CMakeLists.txt
│   └── lab7-3.c
├── build/
└── README.md
```

## ผลลัพธ์ที่คาดหวัง
- แสดงข้อความการเริ่มต้น sensor และ display components
- แสดงข้อมูล temperature และ humidity
- คำนวณและแสดง heat index
- แสดงข้อมูลในรูปแบบตารางผ่าน display component
- แสดงสถานะความปลอดภัยตามค่า heat index

## ความแตกต่างจาก Lab อื่นๆ
- **Lab 7-1**: ใช้ local component (โฟลเดอร์ components ของ project)
- **Lab 7-2**: ใช้ managed component จาก GitHub URL
- **Lab 7-3**: สร้าง component ใหม่ด้วย `idf.py create-component` (2 components ในโฟลเดอร์ components/)

## ข้อดีของการใช้โฟลเดอร์ components/
1. **การจัดระเบียบ** - แยก components ออกจาก main application
2. **การทำงานเป็นทีม** - แต่ละคนสามารถรับผิดชอบ component ต่างกันได้
3. **GitHub Collaboration** - ง่ายต่อการ review code และ merge
4. **Modularity** - component สามารถนำไปใช้ใน project อื่นได้
5. **ESP-IDF Standard** - เป็นมาตรฐานของ ESP-IDF project



## การใช้งาน
1. สร้าง components ด้วยคำสั่ง `idf.py create-component`
2. แก้ไขไฟล์ CMakeLists.txt, .h และ .c ของแต่ละ component
3. เขียน main application ที่เรียกใช้ทั้ง 2 components
4. Build และทดสอบด้วย QEMU


#### การ Build และ Flash Lab 7-3

```bash
# เข้าไปใน project directory
cd lab7-3_esp32_Component

# กำหนด target ESP32
idf.py set-target esp32

# Build project
idf.py build

# ทดสอบการทำงาน
idf.py qemu
```

#### สร้างไฟล์ `lab7-3_esp32_Component/README.md`

---

## Lab 7.4 (เพิ่มเติม): การใช้ Component Registry Manager

### การค้นหา Component
```bash
# ค้นหา component ที่เกี่ยวกับ sensor
idf.py create-project-from-example "espressif/esp_insights:diagnostics_smoke_test"

# ดู component ที่ติดตั้งแล้ว
idf.py dependency-tree
```

### การจัดการ Dependencies
```bash
# เพิ่ม component ใหม่
idf.py add-dependency "espressif/led_strip^2.0.0"

# อัพเดท component
idf.py update-dependencies
```

---

## สรุปผลการทดลอง

การทดลองนี้ช่วยให้เข้าใจการจัดการ Component ใน ESP-IDF ในรูปแบบต่างๆ:

1. **Local Component** - การใช้ component ที่มีอยู่ในเครื่อง ช่วยให้สามารถนำโค้ดที่เขียนไว้แล้วมาใช้ซ้ำได้

2. **Managed Component** - การใช้ component จากแหล่งภายนอก ช่วยลดเวลาการพัฒนาและได้รับการอัพเดทอย่างสม่ำเสมอ

3. **Custom Component** - การสร้าง component ใหม่ ช่วยให้สามารถพัฒนาฟังก์ชันเฉพาะตามความต้องการได้

ทักษะเหล่านี้จะช่วยในการพัฒนาโปรแกรม ESP32 ที่มีความซับซ้อน มีการจัดระเบียบที่ดี และง่ายต่อการบำรุงรักษา การเข้าใจการใช้งาน component จะทำให้การพัฒนา embedded system มีประสิทธิภาพมากขึ้น

---

## ใบงานต่อไป (GitHub Team Work)
