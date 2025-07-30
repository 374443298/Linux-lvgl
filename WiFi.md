### 主要模块与功能
#### 结构体
```
// 获取WiFi信息结构体
typedef struct {
    char ssid[64];      // WiFi名称
    char ip[32];        // IP地址
    int signal_dbm;     // 信号强度 dBm
    int connected;      // 1=已连接，0=未连接
} wifi_info_t;

// WiFi网络信息结构体
typedef struct {
    char ssid[64];
    int signal_dbm;
    char security[32];  // WPA, WEP, OPEN等
    bool connected;     // 是否已连接
} wifi_network_t;

// WiFi列表结构体
typedef struct {
    wifi_network_t networks[50];  // 最多50个网络
    int count;
} wifi_scan_result_t;
```
#### 1. Wi-Fi 控制
wifi_enabled：标记Wi-Fi是否启用，用于启动和关闭Wi-Fi接口。

wifi_control_thread_func：这是一个Wi-Fi控制线程函数，它不断检查Wi-Fi开关的状态，并执行启动或关闭Wi-Fi接口的操作。

启动Wi-Fi：如果Wi-Fi开关开启，调用ifconfig wlan0 up命令启用Wi-Fi接口，然后开始扫描网络。

关闭Wi-Fi：如果Wi-Fi开关关闭，调用ifconfig wlan0 down命令禁用Wi-Fi接口，并停止网络扫描。


#### 2. Wi-Fi 扫描
wifi_scan_networks：扫描周围的Wi-Fi网络，收集相关信息（如SSID、信号强度、安全类型等）。

使用iwlist wlan0 scan命令进行扫描。

解析扫描结果并保存到wifi_scan_result_t结构体中。

auto_scan_thread：定时扫描Wi-Fi网络，每10秒扫描一次。扫描结果被存储在last_scan_result中。


#### 3. Wi-Fi 连接
wifi_connect_network：连接到指定的Wi-Fi网络，首先检查是否有保存的密码，如果没有则弹出密码输入界面。

wifi_connect_thread_func：这是一个后台线程，负责在Wi-Fi连接请求到来时启动连接操作。

wifi_save_password：保存Wi-Fi密码到配置文件中，以便下次连接时使用。


#### 4. UI 更新
wifi_update_ui_list：将扫描到的Wi-Fi网络显示在UI面板中。每个网络信息包括SSID、信号强度、安全类型、连接状态等。通过调用lv_obj_create创建UI对象，动态显示Wi-Fi列表。

wifi_check_scan_result：定期检查扫描结果并更新UI显示。


#### 5. 密码输入和断开Wi-Fi
密码键盘

当Wi-Fi网络需要密码时，弹出密码输入键盘（wifi_show_password_keyboard）。用户输入密码后，会通过wifi_connect_network函数进行连接。

password_keyboard_confirm_event：处理用户点击确认按钮时的事件，连接Wi-Fi。

password_keyboard_cancel_event：处理用户点击取消按钮时的事件，关闭密码输入界面。

Wi-Fi断开

wifi_show_disconnect_dialog：显示Wi-Fi断开确认对话框，用户确认断开时调用wifi_disconnect_network断开当前连接的Wi-Fi。


#### 6. 线程管理
使用pthread库创建和管理多个线程，如Wi-Fi控制线程、Wi-Fi连接线程、自动扫描线程等。

线程间同步使用pthread_mutex_t和pthread_cond_t实现。

#### 详细聊聊线程的各种操作
在代码中，线程是通过pthread_create函数创建的。线程函数是在后台执行的任务，可以执行一些长时间运行的操作，比如Wi-Fi扫描、Wi-Fi连接等。  
1. 线程的创建
在代码中，线程是通过pthread_create函数创建的。线程函数是在后台执行的任务，可以执行一些长时间运行的操作，比如Wi-Fi扫描、Wi-Fi连接等。

* 示例：创建Wi-Fi控制线程
```
// 创建Wi-Fi控制线程
static void wifi_control_thread_start(void) {
    if (wifi_control_running) return;
    
    wifi_control_running = true;
    if (pthread_create(&wifi_control_thread, NULL, wifi_control_thread_func, NULL) != 0) {
        printf("创建WiFi控制线程失败\n");
        wifi_control_running = false;
        return;
    }
    printf("WiFi控制线程已启动\n");
}
```
pthread_create函数的参数：

&wifi_control_thread：线程的标识符，保存新创建线程的ID。

NULL：线程的属性，默认属性可以设置为NULL。

wifi_control_thread_func：线程函数，线程将开始执行这个函数的内容。

NULL：传递给线程的参数，这里没有需要传递的参数，因此是NULL。

这个函数启动了一个新的线程来执行Wi-Fi控制操作。每个线程都是独立运行的，可以并行执行任务。

