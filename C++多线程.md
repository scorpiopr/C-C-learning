LeetCode1195. 交替打印字符串 https://leetcode.cn/problems/fizz-buzz-multithreaded/solutions/3887522/c-python3-1195jiao-ti-da-yin-zi-fu-chuan-zw7y/
菜鸟教程 https://www.runoob.com/cplusplus/cpp-multithreading.html
------------------------------
## 一、 工业级标准的完美现代 C++ 代码
```
#include <iostream>#include <thread>#include <mutex>#include <condition_variable>#include <vector>
class AlternatePrinter {private:
    std::mutex mtx;
    std::condition_variable cv;
    int state = 0; // 🔑 核心状态机：0->A, 1->B, 2->C
    int max_loops;
public:
    AlternatePrinter(int loops) : max_loops(loops) {}

    // 打印字符 A 的线程函数
    void printA() {
        for (int i = 0; i < max_loops; ++i) {
            // 1. 现代 C++ 必须使用 unique_lock 配合条件变量
            std::unique_lock<std::mutex> lock(mtx);
            
            // 2. 🔑 绝杀点：利用 Lambda 表达式防御“虚假唤醒”
            cv.wait(lock, [this]() { return state == 0; });
            
            // 3. 执行核心业务逻辑
            std::cout << "A";
            
            // 4. 状态机流转：A 打印完，下一个轮到 B (state = 1)
            state = 1;
            
            // 5. 🔑 必须使用 notify_all，因为 notify_one 可能唤醒错误的线程导致死锁
            cv.notify_all();
        }
    }

    // 打印字符 B 的线程函数
    void printB() {
        for (int i = 0; i < max_loops; ++i) {
            std::unique_lock<std::mutex> lock(mtx);
            cv.wait(lock, [this]() { return state == 1; });
            
            std::cout << "B";
            
            state = 2; // 下一个轮到 C
            cv.notify_all();
        }
    }

    // 打印字符 C 的线程函数
    void printC() {
        for (int i = 0; i < max_loops; ++i) {
            std::unique_lock<std::mutex> lock(mtx);
            cv.wait(lock, [this]() { return state == 2; });
            
            std::cout << "C";
            
            state = 0; // 形成闭环，下一个重新轮到 A
            cv.notify_all();
        }
    }
};
int main() {
    int loops = 10; // 假设交替打印 10 次
    AlternatePrinter printer(loops);

    // 开启三个独立的硬件线程
    std::thread t1(&AlternatePrinter::printA, &printer);
    std::thread t2(&AlternatePrinter::printB, &printer);
    std::thread t3(&AlternatePrinter::printC, &printer);

    // 🔑 工业安全规范：主线程必须等待子线程执行完毕，防止生命周期提前结束导致崩溃
    t1.join();
    t2.join();
    t3.join();

    std::cout << std::endl;
    return 0;
}
```
------------------------------
## 二、 注意点
在 Code Review（码评）阶段，面试官通常会死盯着以下三个底层细节来给你评级：
## 1. 为什么 cv.wait 里面一定要写 Lambda 表达式？（考点：虚假唤醒）

* 翻车写法：很多新手会写成 if (state != 0) cv.wait(lock);。
* 致命代价：操作系统底层存在「虚假唤醒（Spurious Wakeup）」现象，即线程可能在没有任何人唤醒它的情况下，莫名其妙地从 wait 中醒来。如果用 if，醒来后线程会直接往下执行打印，从而彻底打乱交替顺序。
* 完美防线：cv.wait(lock, []{ return condition; }); 在底层等价于一个 while (!condition) { cv.wait(lock); } 循环。即使发生虚假唤醒，由于不满足状态机条件，线程会立刻被再次无情地按回休眠状态。

## 2. 为什么必须用 notify_all()，而不能用 notify_one()？（考点：线程饥饿与死锁）

* 致命代价：如果使用 notify_one()，它只会随机唤醒等待队列中的某一个线程。假设当前 state = 1（轮到 B 打印），但 notify_one() 却不幸唤醒了正在等待 state = 2 的 C 线程。C 线程醒来发现不满足 Lambda 条件，再次躺下休眠。而此时由于没有新的通知发出，三个线程将全部陷入无限死锁。
* 完美防线：使用 notify_all() 唤醒所有等待者，虽然有轻微的惊群效应，但能确保真正符合当前状态机的那个线程百分之百能拿到锁并执行。

## 3. 为什么只能用 std::unique_lock，不能用 std::lock_guard？

* 底层硬件限制：std::lock_guard 是一个极其简陋的 RAII 锁，它不支持手动解锁和重新加锁。而 std::condition_variable::wait 在把线程挂起休眠的瞬间，必须在底层原子性地释放互斥锁（否则其他线程拿不到锁，永远无法更新状态），在线程被唤醒的瞬间又要重新自动抢锁。这种复杂的流式生命周期只有 std::unique_lock 能够支持。

