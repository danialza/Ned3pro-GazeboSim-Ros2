# راهنمای کامل: نمایش Niryo Ned3Pro در RViz2 (macOS)

## مقدمات
- **سیستم عامل**: macOS (Apple Silicon یا Intel)
- **محیط**: Conda/RoboStack
- **ROS2 Distribution**: Humble
- **هدف**: دیدن مدل Ned3 Pro به‌صورت 3D در RViz2

---

## مرحله ۱: آماده‌سازی محیط Conda

### 1.1 فعال‌کردن محیط ros2
```bash
conda activate ros2
```

### 1.2 نصب وابستگی‌های لازم
اگر `joint_state_publisher_gui` و `xacro` نصب نیستند:

```bash
mamba install -c robostack -c conda-forge ros-humble-joint-state-publisher-gui
mamba install -c robostack -c conda-forge ros-humble-xacro
```

---

## مرحله ۲: Source کردن ROS2 Environment

### 2.1 رفتن به workspace
```bash
cd ~/ros2_ws/sim2
```

### 2.2 Source کردن ROS2 conda env
```bash
source $CONDA_PREFIX/setup.zsh
# (اگر setup.zsh نداشتی: source $CONDA_PREFIX/local_setup.zsh)
```

### 2.3 Source کردن workspace
```bash
source install/setup.zsh
```

### 2.4 تست: پکیج‌های Niryo باید دیده شوند
```bash
ros2 pkg list | grep -i niryo
```

**خروجی انتظاری:**
```
niryo_ned2_moveit_config
niryo_ned3pro_moveit_config
niryo_ned_description
niryo_ned_ros2_driver
niryo_ned_ros2_interfaces
```

---

## مرحله ۳: اجرای RViz2 با Ned3Pro (سه ترمینال)

### ترمینال 1: joint_state_publisher_gui
```bash
conda activate ros2
cd ~/ros2_ws/sim2
source $CONDA_PREFIX/setup.zsh
source install/setup.zsh

ros2 run joint_state_publisher_gui joint_state_publisher_gui
```
**توضیح**: این برنامه GUI تولید می‌کند برای کنترل هر joint به‌صورت اسلایدر.

---

### ترمینال 2: robot_state_publisher (برای Ned3Pro)
```bash
conda activate ros2
cd ~/ros2_ws/sim2
source $CONDA_PREFIX/setup.zsh
source install/setup.zsh

ros2 run robot_state_publisher robot_state_publisher \
  --ros-args -p robot_description:="$(xacro $(ros2 pkg prefix niryo_ned_description)/share/niryo_ned_description/urdf/ned3pro/niryo_ned3pro.urdf.xacro)"
```
**توضیح**: 
- این node URDF (Unified Robot Description Format) را لود می‌کند
- مدل Ned3Pro با xacro compile می‌شود
- تبدیل‌های TF (Transform) را منتشر می‌کند
- سیگنال‌های `/joint_states` را دریافت می‌کند و TF به‌روز می‌کند

---

### ترمینال 3: RViz2
```bash
conda activate ros2
cd ~/ros2_ws/sim2
source $CONDA_PREFIX/setup.zsh
source install/setup.zsh

rviz2 -d $(ros2 pkg prefix niryo_ned_description)/share/niryo_ned_description/rviz/default.rviz
```
**توضیح**: 
- RViz2 را با config فایل پیش‌فرض Niryo بالا می‌آورد
- RobotModel و Grid و TF نمایش داده می‌شوند

---

## مرحله ۴: تنظیم RViz2

### 4.1 بررسی Fixed Frame
- **Displays** (پنل سمت چپ) → **Global Options** 
- **Fixed Frame** را روی `base_link` بگذار

### 4.2 فعال‌کردن TF (Transform)
- اگر TF دیده نشد: **Add** → جستجو برای **TF** → اضافه کن
- این تمام جوینت‌های ربات را به‌صورت فریم‌های رنگ‌دار نمایش می‌دهد

### 4.3 فعال‌کردن RobotModel
- **RobotModel** باید از قبل فعال باشد
- اگر نیست: **Add** → جستجو برای **RobotModel** → اضافه کن

### 4.4 کنترل Joints
- **joint_state_publisher_gui** window می‌اید (یک پنل جداگانه)
- از اسلایدرها برای تغییر هر joint استفاده کن
- مدل در RViz به‌صورت real-time به‌روز می‌شود

---

## مرحله ۵: مشاهدهٔ مدل

- **zoom کردن**: Scroll mouse
- **حرکت دادن**: Middle mouse drag (یا Command + Mouse drag روی macOS)
- **چرخش**: Right mouse drag

مدل Ned3Pro با رنگ‌های ممکن است قرمز یا ساده دیده شود (به‌دلیل محدودیت OpenGL 2.1 روی macOS).

---

## نوت: Ned2 vs Ned3Pro

اگر سهواً Ned2 را دیدی:
- اطمینان حاصل کن دستور ترمینال 2 از اسم درست استفاده می‌کند:
  ```
  urdf/ned3pro/niryo_ned3pro.urdf.xacro
  ```
  (نه `ned2/niryo_ned2.urdf.xacro`)

---

## خلاصهٔ Workspace Structure

```
~/ros2_ws/sim2/
├── src/
│   ├── niryo_ned_description/         # URDF/Mesh files
│   ├── niryo_ned_ros2_driver/         # Main driver
│   ├── niryo_ned_ros2_interfaces/     # Message definitions
│   ├── niryo_ned3pro_moveit_config/   # MoveIt config (فعلاً بدون MoveIt)
│   └── niryo_ned2_moveit_config/
├── build/                             # Build files
├── install/                           # Compiled packages
└── log/                              # ROS logs
```

---

## مشاکل رایج و حل‌ها

### مشکل: "Package not found"
**حل**: اطمینان حاصل کن تمام ترمینال‌ها از setup.zsh source کرده‌اند:
```bash
source $CONDA_PREFIX/setup.zsh
source install/setup.zsh
```

### مشکل: "xacro: command not found"
**حل**: `ros-humble-xacro` را نصب کن:
```bash
mamba install -c robostack -c conda-forge ros-humble-xacro
```

### مشکل: "joint_state_publisher_gui" window باز نشد
**حل**: `ros-humble-joint-state-publisher-gui` را نصب کن:
```bash
mamba install -c robostack -c conda-forge ros-humble-joint-state-publisher-gui
```

### مشکل: RViz خالی است (مدل دیده نشد)
**حل**:
1. Fixed Frame را بر روی `base_link` بگذار
2. RobotModel و TF را Add کن
3. در terminal 2، دستور robot_state_publisher باید هیچ خطایی دهد ندهد

### مشکل: رنگ‌های نادرست (همه قرمز)
**دلیل**: OpenGL 2.1 macOS limitation
**راه‌حل موقتی**: فعلاً قابل‌قبول است؛ مهم اینه مدل درست load شود

---

## مرحلهٔ بعدی: Gazebo Simulation

پس از اینکه مدل در RViz نمایش داده شد:
1. **MoveIt integration** (برای motion planning)
2. **Gazebo simulation** (برای physics و actuators)
3. **ROS2 control** (برای real-time control)

---

## مراجع

- **Niryo Ned3Pro URDF**: `/install/niryo_ned_description/share/niryo_ned_description/urdf/ned3pro/`
- **RViz2 Config**: `/install/niryo_ned_description/share/niryo_ned_description/rviz/default.rviz`
- **RoboStack**: https://robostack.github.io/