2. 线程的停止与等待
线程创建后，需要在适当的时候停止并等待线程完成。

* 示例：停止Wi-Fi控制线程
```
// 停止Wi-Fi控制线程
static void wifi_control_thread_stop(void) {
    if (!wifi_control_running) return;
    
    pthread_mutex_lock(&wifi_control_mutex);  // 加锁，保证操作的原子性
    wifi_control_running = false;
    pthread_cond_signal(&wifi_control_cond);   // 唤醒等待的线程
    pthread_mutex_unlock(&wifi_control_mutex);  // 解锁
    
    pthread_join(wifi_control_thread, NULL);   // 等待线程结束
    printf("WiFi控制线程已停止\n");
}
```
pthread_join(wifi_control_thread, NULL)：阻塞主线程，直到指定的线程（wifi_control_thread）完成执行。这个函数会等待线程执行完毕并清理资源。

3. 线程同步
在多线程程序中，不同的线程可能会访问共享资源，这时需要使用同步机制来防止数据竞争。代码中使用了**互斥锁（mutex）和条件变量（condition variable）**来确保线程安全。

互斥锁（pthread_mutex_t）
互斥锁用来保护临界区，防止多个线程同时访问共享资源。通过pthread_mutex_lock和pthread_mutex_unlock来加锁和解锁。

```
static pthread_mutex_t wifi_control_mutex = PTHREAD_MUTEX_INITIALIZER;  // 初始化互斥锁
static pthread_cond_t wifi_control_cond = PTHREAD_COND_INITIALIZER;    // 初始化条件变量

// 在wifi_control_thread_func函数中等待
while (!pending_switch_obj && wifi_control_running) {
    pthread_cond_wait(&wifi_control_cond, &wifi_control_mutex);  // 等待信号
}
```
pthread_mutex_lock(&wifi_control_mutex)：加锁，确保在访问共享资源时不会被其他线程打断。

pthread_cond_wait(&wifi_control_cond, &wifi_control_mutex)：线程等待一个条件变量，这里表示等待pending_switch_obj被设置。这个函数会在加锁的状态下进入等待，释放锁并将线程挂起，直到条件满足。

pthread_cond_signal(&wifi_control_cond)：发出一个信号，唤醒一个正在等待的线程。

互斥锁和条件变量的使用目的：
互斥锁：用来保证只有一个线程能访问共享资源。

条件变量：用来让一个线程在特定条件下等待，直到另一个线程通知它继续执行。

* 示例：Wi-Fi控制的线程同步
```
pthread_mutex_lock(&wifi_control_mutex);   // 加锁
pending_switch_obj = switch_obj;
pending_switch_state = switch_checked;    // 设置待处理的开关对象和状态
pthread_cond_signal(&wifi_control_cond);  // 通知线程继续执行
pthread_mutex_unlock(&wifi_control_mutex); // 解锁
```
上述代码中，当需要等待线程之间的某些操作时，wifi_control_mutex用来加锁，确保在修改共享变量pending_switch_obj和pending_switch_state时不会发生数据竞争。

pthread_cond_signal用来通知正在等待条件的线程继续执行。

4. 线程条件变量
条件变量在多个线程之间提供一种通信方式，让线程可以在某些条件满足时继续执行。

```
while (strlen(pending_connect_ssid) == 0 && wifi_connect_running) {
    pthread_cond_wait(&wifi_connect_cond, &wifi_connect_mutex);  // 等待连接条件
}
```
pthread_cond_wait：该函数会在条件变量wifi_connect_cond上等待，并释放wifi_connect_mutex，当被唤醒时会重新获取锁继续执行。

5. 多个线程的同步
在Wi-Fi扫描线程和控制线程之间，代码使用条件变量和互斥锁来保证多个线程能够有序执行。特别是在Wi-Fi扫描和连接的过程中，代码需要同步扫描结果和UI更新。

* 示例：Wi-Fi扫描结果更新
```
void wifi_check_scan_result(void) {
    pthread_mutex_lock(&wifi_control_mutex);   // 加锁
    if (clear_wifi_list_immediately && wifi_display_panel) {
        clear_wifi_list_immediately = false;
        pthread_mutex_unlock(&wifi_control_mutex);  // 解锁
        lv_obj_clean(wifi_display_panel);  // 清空面板内容
        return;
    }
    pthread_mutex_unlock(&wifi_control_mutex);  // 解锁

    if (scan_result_ready && current_panel) {
        wifi_update_ui_list(current_panel, &last_scan_result);  // 更新UI列表
        scan_result_ready = false;  // 清除扫描结果标记
    }
}
```
pthread_mutex_lock和pthread_mutex_unlock用于同步对clear_wifi_list_immediately的访问，确保扫描过程中不会同时清空Wi-Fi列表。

scan_result_ready表示是否有新的扫描结果，需要同步检查和更新UI。
