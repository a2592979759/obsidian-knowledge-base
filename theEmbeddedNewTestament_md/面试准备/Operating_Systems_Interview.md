---
tags:
  - 面试准备
  - 嵌入式面试
source: "Interview_Preparation/Specialized_Domains/Operating_Systems_Interview.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入
>
> 在网站上刷社区排名的题库、带 AI 反馈的编程练习，以及结构化的面试准备。
>
> 👉 **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_preparation)** &nbsp;·&nbsp; **[探索面试准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_preparation)**

---

# 🎯 操作系统面试准备

## 🚀 **快速导航**
- [常见问题](#常见问题)
- [问题求解示例](#问题求解示例)
- [练习题](#练习题)
- [资源](#资源)

## 📚 **速查表：核心概念**
- **Linux 内核（Linux Kernel）**：内核模块、设备驱动、系统调用（system calls）、内存管理
- **设备驱动（Device Drivers）**：字符（character）、块（block）、网络驱动、生命周期管理
- **实时 Linux（Real-time Linux）**：PREEMPT_RT、Xenomai、实时扩展
- **嵌入式 Linux（Embedded Linux）**：Buildroot、Yocto、自定义发行版
- **系统编程（System Programming）**：POSIX API、进程管理、进程间通信（inter-process communication）

## 🎯 **常见面试问题**

### **问题 1：实现一个带 ioctl 支持的字符设备驱动**

**为什么这很重要**：设备驱动是嵌入式 Linux 系统的基础，能展示内核编程技能。

**问题**：为一个支持读、写与 ioctl 操作的自定义传感器创建一个字符设备驱动。

**需求**：
- 支持读/写操作
- 实现自定义 ioctl 命令
- 处理多个设备实例
- 支持 poll/select 操作

**方案设计**：
```c
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/uaccess.h>
#include <linux/device.h>
#include <linux/cdev.h>
#include <linux/slab.h>
#include <linux/poll.h>
#include <linux/wait.h>

#define DEVICE_NAME "custom_sensor"
#define MAX_DEVICES 4
#define BUFFER_SIZE 1024

// Custom ioctl commands
#define SENSOR_GET_STATUS _IOR('S', 0, int)
#define SENSOR_SET_MODE _IOW('S', 1, int)
#define SENSOR_GET_CALIBRATION _IOR('S', 2, struct sensor_calib)
#define SENSOR_SET_CALIBRATION _IOW('S', 3, struct sensor_calib)

// Sensor calibration structure
struct sensor_calib {
    int offset;
    int scale;
    int temperature;
};

// Device private data
struct sensor_dev {
    struct cdev cdev;
    struct device *device;
    char buffer[BUFFER_SIZE];
    size_t buffer_len;
    struct mutex lock;
    wait_queue_head_t read_queue;
    bool data_ready;
    int mode;
    struct sensor_calib calibration;
};

static struct sensor_dev *sensor_devices[MAX_DEVICES];
static int major_number;
static struct class *sensor_class;

// File operations
static int sensor_open(struct inode *inode, struct file *file) {
    struct sensor_dev *dev = container_of(inode->i_cdev, struct sensor_dev, cdev);
    file->private_data = dev;
    
    pr_info("Sensor device opened\n");
    return 0;
}

static int sensor_release(struct inode *inode, struct file *file) {
    pr_info("Sensor device closed\n");
    return 0;
}

static ssize_t sensor_read(struct file *file, char __user *buf, 
                          size_t count, loff_t *ppos) {
    struct sensor_dev *dev = file->private_data;
    ssize_t ret = 0;
    
    if (mutex_lock_interruptible(&dev->lock))
        return -ERESTARTSYS;
    
    // Wait for data to be ready
    while (!dev->data_ready && !signal_pending(current)) {
        mutex_unlock(&dev->lock);
        if (wait_event_interruptible(dev->read_queue, dev->data_ready))
            return -ERESTARTSYS;
        if (mutex_lock_interruptible(&dev->lock))
            return -ERESTARTSYS;
    }
    
    if (dev->data_ready) {
        count = min(count, dev->buffer_len);
        if (copy_to_user(buf, dev->buffer, count)) {
            ret = -EFAULT;
        } else {
            ret = count;
            dev->data_ready = false;
        }
    }
    
    mutex_unlock(&dev->lock);
    return ret;
}

static ssize_t sensor_write(struct file *file, const char __user *buf,
                           size_t count, loff_t *ppos) {
    struct sensor_dev *dev = file->private_data;
    ssize_t ret = 0;
    
    if (mutex_lock_interruptible(&dev->lock))
        return -ERESTARTSYS;
    
    count = min(count, BUFFER_SIZE);
    if (copy_from_user(dev->buffer, buf, count)) {
        ret = -EFAULT;
    } else {
        dev->buffer_len = count;
        dev->data_ready = true;
        wake_up_interruptible(&dev->read_queue);
        ret = count;
    }
    
    mutex_unlock(&dev->lock);
    return ret;
}

static long sensor_ioctl(struct file *file, unsigned int cmd, unsigned long arg) {
    struct sensor_dev *dev = file->private_data;
    long ret = 0;
    
    if (mutex_lock_interruptible(&dev->lock))
        return -ERESTARTSYS;
    
    switch (cmd) {
        case SENSOR_GET_STATUS:
            ret = put_user(dev->mode, (int __user *)arg);
            break;
            
        case SENSOR_SET_MODE:
            ret = get_user(dev->mode, (int __user *)arg);
            pr_info("Sensor mode set to %d\n", dev->mode);
            break;
            
        case SENSOR_GET_CALIBRATION:
            ret = copy_to_user((struct sensor_calib __user *)arg, 
                              &dev->calibration, sizeof(struct sensor_calib));
            if (ret)
                ret = -EFAULT;
            break;
            
        case SENSOR_SET_CALIBRATION:
            ret = copy_from_user(&dev->calibration, 
                                (struct sensor_calib __user *)arg, 
                                sizeof(struct sensor_calib));
            if (ret)
                ret = -EFAULT;
            else
                pr_info("Calibration updated: offset=%d, scale=%d\n", 
                       dev->calibration.offset, dev->calibration.scale);
            break;
            
        default:
            ret = -ENOTTY;
            break;
    }
    
    mutex_unlock(&dev->lock);
    return ret;
}

static unsigned int sensor_poll(struct file *file, poll_table *wait) {
    struct sensor_dev *dev = file->private_data;
    unsigned int mask = 0;
    
    poll_wait(file, &dev->read_queue, wait);
    
    if (mutex_lock_interruptible(&dev->lock))
        return -ERESTARTSYS;
    
    if (dev->data_ready)
        mask |= POLLIN | POLLRDNORM;
    
    mutex_unlock(&dev->lock);
    return mask;
}

static const struct file_operations sensor_fops = {
    .owner = THIS_MODULE,
    .open = sensor_open,
    .release = sensor_release,
    .read = sensor_read,
    .write = sensor_write,
    .unlocked_ioctl = sensor_ioctl,
    .poll = sensor_poll,
};

// Module initialization
static int __init sensor_init(void) {
    int ret, i;
    dev_t dev;
    
    // Allocate major number
    ret = alloc_chrdev_region(&dev, 0, MAX_DEVICES, DEVICE_NAME);
    if (ret < 0) {
        pr_err("Failed to allocate major number\n");
        return ret;
    }
    major_number = MAJOR(dev);
    
    // Create device class
    sensor_class = class_create(THIS_MODULE, DEVICE_NAME);
    if (IS_ERR(sensor_class)) {
        pr_err("Failed to create device class\n");
        unregister_chrdev_region(dev, MAX_DEVICES);
        return PTR_ERR(sensor_class);
    }
    
    // Initialize devices
    for (i = 0; i < MAX_DEVICES; i++) {
        sensor_devices[i] = kzalloc(sizeof(struct sensor_dev), GFP_KERNEL);
        if (!sensor_devices[i]) {
            ret = -ENOMEM;
            goto cleanup;
        }
        
        // Initialize device
        cdev_init(&sensor_devices[i]->cdev, &sensor_fops);
        sensor_devices[i]->cdev.owner = THIS_MODULE;
        
        // Add device
        dev = MKDEV(major_number, i);
        ret = cdev_add(&sensor_devices[i]->cdev, dev, 1);
        if (ret < 0) {
            pr_err("Failed to add device %d\n", i);
            goto cleanup;
        }
        
        // Create device file
        sensor_devices[i]->device = device_create(sensor_class, NULL, dev, 
                                                NULL, "%s%d", DEVICE_NAME, i);
        if (IS_ERR(sensor_devices[i]->device)) {
            pr_err("Failed to create device file %d\n", i);
            cdev_del(&sensor_devices[i]->cdev);
            goto cleanup;
        }
        
        // Initialize device data
        mutex_init(&sensor_devices[i]->lock);
        init_waitqueue_head(&sensor_devices[i]->read_queue);
        sensor_devices[i]->mode = 0;
        sensor_devices[i]->calibration.offset = 0;
        sensor_devices[i]->calibration.scale = 1000;
    }
    
    pr_info("Sensor driver loaded successfully\n");
    return 0;
    
cleanup:
    for (i = 0; i < MAX_DEVICES; i++) {
        if (sensor_devices[i]) {
            if (sensor_devices[i]->device)
                device_destroy(sensor_class, MKDEV(major_number, i));
            cdev_del(&sensor_devices[i]->cdev);
            kfree(sensor_devices[i]);
        }
    }
    class_destroy(sensor_class);
    unregister_chrdev_region(MKDEV(major_number, 0), MAX_DEVICES);
    return ret;
}

// Module cleanup
static void __exit sensor_exit(void) {
    int i;
    
    for (i = 0; i < MAX_DEVICES; i++) {
        if (sensor_devices[i]) {
            device_destroy(sensor_class, MKDEV(major_number, i));
            cdev_del(&sensor_devices[i]->cdev);
            kfree(sensor_devices[i]);
        }
    }
    
    class_destroy(sensor_class);
    unregister_chrdev_region(MKDEV(major_number, 0), MAX_DEVICES);
    
    pr_info("Sensor driver unloaded\n");
}

module_init(sensor_init);
module_exit(sensor_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Embedded Engineer");
MODULE_DESCRIPTION("Custom Sensor Driver");
```

**驱动特性**：
- **多实例**：支持最多 4 个设备
- **Ioctl 支持**：用于配置的自定义命令
- **Poll/Select**：异步 I/O 支持
- **正确同步**：互斥锁与等待队列

**追问**：
- 你会如何处理中断驱动的 I/O？
- 如果需要支持 DMA 传输，怎么办？

### **问题 2：使用 PREEMPT_RT 设计一个实时任务调度器**

**问题**：实现一个能处理带截止期限的周期性任务的实时任务调度器。

**需求**：
- 支持周期性任务
- 处理任务优先级
- 实现截止期限监控
- 支持任务抢占

**方案设计**：
```c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/sched.h>
#include <linux/sched/rt.h>
#include <linux/timer.h>
#include <linux/workqueue.h>
#include <linux/spinlock.h>
#include <linux/list.h>

#define MAX_TASKS 16
#define MAX_PRIORITY 99

// Task states
typedef enum {
    TASK_STATE_READY,
    TASK_STATE_RUNNING,
    TASK_STATE_WAITING,
    TASK_STATE_COMPLETED
} task_state_t;

// Real-time task structure
typedef struct rt_task {
    struct list_head list;
    int task_id;
    char name[32];
    void (*function)(void*);
    void *data;
    unsigned long period;      // Period in nanoseconds
    unsigned long deadline;    // Deadline in nanoseconds
    unsigned long last_release; // Last release time
    unsigned long next_release; // Next release time
    int priority;
    task_state_t state;
    unsigned long execution_time;
    unsigned long missed_deadlines;
    spinlock_t lock;
} rt_task_t;

// Scheduler context
typedef struct {
    struct list_head ready_queue;
    struct list_head waiting_queue;
    rt_task_t *current_task;
    spinlock_t scheduler_lock;
    struct timer_list scheduler_timer;
    unsigned long current_time;
    int num_tasks;
} rt_scheduler_t;

static rt_scheduler_t scheduler;

// Initialize real-time task
int rt_task_create(rt_task_t *task, const char *name, 
                   void (*function)(void*), void *data,
                   unsigned long period, unsigned long deadline, int priority) {
    if (!task || !function || period == 0 || deadline == 0)
        return -EINVAL;
    
    if (priority < 1 || priority > MAX_PRIORITY)
        return -EINVAL;
    
    // Initialize task
    task->task_id = scheduler.num_tasks++;
    strncpy(task->name, name, sizeof(task->name) - 1);
    task->function = function;
    task->data = data;
    task->period = period;
    task->deadline = deadline;
    task->priority = priority;
    task->state = TASK_STATE_READY;
    task->execution_time = 0;
    task->missed_deadlines = 0;
    
    // Set initial release time
    task->last_release = 0;
    task->next_release = ktime_get_ns();
    
    spin_lock_init(&task->lock);
    
    // Add to ready queue
    spin_lock(&scheduler.scheduler_lock);
    list_add_tail(&task->list, &scheduler.ready_queue);
    spin_unlock(&scheduler.scheduler_lock);
    
    pr_info("RT Task '%s' created with period %lu ns, deadline %lu ns\n", 
            name, period, deadline);
    
    return 0;
}

// Start real-time task
int rt_task_start(rt_task_t *task) {
    unsigned long flags;
    
    spin_lock_irqsave(&task->lock, flags);
    
    if (task->state != TASK_STATE_READY) {
        spin_unlock_irqrestore(&task->lock, flags);
        return -EINVAL;
    }
    
    task->state = TASK_STATE_WAITING;
    task->next_release = ktime_get_ns();
    
    spin_unlock_irqrestore(&task->lock, flags);
    
    pr_info("RT Task '%s' started\n", task->name);
    return 0;
}

// Stop real-time task
int rt_task_stop(rt_task_t *task) {
    unsigned long flags;
    
    spin_lock_irqsave(&task->lock, flags);
    
    if (task->state == TASK_STATE_RUNNING) {
        // Preempt current task
        if (scheduler.current_task == task) {
            scheduler.current_task = NULL;
        }
    }
    
    task->state = TASK_STATE_READY;
    
    spin_unlock_irqrestore(&task->lock, flags);
    
    pr_info("RT Task '%s' stopped\n", task->name);
    return 0;
}

// Set task priority
int rt_task_set_priority(rt_task_t *task, int priority) {
    unsigned long flags;
    
    if (priority < 1 || priority > MAX_PRIORITY)
        return -EINVAL;
    
    spin_lock_irqsave(&task->lock, flags);
    task->priority = priority;
    spin_unlock_irqrestore(&task->lock, flags);
    
    return 0;
}

// Scheduler timer callback
static void scheduler_timer_callback(struct timer_list *t) {
    unsigned long flags;
    rt_task_t *task, *next_task = NULL;
    unsigned long current_time = ktime_get_ns();
    
    spin_lock_irqsave(&scheduler.scheduler_lock, flags);
    
    // Check for tasks that need to be released
    list_for_each_entry(task, &scheduler.waiting_queue, list) {
        if (current_time >= task->next_release) {
            // Move to ready queue
            list_del(&task->list);
            list_add_tail(&task->list, &scheduler.ready_queue);
            task->state = TASK_STATE_READY;
            task->last_release = task->next_release;
            task->next_release += task->period;
            
            pr_debug("Task '%s' released at %lu\n", task->name, current_time);
        }
    }
    
    // Select next task to run (highest priority first)
    if (!scheduler.current_task || scheduler.current_task->state != TASK_STATE_RUNNING) {
        list_for_each_entry(task, &scheduler.ready_queue, list) {
            if (!next_task || task->priority > next_task->priority) {
                next_task = task;
            }
        }
        
        if (next_task) {
            // Preempt current task if needed
            if (scheduler.current_task && 
                next_task->priority > scheduler.current_task->priority) {
                scheduler.current_task->state = TASK_STATE_READY;
                pr_debug("Task '%s' preempted by '%s'\n", 
                        scheduler.current_task->name, next_task->name);
            }
            
            scheduler.current_task = next_task;
            next_task->state = TASK_STATE_RUNNING;
            
            // Execute task
            spin_unlock_irqrestore(&scheduler.scheduler_lock, flags);
            next_task->function(next_task->data);
            spin_lock_irqsave(&scheduler.scheduler_lock, flags);
            
            next_task->state = TASK_STATE_WAITING;
            scheduler.current_task = NULL;
            
            // Check deadline
            if (ktime_get_ns() > next_task->last_release + next_task->deadline) {
                next_task->missed_deadlines++;
                pr_warn("Task '%s' missed deadline\n", next_task->name);
            }
        }
    }
    
    spin_unlock_irqrestore(&scheduler.scheduler_lock, flags);
    
    // Reschedule timer
    mod_timer(&scheduler.scheduler_timer, jiffies + 1);
}

// Initialize scheduler
static int __init rt_scheduler_init(void) {
    // Initialize scheduler
    INIT_LIST_HEAD(&scheduler.ready_queue);
    INIT_LIST_HEAD(&scheduler.waiting_queue);
    scheduler.current_task = NULL;
    scheduler.num_tasks = 0;
    spin_lock_init(&scheduler.scheduler_lock);
    
    // Setup scheduler timer
    timer_setup(&scheduler.scheduler_timer, scheduler_timer_callback, 0);
    mod_timer(&scheduler.scheduler_timer, jiffies + 1);
    
    pr_info("Real-time scheduler initialized\n");
    return 0;
}

// Cleanup scheduler
static void __exit rt_scheduler_exit(void) {
    del_timer_sync(&scheduler.scheduler_timer);
    pr_info("Real-time scheduler unloaded\n");
}

module_init(rt_scheduler_init);
module_exit(rt_scheduler_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Embedded Engineer");
MODULE_DESCRIPTION("Real-time Task Scheduler");
```

**调度器特性**：
- **基于优先级调度**：高优先级任务先运行
- **截止期限监控**：跟踪错过的截止期限
- **任务抢占**：更高优先级任务可抢占低优先级任务
- **周期性执行**：支持周期性实时任务

### **问题 3：为嵌入式 Linux 实现一个自定义 init 系统**

**问题**：创建一个能启动与管理系统服务的轻量 init 系统。

**需求**：
- 服务依赖管理
- 服务生命周期控制
- 配置文件解析
- 日志与监控

**方案设计**：
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <signal.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <errno.h>
#include <time.h>

#define MAX_SERVICES 32
#define MAX_DEPENDENCIES 8
#define MAX_LINE_LENGTH 256
#define CONFIG_FILE "/etc/init.conf"

// Service states
typedef enum {
    SERVICE_STATE_STOPPED,
    SERVICE_STATE_STARTING,
    SERVICE_STATE_RUNNING,
    SERVICE_STATE_STOPPING,
    SERVICE_STATE_FAILED
} service_state_t;

// Service structure
typedef struct {
    char name[64];
    char command[256];
    char dependencies[MAX_DEPENDENCIES][64];
    int num_dependencies;
    pid_t pid;
    service_state_t state;
    int restart_count;
    int max_restarts;
    bool auto_start;
    time_t start_time;
    time_t last_restart;
} service_t;

// Init system context
typedef struct {
    service_t services[MAX_SERVICES];
    int num_services;
    bool running;
    int log_fd;
} init_system_t;

static init_system_t init_system;

// Logging functions
void init_log(const char *format, ...) {
    va_list args;
    char timestamp[64];
    time_t now = time(NULL);
    
    strftime(timestamp, sizeof(timestamp), "%Y-%m-%d %H:%M:%S", localtime(&now));
    
    dprintf(init_system.log_fd, "[%s] ", timestamp);
    
    va_start(args, format);
    vdprintf(init_system.log_fd, format, args);
    va_end(args);
    
    dprintf(init_system.log_fd, "\n");
}

// Parse configuration file
int parse_config_file(const char *filename) {
    FILE *file = fopen(filename, "r");
    if (!file) {
        perror("Failed to open config file");
        return -1;
    }
    
    char line[MAX_LINE_LENGTH];
    service_t *current_service = NULL;
    
    while (fgets(line, sizeof(line), file)) {
        // Skip comments and empty lines
        if (line[0] == '#' || line[0] == '\n')
            continue;
        
        // Remove newline
        line[strcspn(line, "\n")] = 0;
        
        if (strncmp(line, "service ", 8) == 0) {
            // New service definition
            if (init_system.num_services >= MAX_SERVICES) {
                init_log("Too many services, maximum is %d", MAX_SERVICES);
                break;
            }
            
            current_service = &init_system.services[init_system.num_services++];
            memset(current_service, 0, sizeof(service_t));
            
            // Parse service name
            char *name = line + 8;
            strncpy(current_service->name, name, sizeof(current_service->name) - 1);
            current_service->max_restarts = 3;
            current_service->auto_start = true;
            
            init_log("Parsing service: %s", current_service->name);
            
        } else if (current_service && strncmp(line, "  command ", 11) == 0) {
            // Service command
            char *command = line + 11;
            strncpy(current_service->command, command, sizeof(current_service->command) - 1);
            
        } else if (current_service && strncmp(line, "  depends ", 10) == 0) {
            // Service dependencies
            char *deps = line + 10;
            char *token = strtok(deps, " ");
            
            while (token && current_service->num_dependencies < MAX_DEPENDENCIES) {
                strncpy(current_service->dependencies[current_service->num_dependencies], 
                       token, 63);
                current_service->num_dependencies++;
                token = strtok(NULL, " ");
            }
            
        } else if (current_service && strncmp(line, "  max_restarts ", 15) == 0) {
            // Maximum restart count
            current_service->max_restarts = atoi(line + 15);
            
        } else if (current_service && strncmp(line, "  auto_start ", 13) == 0) {
            // Auto-start setting
            if (strstr(line, "true"))
                current_service->auto_start = true;
            else
                current_service->auto_start = false;
        }
    }
    
    fclose(file);
    return 0;
}

// Check if service dependencies are met
bool check_dependencies(service_t *service) {
    for (int i = 0; i < service->num_dependencies; i++) {
        bool found = false;
        
        for (int j = 0; j < init_system.num_services; j++) {
            if (strcmp(service->dependencies[i], init_system.services[j].name) == 0) {
                if (init_system.services[j].state != SERVICE_STATE_RUNNING) {
                    return false;  // Dependency not running
                }
                found = true;
                break;
            }
        }
        
        if (!found) {
            init_log("Service %s depends on unknown service %s", 
                    service->name, service->dependencies[i]);
            return false;
        }
    }
    
    return true;
}

// Start a service
int start_service(service_t *service) {
    if (service->state != SERVICE_STATE_STOPPED && 
        service->state != SERVICE_STATE_FAILED) {
        return -1;  // Service already running or starting
    }
    
    // Check dependencies
    if (!check_dependencies(service)) {
        init_log("Service %s dependencies not met", service->name);
        return -1;
    }
    
    service->state = SERVICE_STATE_STARTING;
    init_log("Starting service: %s", service->name);
    
    // Fork and exec service
    pid_t pid = fork();
    if (pid == 0) {
        // Child process
        close(init_system.log_fd);
        
        // Redirect output to log file
        int log_fd = open("/var/log/services.log", O_WRONLY | O_APPEND | O_CREAT, 0644);
        if (log_fd >= 0) {
            dup2(log_fd, STDOUT_FILENO);
            dup2(log_fd, STDERR_FILENO);
            close(log_fd);
        }
        
        // Execute service command
        execl("/bin/sh", "sh", "-c", service->command, NULL);
        exit(1);
        
    } else if (pid > 0) {
        // Parent process
        service->pid = pid;
        service->start_time = time(NULL);
        service->state = SERVICE_STATE_RUNNING;
        
        init_log("Service %s started with PID %d", service->name, pid);
        return 0;
        
    } else {
        // Fork failed
        service->state = SERVICE_STATE_FAILED;
        init_log("Failed to start service %s: fork error", service->name);
        return -1;
    }
}

// Stop a service
int stop_service(service_t *service) {
    if (service->state != SERVICE_STATE_RUNNING) {
        return -1;  // Service not running
    }
    
    service->state = SERVICE_STATE_STOPPING;
    init_log("Stopping service: %s", service->name);
    
    // Send SIGTERM to service
    if (kill(service->pid, SIGTERM) == 0) {
        // Wait for service to terminate
        int status;
        pid_t result = waitpid(service->pid, &status, WNOHANG);
        
        if (result == 0) {
            // Service still running, send SIGKILL after timeout
            sleep(5);
            if (kill(service->pid, SIGKILL) == 0) {
                waitpid(service->pid, &status, 0);
            }
        }
    }
    
    service->pid = 0;
    service->state = SERVICE_STATE_STOPPED;
    init_log("Service %s stopped", service->name);
    
    return 0;
}

// Restart a service
int restart_service(service_t *service) {
    init_log("Restarting service: %s", service->name);
    
    stop_service(service);
    sleep(1);  // Brief delay between stop and start
    return start_service(service);
}

// Monitor services
void monitor_services() {
    int status;
    pid_t pid;
    
    while ((pid = waitpid(-1, &status, WNOHANG)) > 0) {
        // Find service by PID
        for (int i = 0; i < init_system.num_services; i++) {
            if (init_system.services[i].pid == pid) {
                service_t *service = &init_system.services[i];
                
                if (WIFEXITED(status)) {
                    int exit_code = WEXITSTATUS(status);
                    init_log("Service %s exited with code %d", service->name, exit_code);
                    
                    if (exit_code != 0 && service->restart_count < service->max_restarts) {
                        service->restart_count++;
                        service->last_restart = time(NULL);
                        init_log("Restarting service %s (attempt %d/%d)", 
                                service->name, service->restart_count, service->max_restarts);
                        
                        start_service(service);
                    } else {
                        service->state = SERVICE_STATE_FAILED;
                        init_log("Service %s failed, not restarting", service->name);
                    }
                } else if (WIFSIGNALED(status)) {
                    int signal = WTERMSIG(status);
                    init_log("Service %s killed by signal %d", service->name, signal);
                    service->state = SERVICE_STATE_STOPPED;
                }
                
                service->pid = 0;
                break;
            }
        }
    }
}

// Signal handler
void signal_handler(int sig) {
    if (sig == SIGTERM || sig == SIGINT) {
        init_log("Received shutdown signal, stopping all services");
        init_system.running = false;
    }
}

// Main init system loop
int main(int argc, char *argv[]) {
    // Open log file
    init_system.log_fd = open("/var/log/init.log", O_WRONLY | O_APPEND | O_CREAT, 0644);
    if (init_system.log_fd < 0) {
        perror("Failed to open log file");
        return 1;
    }
    
    init_log("Custom init system starting");
    
    // Parse configuration
    if (parse_config_file(CONFIG_FILE) < 0) {
        init_log("Failed to parse configuration file");
        return 1;
    }
    
    init_log("Loaded %d services", init_system.num_services);
    
    // Set up signal handlers
    signal(SIGTERM, signal_handler);
    signal(SIGINT, signal_handler);
    signal(SIGCHLD, SIG_IGN);  // Let waitpid handle child processes
    
    // Start auto-start services
    for (int i = 0; i < init_system.num_services; i++) {
        if (init_system.services[i].auto_start) {
            start_service(&init_system.services[i]);
        }
    }
    
    // Main loop
    init_system.running = true;
    while (init_system.running) {
        monitor_services();
        usleep(100000);  // 100ms sleep
    }
    
    // Shutdown: stop all services
    for (int i = 0; i < init_system.num_services; i++) {
        if (init_system.services[i].state == SERVICE_STATE_RUNNING) {
            stop_service(&init_system.services[i]);
        }
    }
    
    init_log("Init system shutdown complete");
    close(init_system.log_fd);
    
    return 0;
}
```

**init 系统特性**：
- **配置解析**：从配置文件解析服务定义
- **依赖管理**：按依赖顺序启动服务
- **服务监控**：追踪服务状态并重启失败的服务
- **优雅关闭**：收到关闭信号时停止所有服务

## 🧪 **练习题**

### **问题 1：内核模块内存管理**

**场景**：分析内核模块中的内存管理。

**问题**：当内核模块分配内存时会发生什么，应如何管理？

**预期分析**：
```
1. 内存分配方法：
   - kmalloc：用于小分配，返回物理连续内存
   - vmalloc：用于大分配，可能非物理连续
   - get_free_pages：用于页对齐分配
   - kmem_cache_alloc：用于频繁分配的对象

2. 内存管理：
   - 始终在 module_exit 中释放已分配内存
   - 使用合适的分配标志（GFP_KERNEL、GFP_ATOMIC）
   - 检查分配失败的返回值
   - 考虑内存碎片与对齐

3. 常见问题：
   - 未释放分配导致的内存泄漏
   - 在中断上下文使用错误的分配标志
   - 未处理分配失败
```

### **问题 2：实时性能分析**

**场景**：分析 Linux 系统的实时性能。

**问题**：你会如何测量与改善实时性能？

**预期方案**：
```
1. 性能测量：
   - 使用 cyclictest 测量延迟
   - 用 /proc/interrupts 监控内核抢占
   - 检查 PREEMPT_RT 补丁状态
   - 使用 ftrace 进行调度分析

2. 性能改善：
   - 启用 PREEMPT_RT 补丁
   - 使用实时调度策略（SCHED_FIFO、SCHED_RR）
   - 最小化中断处理时间
   - 使用高分辨率定时器

3. 分析工具：
   - cyclictest、ftrace、perf
   - 实时监控工具
   - 内核配置分析
```

### **问题 3：设备驱动错误处理**

**场景**：设计设备驱动的错误处理。

**问题**：你会如何实现稳健的错误处理？

**预期方案**：
```
1. 错误类别：
   - 硬件错误（设备无响应）
   - 软件错误（无效参数）
   - 资源错误（内存分配失败）
   - 通信错误（I/O 失败）

2. 错误处理策略：
   - 返回合适的错误码
   - 记录足够详细的错误
   - 实现重试机制
   - 提供用户空间错误信息

3. 恢复机制：
   - 关键错误时设备复位
   - 优雅降级
   - 带退避的自动重试
   - 错误时通知用户
```

## ✅ **自我评估清单**

### **Linux 内核编程** ✅
- [ ] 能编写内核模块
- [ ] 理解设备驱动架构
- [ ] 能实现系统调用
- [ ] 掌握内核内存管理

### **设备驱动** ✅
- [ ] 能实现字符驱动
- [ ] 理解驱动生命周期
- [ ] 能处理中断与 DMA
- [ ] 掌握驱动调试技术

### **实时 Linux** ✅
- [ ] 能配置 PREEMPT_RT
- [ ] 理解实时调度
- [ ] 能测量实时性能
- [ ] 掌握实时最佳实践

### **嵌入式 Linux** ✅
- [ ] 能构建自定义发行版
- [ ] 理解构建系统
- [ ] 能针对嵌入式使用优化
- [ ] 掌握部署策略

## 🔗 **相关主题**
- [[Linux_Kernel_Programming]]
- [[Device_Drivers]]
- [[Real_time_Linux]]
- [[Embedded_Linux]]
- [[System_Programming]]

## 📚 **附加资源**
- **书籍**：《Linux Device Drivers》作者 Jonathan Corbet
- **在线**：[Linux 内核文档](https://www.kernel.org/doc/)
- **练习**：[Linux 内核新手](https://kernelnewbies.org/)
- **标准**：[POSIX 标准](https://pubs.opengroup.org/onlinepubs/9699919799/)

## 相关页面

- [[Real_Time_Systems_Interview]]
- [[System_Integration_Interview]]
- [[Embedded_Security_Interview_advanced]]
- [[Technical_Interview_Guide]]
- [[00-索引]]

返回索引 [[00-索引]]
