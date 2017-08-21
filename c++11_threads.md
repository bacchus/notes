# C++11 Thread Library

<a id="home"></a>
* [Part 1 - threads control         ](#p1)
* [Part 2 - data share              ](#p2)
* [Part 3 - threads sync            ](#p3)
* [Part 4 - std::atomic             ](#p4)
* [Part 5 - structs with locks      ](#p5)
* [Part 6 - lockfree structs        ](#p6)
* [Part 7 - parallel program design ](#p7)
* [Part 8 - thread_pool             ](#p8)
* [Part 9 - test and debug          ](#p9)

<a id="p1"></a>
## Part 1 - threads control [home](#home)
- concurent tasks vs data  
- noncopyable: thread, unique_lock, future, promise, packaged_task

#### create
param copy, so passed param can be destr immediately:

    std::thread t(func);
    std::thread t{someClass()};
    std::thread t([](){});

#### join
do not store refs or ptrs to local objs that can be destr before t finishes  
must be called before t destr, else terminate()  
call once, after call has no connection to thread

    t.join();       // wait
    t.detach();     // no wait, t works in bg even after destr
    t.joinable();   // check if not joined or detached

bad case: join() not called

    std::thread t(func);
    throw 42;
    t.join();

solve ugly:

    std::thread t(func);
    try {
        throw 42;
    } catch(...) {
        t.join();
    }
    t.join();

beter RAII: ~guard() { if (t.joinable()) t.join(); }

    std::thread t(func);
    guard g(t);
    throw 42;

#### args

    std::thread t(func, args...);

bad: buff can be destr before conversion (std::thread() copys args as is)

    void func(const std::string&);
    char buff[1024];
    std::thread t(func, buff);// const char* -> std::string
    t.detach();

ok:

    char buff[1024];
    std::thread t(func, std::string(buff));
    t.detach();

#### refs
bad:

    void func(Data&);
    Data data;
    std::thread t(func, data); // copys data
    t.join();
    display_data(data); // old data

use std::ref:

    Data data;
    std::thread t(func, std::ref(data)); // ref to data
    t.join();
    display_data(data); // good data

#### ptr to member
    class X { public: run() {} };
    X myx;
    std::thread t(&X::run, &myx);

#### movable Data
    void func(std::unique_ptr<Data>);
    std::unique_ptr<Data> data(new Data);
    std::thread t(func, std::move(data));

#### std::thread - movable, noncopyable
    std::thread t1(func);
    std::thread t2 = std::move(t1);
    std::thread t3; t3 = std::move(t2);
    
bad:

    std::thread t1(func1);
    std::thread t2(func2);
    t1 = std::move(t2); // terminate, because t1 lost, destr


#### return std::thread
    std::thread g() {
        std::thread t(func);
        return t; // auto move
        return std::thread(func); // also ok
    }

#### as param
    void g(std::thread);
    std::thread t(func);
    g(std::move(t))
    g(std::thread(func)); // auto move


#### beter RAII guard:
    scoped_thread(std::thread t1): t(std::move(t1)) { 
        if (!t.joinable()) throw std::logic_error("no thread");
    }

    ~scoped_thread() { t.join(); }

    scoped_thread(std::thread(func));

#### in container
    std::vector<std::thread> threads;
    for (i: 0..n) threads.push_back(std::thread(func, i));
    for_each(threads, std::mem_fn(&std::thread::join));

#### thread id
    std::thread::hardware_concurency(); // N real threads
    std::this_thread::get_id();     // this thread id
    std::thread::id i = t.get_id(); // t-s id
    id1 == id2 // same thread or both no_thread
    id1 != id2 // diff threads or one of them is no_thread


***

<a id="p2"></a>
## Part 2 - data share [home](#home)

- race condition - breaks invariant (db-linked list delete el)
- data race - simultaneously modify same obj
  , nonsynchormized access, wrong atomic usage
  , access not protected data from diff threads
- 1 global mutex vs many mutexes (granularity)
- many mutexes can cause deadlock

#### locking
- lock(), unlock()
- try_lock() - not blocking, if already locked return false

use lock_guard:

    std::mutex m; 
    std::lock_guard<std::mutex> g(m);

#### race
do not give ptr or ref to private class data  
, as outer code can modify it without mutex lock  
also data passed to func as ref or ptr  
or saving it to memory visible from outer code  

    class C {
    public:
        template<typename Func>
        void process(Func func) {
            std::lock_guard<std::mutex> g(m);
            func(data);
        }
    private:
        std::mutex m;
        Data data;
    }

    Data* static_data; 
    void malicious_func(Data& d) {
        static_data = &d;
    }

    main() {
        C c; 
        c.process(malicious_func);
        static_data->break_security();
    }


race: stack size > 1 - process same val (3), val (2) lost  
stack: [1,2,3]

    if (!stack.empty()) {                                   // true 
                                if (!stack.empty()) {       // true
        int val = stack.top();                              // val = 3
                                    int val = stack.top();  // val = 3
        stack.pop();                                        // [1,2]
                                    stack.pop();            // [1]
        process(val);                                       // process(3)
                                    process(val);           // process(3)

or top() from empty stack - undefined behavior  
stack size = 1

    if (!stack.empty()) {                                   // true
                                if (!stack.empty()) {       // true
        int val = stack.top();  
        stack.pop();                                        // stack empty
                                    int val = stack.top();  // undefined

#### solution
    std::shared_ptr<T> threadsafe_stack::pop() {
        std::lock_guard<std::mutex> g(m);
        if (stack.empty()) throw empty_stack();
        const std::shared_ptr<T> res(std::make_shared(stack.top()));
        stack.pop();
        return res;
    }

    struct empty_stack: std::exception {
        const char* what() const throw();
    };

#### deadlock
call lock() 2 times on same mutex - undefined behavior  
std::adopt_lock - means already locked  
std::lock(m1, m2, ...); // simult locks m1, m2, ...  
if exception in any lock()/unlock() all locks unlocked and rethrows

    class X;
    void swap(X& lhs, X& rhs);

    class X {
        Data data;
        std::mutex m;
    public:
        friend void swap(X& lhs, X& rhs) {
            if (&lhs == &rhs)
                return; 
            std::lock(lhs.m, rhs.m);    // saves from deadlock
            std::lock_guard<std::mutex> g1(lhs.m, std::adopt_lock);
            std::lock_guard<std::mutex> g2(rhs.m, std::adopt_lock);
            swap(lhs.data, rhs.data);
        }
    }


#### funy thread deadlock
solution: create and join thread in same function

    std::thread t1([](){    std::thread t2([](){
        ...                     ...
        t2.join();                                  // t1 waits for t2
                                t1.join();          // t2 waits for t1
    })                      })

#### advices
- avoid enclosed locks: in every moment each thread has only one lock
- if need several locks use std::lock
- try not to call user code whith locked mutex, it can also lock its
- lock mutexes in same order in all threads
- mutex hierarchy (thread_local int curent_hierarchy)

bad:

    bool operator < (const X& lhs, const X& rhs) {
        if (&lhs == &rhs) return false; // need: prevent multilock
        std::lock_guard<std::mutex> g1(lhs.m);
        std::lock_guard<std::mutex> g2(rhs.m);
        return lhs.data < rhs.data;
    }

    X a,b;
    // thread1          // thread2
    if (a < b) ...                          
    // a.lock()
                        if (b < a) ...
                        // b.lock()
                deadlock
    // b.lock()                             // waits for thread2
                        // a.lock()         // waits for thread1

    // use std::lock(lhs.m, rhs.m);


#### std::unique_lock 
- more flexible but uses more memory, slower than lock_guard
- std::defer_lock - means mutex can be locked later
- has lock, unlock, try_lock; owns_lock() - is locked

use: deffered lock

    std::unique_lock<std::mutex> ul(m, std::defer_lock);
    ul.lock();              // same as m.lock()
    std::lock(ul1, ul2);    // can be passed to std::lock

use: passing mutex to other context (unique_lock can move):

    std::unique_lock<std::mutex> get_lock() {
        extern std::mutex m;
        std::unique_lock<std::mutex> ul(m);
        prepare_data();
        return ul;
    }

    void process_data() {
        std::unique_lock<std::mutex> ul(get_lock());
        // mutex still locked
        do_smth(); 
    }


if data is int, it can be easy copied

    int X::get_data() {
        std::lock_guard<std::mutex> g(m);
        return data;
    }

    bool operator < (const X& lhs, const X& rhs) {
        if (&lhs == &rhs) return false; // not so need
        const int l_data = lhs.get_data();
        // bad: here l_data or/and r_data can be changed
        const int r_data = rhs.get_data();
        return l.data < r.data;
    }

#### protect data on init: 
asume Data is thread safe

    std::shared_ptr<Data> data;

no protection

    void foo() {
        if (!data) {
            data.reset(new Data);
        }
        data->do_smth();
    }

forward solution: thread serialization

    void foo() {
        std::unique_lock<std::mutex> ul(m);
        if (!data) {
            data.reset(new Data);
        }
        ul.unlock();
        data->do_smth();
    }

bad: db-checked locking (old method)

    void undef_behav_dbcheck_lock() {
        if (!data) {
            std::lock_guard<std::mutex> g(m);
            if (!data) {
                data.reset(new Data);
            }
        }
        data->do_smth(); // can be called for wrong obj
    }

std::call_once, std::once_flag

    std::once_flag flag;

    void init() {
        data.reset(new Data);
    }

    void foo() {
        std::call_once(flag, init);
        data->do_smth();
    }

    // for class members:
    std::call_once(flag, &X::foo, this);


#### local static
x inits when execution goes through its declaration  
before c++11: one thread inits, other try use x before init ends, or both trying init  
c++11: ok, inited in one thread, no thread can go until init ends

    X& foo {
        static X x;
        return x;
    }

#### boost::shared_lock
frequent reads, write called rarely

    class X {
        Data data;
        mutable boost::shared_mutex sm;
        
    public:
        // can be called from many threads simultaneously
        // if no one called write
        int read() const {
            boost::shared_lock<boost::shared_mutex> sl(sm);
            return data.get_int();
        }
        
        // can lock only when no other has locked by shared_lock, or any lock
        // when locked no other thread can do read or write
        void write(int i) {
            std::lock_guard<boost::shared_mutex> sl(sm);
            data.set_int(i);
        }
    }

#### std::recursive_mutex
- 2 calls m.lock() in same thread - undefined behavior
- std::recursive_mutex can call lock() many times
- but thread must call unlock() same times as called lock()

ok:

    std::recursive_mutex rm;

    void foo() {
        std::lock_guard<std::recursive_mutex> g(rm);
        do_smth();
    }

    void bar() {
        std::lock_guard<std::recursive_mutex> g(rm);
        foo(); // rm.lock() called 2 times
        // on return rm.unlock() also called 2 times
    }


***

<a id="p3"></a>
## Part 3 - threads sync [home](#home)
forward solution

    bool flag;
    void wait_for_flag() {
        std::unique_lock<std::mutex> ul(m);
        while (!flag) {
            ul.unlock();
            // bad: what time to choose; to long or to freq ask
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
            ul.lock();
        }
    }

std::condition_variable - works with std::mutex  
std::condition_variable_any - works with any mutex type

    std::mutex m;
    std::condition_variable cond;
    std::queue<Data> data_que;

    void prepare_thread() {
        while (more_data()) {
            const Data data = prepare_data();
            std::lock_guard<std::mutex> lg(m);  // que push protection
            data_que.push(data);
            cond.notify_one();  // notify cond
        }
    }

    void process_thread() {
        while (true) {
            std::unique_lock<std::mutex> ul(m); // que protection
            cond.wait(ul, [](){ return !data_que.empty(); });
            Data data = data_que.front();
            data_que.pop();
            ul.unlock();    // que front and pop protection end
            process_data(data);
            if (end_of_data())
                break;
        }
    }

condition_variable.wait(unique_lock, func):

    if (func() == true)
        continue thread // unique_lock stay locked

    else if (func() == false)
        unique_lock.unlock();
        block thread
                                ...
                                        condition_variable.notify_one()
    unique_lock.lock();
    goto [1] or [2]

or same as:

    while (!func()) {
        ul.unlock();
        block thread until notify or spurious
        ul.lock();
    }

wake without notify - spurious wakeup (may happen many times, so func must have no side effects)


#### threadsafe_queue: try_pop, wait_and_pop
same as in prev code: prepare_thread, process_thread

    mutable std::mutex m; // mutex must be mutable for const methods

    void threadsafe_queue::push(T data) {
        std::lock_guard<std::mutex> lg(m);
        data_que.push(data);
        cond.notify_one();
    }

    void threadsafe_queue::wait_and_pop(T& data) {
        std::unique_lock<std::mutex> ul(m);
        cond.wait(ul, [this](){ return !data_que.empty(); });
        data = data_que.front();
        data_que.pop();
    }

    bool threadsafe_queue::try_pop(T& data) {
        std::lock_guard<std::mutex> lg(m);
        if (data_que.empty())
            return false;
        data = data_que.front();
        data_que.pop();
        return true;
    }

if several threads waiting condition
- notify_one - wakes one of them (any could be)
- notify_all - wakes all threads


#### std::future
- use: when need to wait only one time, and no return to that condition again
- one time event
- use: return data from thread

shared_future:
- one event - one std::future
- one event - many std::shared_future

if no data: std::future<void>

    int main() {
        // runs find_answer() in separate thread (async) or same thread (sync)
        std::future<int> ans = std::async(find_answer);
        do_smth();
        std::cout << ans.get(); // here we wait for find_answer finish
    }

std::async examlps:

    X x;
    void baz(X&);
    std::async(&X::foo, &x, 42); // x.foo(42);
    std::async(&X::foo, x, 42); // X tmpx(x); tmpx.foo(42);
    std::async(baz, std::ref(x)); // baz(x);
    std::async(X(), 42); // X tmpx(std::move(X())); tmpx.operator()(42);
    std::async(std::ref(x), 42); // x.operator()(42);

type std::launch, as 1-st param in std::async()
- std::launch::deffered   // defer launch until wait() or get()
- std::launch::async      // launch in separate thread
- deffered | async        // default: implementation desides

deffered:

    auto f = std::async(std::launch::deffered, baz, std::ref(x));
    ...
    f.wait(); // launches here baz(x);


#### std::packaged_task
abstraction layer on future

    template<>
    class std::packaged_task<int(std::string&, double)> {
    public:
        template<typename Func> explicit packaged_task(Func&& f);
        std::future<int> get_future();
        void operator()(std::string&, double);
    }

post tasks between threads:

    std::mutex m;
    std::deque<std::packaged_task<void()>> tasks;

    template<typename Func> 
    std::future<void> post_task(Func f) {
        std::packaged_task<void()> task(f);
        std::future<void> res = task.get_future();
        std::lock_guard<std::mutex> lg(m);
        tasks.push_back(std::move(task));
        return res;
    }

    void gui_thread() {
        while (!gui_shutdown()) {
            get_process_msg();
            std::packaged_task<void()> task;
            {
                std::lock_guard<std::mutex> lg(m);
                if (tasks.empty()) continue;
                task = std::move(tasks.front());
                tasks.pop_front();
            }
            task();
        }
    }

#### std::promise and std::future
- std::future - read data in receiving thread
- std::promise - set data in sending thread

promise:

    std::future<T> get_future();
    set_value(T); // future res become ready

use: when task is not simple func call, or res return from different places

several connections in one thread:

    void process_connections(Container& conns) {
        while (!done(conns)) {
            for (Connection c: conns) {
                if (c.has_in_data()) {
                    Data data = c.get_in_data();
                    std::promise<payload_t>& p = c.get_promise(data.id);
                    p.set_value(data.payload);
                }
                
                if (c.has_out_data()) {
                    Data data = c.get_out_data();
                    c.send(data.payload);
                    data.promise.set_value(true);
                }
            }
        }
    }


#### exceptions between threads
if func() throws in std::async(func)- it rethrows when get() called

    std::promise<int> p;
    try {
        p.set_value(calc_value()); // can throw
    } catch(...) {
        p.set_exception(std::current_exception());
        // or (no need try/catch mb if set in calc_value)
        // p.set_exception(std::copy_exception(std::logic_error("fuuu")));
    }

if promise or packaged_task destr without call set_value()  
in res future exception will be saved  
std::future_error; id: std::future_errc::broken_promise  
in this case of error can ignore set_value()


#### std::shared_future
- std::future is not thread safe
- only one thread can call get()
- next calls to get() return nothing
- std::future is only movable

- for several threads use std::shared_future (copyable)  
- not thread safe:  
    + bad: need to use mutex  
    + beter: access to shared async obj from individual copy of std::shared_future in each thread is safe  

    std::promise<int> p;
    std::future<int> f(p.get_future());
    assert(f.valid());

    std::shared_future<int> sf(std::move(f)); // ctor param - std::future 
    assert(!f.valid());
    assert(sf.valid());

or just

    std::shared_future<int> sf(p.get_future()); // auto move

or

    std::shared_future<int> sf = f.share(); // returns shared_future

or

    auto sf = p.get_future().share(); // template argument deduction


#### std::chrono
    std::chrono::system_clock::now() // cur sys time

clocks

    class some_clock {
        typedef time_point; // each clock has own time type
        period;             // time beat in ratio 1/sec
        bool is_steady;     // if not steady: now1 can be > now2
    };

    some_clock::time_point t = some_clock::now();

- usually system_clock not steady
- std::chrono::steady_clock - is steady
- std::chrono::high_resolution_clock - has smallest period
- system_clock - has funcs to convert to/from std::time_t

duration:  
T - int, long, double
Ratio - number of seconds in 1 interval

    std::chrono::duration<T, Ratio> 

    std::chrono::duration<short, std::ratio<60,1>> // minutes (short)
    std::chrono::duration<double, std::ratio<1,1000>> // ms (double)

typedefs std::chrono:: nanoseconds, microseconds, milliseconds, seconds, minutes, hours

    std::duration<double, std::senti> // 0.01 sec

casts:
    hours to seconds // ok
    seconds to hours // error: loose precision, use duration_cast
    seconds s(3640); // = 1h40s
    hours h = std::chrono::duration_cast<hours>(s); // ok: h = 1h
    // arth ops: +-*/
    5*seconds(1) == seconds(5);
    minutes(1) - seconds(55) == seconds(5);
    // count
    seconds(42).count() == 42;

epoch - can be since 1970, last power up
    
    std::chrono::time_point
    time_since_epoh()

    std::chrono::time_point<Clock, Duration>
    std::chrono::time_point<system_clock, minutes>

ops: +- duration (time_point of same clock)

    auto start = std::chrono::high_resolution_clock::now();
    do_smth();
    auto stop = std::chrono::high_resolution_clock::now();
    std::cout << std::chrono::duration<double, seconds>(stop-start).count();

    std::chrono::system_clock::to_time_point(time_t)



#### wait_for, wait_until
- wait_for() - wait for 10 milliseconds
- wait_until() - wait until 17:01:35 2007 September 28

example:

    std::future<int> f = std::async(some_task);
    if (f.wait_for(std::chrono::milliseconds(10)) 
            == std::future_status::ready)
    {
        process(f.get());
    }

future_status:
- std::future_status::ready       // res ready
- std::future_status::timeout     // time ended
- std::future_status::deffered    // task is deffered

example:

    bool wait_loop() {
        auto const t = std::chrono::steady_clock::now()
            + std::chrono::milliseconds(500);
        std::unique_lock<std::mutex> ul(m);
        while (!done) {
            // if use wait_for spurious wakeup make this loop inf
            if (cond.wait_until(lk,t)==std::cv_status::timeout)
                break;
        }
        return done;
    }

    // sleep
    std::this_thread::sleep_for();
    std::this_thread::sleep_until();

    // mutex with timeout
    std::timed_mutex
    std::recursive_timed_mutex
    std::unique_lock
    // methods: returns true if locked
    try_lock_for()
    try_lock_until()
    // condition_variable
    // returns: std::cv_status::timeout/no_timeout
    wait_for()  
    wait_until()


#### sync example
- functional programing: func not depend on out state
- 2 calls with same args returns same value
- clear func: not modify outer state
- reenterable func
- Haskel, CSP, Erlang, MPI

quick_sort:

    template<typename T>
    std::list<T> sequent_quick_sort(std::list<T> in) {
        if (in.empty())
            return in;
        
        // move 1st el to res
        std::list<T> res;
        res.splice(res.begin(), in, in.begin()); 
        
        // partition
        const T& pivot = *res.begin();
        auto div_point = std::partition(in.begin(), in.end()
            , [&](const T& t){ return t < pivot; }); 
        
        // move [in.begin(), div_point] to lower
        std::list<T> lower; 
        lower.splice(lower.end(), in, in.begin(), div_point);
        
        // recursive sort both parts
        auto new_lower(sequent_quick_sort(std::move(lower)));
        auto new_higher(sequent_quick_sort(std::move(in)));
        
        // merge them
        res.splice(res.end(), new_higher);
        res.splice(res.begin(), new_lower);
        
        return res;
    }


    template<typename T>
    std::list<T> parall_quick_sort(std::list<T> in) {
        if (in.empty())
            return in;
        
        // move 1st el to res
        std::list<T> res;
        res.splice(res.begin(), in, in.begin()); 
        
        // partition
        const T& pivot = *res.begin();
        auto div_point = std::partition(in.begin(), in.end()
            , [&](const T& t){ return t < pivot; }); 
        
        // move [in.begin(), div_point] to lower
        std::list<T> lower; 
        lower.splice(lower.end(), in, in.begin(), div_point);
        
        // recursive sort both parts
        //WAS: auto new_lower(sequent_quick_sort(std::move(lower)));
        // instead we run it async
        std::future<std::list<T>> new_lower(
            std::async(&parall_quick_sort<T>, std::move(lower));
        auto new_higher(parall_quick_sort(std::move(in)));
        
        // merge them
        res.splice(res.end(), new_higher);
        res.splice(res.begin(), new_lower.get());// get async res
        // if too many threads - this will run syn
        
        return res;
    }

instead of use std::async can write own (thread_pool see [Part 9])

    template<typename F, typename A>
    std::future<std::result_of<F(A&&)>::type>
    spawn_task(F&& f, A&& a) {
        typedef std::result_of<F(A&&)>::type res_t;
        std::packaged_task<res_t(A&&)> task(std::move(f));
        std::future<res_t> res(task.get_future());
        std::thread t(std::move(task), std::move(a));
        t.detach();
        return res;
    }


***

<a id="p4"></a>
## Part 4 - std::atomic [home](#home)
obj modification order: in what order threads are modifying obj

    is_lock_free()

    std::atomic_flag // guaranted lockfree
    test_and_set()
    clear()

#### std::atomic<>
- noncopyable, nocopyconstructable
- do not mix std::atomic<T> and atomic_T in code: they can differ in diff environments

types:
- atomic_bool // std::atomic<bool>
- _char, _schar, _uchar, _int, _uint ...
- _size_t, _ptrdiff_t, ...

methods in std::atomic<>:
+ basic ops
    - load()      // get
    - store()     // set
    - exchange()  // get-mod-set
+ get-if-mod-set: returns true if mod
    - compare_exchange_weak()
    - compare_exchange_strong()
+ in specs (methods return previous value)
    - +=, -=, *=, |=, ...
    - ++, -- // for int types
    - fetch_add(), fetch_sub() // same as ++,--


params for memory access order: memory_order_
- default: _seq_cst
- set: _relaxed, _release, _seq_cst
- get: _relaxed, _consume, _acquire, _seq_cst
- get-mod-set: _relaxed, _consume, _acquire, _release, _acq_rel, _seq_cst


#### std::atomic_flag
atomic_flag not used directly, but used as building block for all other types

    std::atomic_flag f = ATOMIC_FLAG_INIT; // only this way: init reset 
    f.clear(_release);
    bool x = f.test_and_set();  // _seq_cst

spinlock_mutex works as mutex but bad in races:

    class spinlock_mutex {
        std::atomic_flag flag;
    public:
        spinlock_mutex(): flag(ATOMIC_FLAG_INIT) {}
        void lock() { while(flag.test_and_set(_acquire)); }
        void unlock() { flag.clear(_release); }
    };

#### std::atomic<bool>
    std::atomic<bool> b(true);
    b = false; // operator=() returns T (not T&)
    bool x = b.load(_acquire);          // get
    b.store(true);                      // set
    x = b.exchange(false, _acq_rel);    // get-mod-set

at.compare_exchange_weak(expect,new_value):

    bool res = (at == expect);
    if (res) at = new_value;
    else     expect = at; 
    return res;

set can fail (if no such hardware atomic op) - false fail; fix in case of false fail:

    bool expect = false;
    extern atomic<bool> b;
    while (!b.compare_exchange_weak(expect, true) && !expect);

compare_exchange_strong() - guaranted not fail, so no loop needed (it's inside)
- if expected - easy to calc use _weak
- else _strong (this diff can be visible)

they has 2 memory_order_ params: changed and fail


#### std::atomic<T*>
ops: +=, -=, ++, --

    X arr[5];
    std::atomic<X*> p(arr);
    X* x = p.fetch_add(2); // returns previous
    assert(x == &arr[0]);
    assert(p.load() == &arr[2]);
    x = (p-=1);
    assert(x == &arr[1]);
    assert(p.load() == &arr[1]);
    p.fetch_add(3, _release);

#### std::atomic<int>
- fetch_add, fetch_sub, fetch_and, fetch_or, fetch_xor // ret old val
- +=, -=, &=, |=, ^= // ret new val
- ++, --

#### std::atomic<T>
T must have:
- default operator=()
- no virtual funcs
- no virtual base class
- can be comparable bit by bit
- same for base class and members
- so compiler can use memcpy(), memcmp()

std::atomic<float> can fail in compare_exchange_strong(), as it compares float

bad: std::atomic<std::vector<int>>


#### nonmember funcs

    std::atomic_load(std::atomic<T>*)
    std::atomic_is_lock_free()

with _explicit for memory_order_

    std::atomic_store(&atomic_var, new_value);
    std::atomic_store_explicit(&atomic_var, new_value, _release);

for std::atomic_flag, _explicit

    std::atomic_flag_test_and_set()
    std::atomic_flag_clear()

for std::shared_ptr

    std::shared_ptr<Data> p;

    void process_global() {
        std::shared_ptr<Data> local = std::atomic_load(&p);
        process_data(local);
    }

    void update_global() {
        std::shared_ptr<Data> local(new Data);
        std::atomic_store(&p, local);
    }


#### ops sync, FORCE-ORDER
write happen before read - FORCE-ORDER

    std::vector<int> data;
    std::atomic<bool> ready(false);

    void reader() {
        while (!ready.load()) // ready get HAPPENS-BEFORE get data
            std::this_thread::sleep(std::milliseconds(1));
        std::cout << data[0]; // when ready writer SYNC-WITH this read
    }

    void writer() {
        data.push_back(42); // data set HAPPENS-BEFORE set ready
        ready = true;
    }

SYNC-WITH - relation only between atomic ops  
real source of sync always atomic ops  
if thread1 write data, and thread2 read data  
then exists relation SYNC-WITH between store in 1 and load in 2  

HAPPENS-BEFORE - order of ops in program  
it determine what ops see effects of other ops and which exactly  
order between threads  
so: if sequence of mod data hapens in thread1  
need only 1 SYNC-WITH for ops in thread2 to see changes

memory_order_: 6 params  
3 models relaxed, sequent, acqure-release

#### sequent: _seq_cst (default)
- all threads see same ops order, all ops - ordered
- sequent store SYNC-WITH sequent load of same var
- any sequent op after this load seen from any thread 
- as folowed after this load, but not for relaxed thread models
- so sequent model must be used in all threads
- this gives it simple but heavy model, as need to sync between all threads

example:

    std::atomic<bool> x(false), y(false);
    std::atomic<int> z(0);

    void write_x() { x.store(true, _seq_cst); }
    void write_y() { y.store(true, _seq_cst); }

    void read_x_then_y() {
        while (!x.load(_seq_cst));
        if (y.load(_seq_cst)) ++z;
    }

    void read_y_then_x() {
        while (!y.load(_seq_cst));
        if (x.load(_seq_cst)) ++z;
    }

    int main() {
        std::thread a(write_x); 
        std::thread b(write_y);
        std::thread c(read_x_then_y); 
        std::thread d(read_y_then_x);
        a.join(); b.join(); c.join(); d.join();
        assert(z.load() != 0); // ok
    }

- first must happen store x or y
- if load y in read_x_then_y false
- then store x must happen before store y
- in that case load x in read_y_then_x must be true
- because while loop guarantee that in that point y == true
- so any thread must see first x==true then y==true order
- this can be if store x must happen before store y

#### nonsequent models
in case of no other restrictions - single requirement is:  
**all threads agree on mod order of each single var**

- threads do not need to sync ops order between each other
- there is no single gloabal ops order
- ops can happen really simultaneously
- compiler and processor can change ops order
- threads internal buffs can have diff val for same obj
- threads can see diff ops order, in case of additional restrictions



#### relaxed: _relaxed
- relaxed ops do not participate in SYNC-WITH relation
- ops on same var in one thread has HAPPENS-BEFORE relation
- but in diff threads there is no restrictions
- [!!!] mod order of each single var works
- "man in a box with a list"

example:

    void write_x_then_y() { 
        x.store(true, _relaxed); 
        y.store(true, _relaxed); 
    }

    void read_y_then_x() {
        while (!y.load(_relaxed));
        if (x.load(_relaxed)) ++z;
    }

    int main() {
        std::thread a(write_x_then_y);
        std::thread b(read_y_then_x);
        a.join(); b.join();
        assert(z.load() != 0); // can assert
    }

exists HAPPENS-BEFORE separate between loads and stores, but no relation between any load and any store

    write_x_then_y              read_y_then_x

    x.store(true, _relaxed)
    y.store(true, _relaxed)     y.load(_relaxed) -> true
                                x.load(_relaxed) -> false


#### acqure-release: _acquire, _release, _acq_rel
- load:        _acquire
- store:       _release
- get-mod-set: _acquire, _release, _acq_rel
- sync between pairs: _acquire _release
- _release SYNC-WITH _acquire when read value 

example:

    void write_x() { x.store(true, _release); }
    void write_y() { y.store(true, _release); }

    void read_x_then_y() {
        while (!x.load(_acquire));
        if (y.load(_acquire)) ++z;
    }

    void read_y_then_x() {
        while (!y.load(_acquire));
        if (x.load(_acquire)) ++z;
    }

    assert(z.load() != 0); // can assert

read_x_then_y and read_y_then_x can see diff vals, because there is no HAPPENS-BEFORE relation

    write_x         read_x_then_y       read_y_then_x       write_y    
                                                            
    x.store(true                                            y.store(true
    , _release)                                             , _release)
                    x.load(_acquire)    y.load(_acquire)
                    -> true             -> true
                                        
                    y.load(_acquire)    x.load(_acquire)
                    -> false            -> false


    void write_x_then_y() { 
        x.store(true, _relaxed); 
        y.store(true, _release); 
    }

    void read_y_then_x() {
        while (!y.load(_acquire));
        if (x.load(_relaxed)) ++z;
    }

    assert(z.load() != 0); // ok

- store x HAPPENS-BEFORE store y (they are in one thread)
- store y SYNC-WITH load y
- and store x HAPPENS-BEFORE load y 
- and store x HAPPENS-BEFORE load x
- so ops on x become ordered 

transitive
- transitive sync: A sync B, B sync C -> A sync C
- thread3 load z become SYNC-WITH thread1 store z
- although thread2 alls ops fo x and y

its enough to sync thread1 and thread3:

    void thread1() {
        z.store(42, _relaxed);      // HAPPENS-BEFORE store x
        x.store(true, _release);
    }

    void thread2() {
        while (!x.load(_acquire));  // SYNC-WITH thread1 store x
        y.store(true, _release);
    }

    void thread3() {
        while (!y.load(_acquire));  // SYNC-WITH thread2 store y
        assert(z.load(_relaxed) == 42); // ok
    }

we can combine x and y using compare_exchange_strong: need both _acquire and _release: _acq_rel

    std::atomic<int> sync(0);

    void thread1() {
        ...
        sync.store(1, _release);
    }

    void thread2() {
        int expect = 1;
        while (!sync.compare_exchange_strong(expect,2,_acq_rel))
            expect = 1;
    }

    void thread3() {
        while (sync.load(_acquire) < 2);
        ...
    }

#### _consume
relations:
- CARRIES-DEPENDENCY-TO - in same thread
- if res of op A used in B
- if A CARRIES-DEPENDENCY-TO B, and B CARRIES-DEPENDENCY-TO C
- then A CARRIES-DEPENDENCY-TO C
- DEPENDENCY-ORDER-BEFORE - between threads
- store with _release, _acq_rel, _seq_cst BEFORE load(_consume)

used for T*:

    struct X { int i; };

    std::atomic<X*> p;
    std::atomic<int> a;

    void create_x() {
        X* x = new X;
        x->i = 42;
        a.store(99, _relaxed);
        p.store(x, _release);
    }

    void use_x() {
        X* x;
        while (!(x = p.load(_consume)))
            std::this_thread::sleep(milliseconds(1));
        assert(x->i == 42); // ok: depends on p
        assert(a == 99);    // can assert: not depend on p (_relaxed)
    }

    int main() {
        std::thread t1(create_x);
        std::thread t2(use_x);
        t1.join(); t2.join();
    }


#### std::kill_dependency()
break dependency order - when need compiler can cash var, change order for optimizm

    int global_data[] = {...}; // for read only
    std::atomic<int> index;
    void foo() {
        int i = index.load(_consume);
        process(global_data[std::kill_dependency(i)]);
    }


#### RELEASE-SEQUENCE
- if store: _release, _acq_rel, _seq_cst
-  && load: _consume, _acquire, _seq_cst
-  && every op in sequence load val stored by prev
- then first store SYNC-WITH (or DEPENDENCY-ORDER-BEFORE if _consume)
- with last load
- and any get-mod-set in sequence can be any order (even _relaxed)
- it is only for get-mod-set

other:
- if one thread then all ok: store _release SYNC-WITH load _acquire
- with 2 threads - they are not in sync: data race
- but 1st call fetch_sub is in RELEASE-SEQUENCE
- and store SYNC-WITH 2nd cal fetch_sub
- but 2 consumers are not in SYNC-WITH

example:

    std::vector<int> data;
    std::atomic<int> count;

    void populate() {
        int num = 20;
        data.clear();
        for (int i=0; i<num; ++i)
            data.push_back(i);
        count.store(num, _release); // 1st store
    }

    void consume() {
        while (true) {
            int i;
            if ((i = count.fetch_sub(1, _acquire)) <= 0) {
                wait_for_more();
                continue;
            }
            process(data[i - 1]);
        }
    }

    int main() {
        std::thread a(populate);
        std::thread b(consume);
        std::thread c(consume);
        a.join(); b.join(); c.join();
    }

fetch_sub in thread1 and thread2 HAPPENS-BEFORE store  
RELEASE-SEQUENCE: store -> fetch_sub 1 -> fetch_sub 2

    populate    thread1         thread2

    count.store
    _release
                count.fetch_sub                
                _acquire
                    
                process         count.fetch_sub 
                                _acquire
                                    
                                process

#### bariers (fence)
additional restrictions, usually for read without mod with _relaxed
- fence _release [2] SYNC-WITH fence _acquire [5]
- because load y [4] reads stored  [3]
- so store x [1] HAPPENS-BEFORE load x [6]
- without fence store x [1] and load x [6] not ordered

same as if store y _release and load y _acquire
- if after fence _release there is store
- if before fence _acquire there is load
- then fences are in SYNC-WITH each other
- fence is a sync point

example:

    void write_x_then_y() { 
        x.store(true, _relaxed);            [1]
        std::atomic_thread_fence(_release); [2]
        y.store(true, _relaxed);            [3]
    }

    void read_y_then_x() {
        while (!y.load(_relaxed));          [4]
        std::atomic_thread_fence(_acquire); [5]
        if (x.load(_relaxed)) ++z;          [6]
    }

    assert(z.load() != 0); // ok

bad: ops are not ordered, fence must be between ops

    void write_x_then_y() { 
        std::atomic_thread_fence(_release);
        x.store(true, _relaxed);
        y.store(true, _relaxed);
    }


#### order nonatomics with atomic
- same as was with T*: assert(x->i == 42);
- same works mutexes and spinlock_mutex

example:

    bool x = false;
    std::atomic<bool> y(false);
    std::atomic<int> z(0);

    void write_x_then_y() { 
        x = true;
        std::atomic_thread_fence(_release);
        y.store(true, _relaxed);           
    }

    void read_y_then_x() {
        while (!y.load(_relaxed));         
        std::atomic_thread_fence(_acquire);
        if (x) ++z;         
    }

    assert(z.load() != 0); // ok


***

<a id="p5"></a>
## Part 5 - structs with locks [home](#home)
- invariant persist from any thread
- no races
- exception safe (lock mutex can throw)
- no deadlock, min code under lock, min included locks

to think:
- serialization: if use mutex - not real parallelism, only one thread at a time has access to data
- granularity: min code under lock, diff parts has own mutexes

#### stack with blocks - all methods use lock_guard
    class threadsafe_stack {
        std::stack<T> data;
        mutable std::mutex m;
        
    public:
        threadsafe_stack(const threadsafe_stack& other) {
            std::lock_guard<std::mutex> g(other.m);
            data = other.data;
        }
        
        std::shared_ptr<T> pop() {
            std::lock_guard<std::mutex> g(m);
            if (data.empty())
                throw empty_stack();
            std::shared_ptr<T> const res(std::make_shared<T>(
                std::move(data.top())));
            data.pop();
            return res;
        }
        
        ...
    };

#### queue with condition_variable
    class threadsafe_queue {
        std::queue<T> data;
        mutable std::mutex m;
        std::condition_variable cond;
        
    public:
        void push(T val) {
            std::lock_guard<std::mutex> g(m);
            data.push(std::move(val));
            cond.notify_one();
        }
        
        std::shared_ptr<T> wait_and_pop() {
            std::unique_lock<std::mutex> ul(m);
            cond.wait(ul, [this](){ return !data.empty(); });
            std::shared_ptr<T> const res(std::make_shared<T>(
                std::move(data.front())));
            data.pop();
            return res;
        }
        
        ...
    };

#### beter exception safe:
    class threadsafe_queue {
        std::queue<std::shared_ptr<T>> data;
    
in wait_and_pop:

    std::shared_ptr<T> res = data.front();

push:

    void push(T val) {
        std::shared_ptr<T> res(std::make_shared<T>(
            std::move(val))); // not under lock: beter performance
        std::lock_guard<std::mutex> g(m);
        data.push(res);
        cond.notify_one();
    }

#### less granularity locks
single thread queue using unique_ptr makes safe auto delete elements

    template<typename T>
    class queue {
        struct node {
            T data; // std::shared_ptr<T> data;
            std::unique_ptr<node> next;
            node(T val): data(std::move(val)) {}
        };
        
        std::unique_ptr<node> head;
        node* tail;
        
    public:
        queue() {} // : head(new node), tail(head.get())
        queue(const queue&) = delete;
        queue& operator=(const queue&) = delete;
        
        std::shared_ptr<T> try_pop() {
            if (!head) // if (head.get() == tail)
                return std::shared_ptr<T>();
            std::shared_ptr<T> res(std::make_shared<T>(
                std::move(head->data)));
            // std::shared_ptr<T> const res(head->data);
            std::unique_ptr<node> const old_head = std::move(head);
            head = std::move(old_head->next);
            return res;
        }
        
        void push(T val) {
            std::unique_ptr<node> p(new node(std::move(val)));
            // tail->data = new_data;
            node* const new_tail = p.get();
            if (tail) // always true
                tail->next = std::move(p);
            else
                head = std::move(p);
            tail = new_tail;
        }
    };

2nd step: 
- std::shared_ptr<T> data;
- there is always 1 empty element (after last)
- in push no calls to head
- in push and pop: no ops on same node

code:

    template<typename T>
    class queue {
        struct node {
            std::shared_ptr<T> data;
            std::unique_ptr<node> next;
        };
        
        std::unique_ptr<node> head;
        node* tail;
        
    public:
        queue(): head(new node), tail(head.get()) {}
        queue(const queue&) = delete;
        queue& operator=(const queue&) = delete;
        
        std::shared_ptr<T> try_pop() {
            if (head.get() == tail)
                return std::shared_ptr<T>();
            std::shared_ptr<T> const res(head->data);
            std::unique_ptr<node> const old_head = std::move(head);
            head = std::move(old_head->next);
            return res;
        }
        
        void push(T val) {
            std::shared_ptr<T> new_data(std::make_shared<T>(
                std::move(val)));
            std::unique_ptr<node> p(new node);
            tail->data = new_data;
            node* const new_tail = p.get();
            tail->next = std::move(p);
            tail = new_tail;
        }
    };

#### threadsafe_queue
- mutex lock can throw, but by the time of lock data not changed
- so pop is ok; in push there created node and T on heap
- but smart ptrs hold them so in case of throw they delete correctly
- in pop head 2 mutexes locked, but always in same order so no race

code:

    template<typename T>
    class threadsafe_queue {
        struct node {
            std::shared_ptr<T> data;
            std::unique_ptr<node> next;
        };
        
        std::unique_ptr<node> head;
        node* tail;
        
        std::mutex head_mutex;
        std::mutex tail_mutex;
        
        node* get_tail() {
            std::lock_guard<std::mutex> tail_lock(tail_mutex);
            return tail;
        }
        
        std::unique_ptr<node> pop_head() {
            std::lock_guard<std::mutex> head_lock(head_mutex);
            if (head.get() == get_tail())
                return nullptr;
            std::unique_ptr<node> const old_head = std::move(head);
            head = std::move(old_head->next);
            return old_head;
        }
        
    public:
        queue(): head(new node), tail(head.get()) {}
        queue(const queue&) = delete;
        queue& operator=(const queue&) = delete;
        
        std::shared_ptr<T> try_pop() {
            std::unique_ptr<node> old_head = pop_head();
            return old_head ? old_head->data : std::shared_ptr<T>();
        }
        
        void push(T val) {
            std::shared_ptr<T> new_data(std::make_shared<T>(
                std::move(val)));
            std::unique_ptr<node> p(new node);
            node* const new_tail = p.get();
            std::lock_guard<std::mutex> tail_lock(tail_mutex);
            tail->data = new_data;
            tail->next = std::move(p);
            tail = new_tail;
        }
    };

bad:

    std::unique_ptr<node> pop_head() {
        node* const old_tail = get_tail(); // not protected by head_mutex
        // here and head and tail can change so old_tail will be invalid
        // it is ok for only tail to change
        std::lock_guard<std::mutex> head_lock(head_mutex);
        if (head.get() == old_tail)
            return nullptr;
        std::unique_ptr<node> const old_head = std::move(head);
        head = std::move(old_head->next);
        return old_head;
    }

#### add condition_variable for wait_and_pop(T& val)
    void push(T val) {
        std::shared_ptr<T> new_data(std::make_shared<T>(
            std::move(val)));
        std::unique_ptr<node> p(new node);
        {
            std::lock_guard<std::mutex> tail_lock(tail_mutex);
            tail->data = new_data;
            node* const new_tail = p.get();
            tail->next = std::move(p);
            tail = new_tail;
        }
        cond.notify_one();
    }

    {
    private:
        std::unique_ptr<node> pop_head() {
            std::unique_ptr<node> const old_head = std::move(head);
            head = std::move(old_head->next);
            return old_head;
        }

        std::unique_lock<std::mutex> wait_for_data() {
            std::unique_lock<std::mutex> head_lock(head_mutex);
            cond.wait(head_lock, [&](){ return head.get() != get_tail(); });
            return std::move(head_lock);
        }

        std::unique_ptr<node> wait_pop_head() {
            std::unique_lock<std::mutex> head_lock(wait_for_data());
            return pop_head();
        }

        std::unique_ptr<node> wait_pop_head(T& val) {
            std::unique_lock<std::mutex> head_lock(wait_for_data());
            val = std::move(*head->data);
            return pop_head();
        }
        
    public:
        std::shared_ptr<T> wait_and_pop() {
            std::unique_ptr<node> const old_head = wait_pop_head();
            return old_head->data;
        }
        
        void wait_and_pop(T& val) {
            std::unique_ptr<node> const old_head = wait_pop_head(val);
        }
    }

    // try_pop
    {
    private:
        std::unique_ptr<node> try_pop_head() { // for: (T& val)
            std::lock_guard<std::mutex> head_lock(wait_for_data());
            if (head.get() == get_tail())
                return std::unique_ptr<node>();
            // for: val = std::move(*head->data);
            return pop_head();
        }
        
    public:
        std::shared_ptr<T> try_pop() {
            std::unique_ptr<node> old_head = try_pop_head();
            return old_head ? old_head->data : std::shared_ptr<T>();
        }
        
        bool try_pop(T& val) {
            std::unique_ptr<node> const old_head = try_pop_head(val);
            return old_head;
        }
        
        bool empty() {
            std::lock_guard<std::mutex> head_lock(head_mutex);
            return (head.get() == get_tail());
        }
    }


#### thread safe map
many reads, rare write  
ops: add, mod, del, get

map inside: rb-tree, sorted arr, hash-table
- hash-table: works good if num_buckets is prime
- num_buckets is const so no need for mutex for get_bucket
- granularity: each bucket has own mutex, so parallelism inc by num_buckets times
- many thread can read but only one can mod bucket

code:

    template<typename Key, typename Value, typename Hash = std::hash<Key>>
    class threadsafe_map {
        class bucket_t {
            typedef std::pair<Key,Value> bucket_value;
            typedef std::list<bucket_value> bucket_data;
            typedef typename bucket_data::iterator bucket_iter;
            
            bucket_data data;
            mutable boost::shared_mutex m;
            
            bucket_iter find(Key const& key) const {
                return std::find_if(data.begin(), data.end(),
                    [&](bucket_value const& item)
                    {return item.first==key;});
            }
        
        public:
            Value value(Key const& key, Value const& def_val) const {
                boost::shared_lock<boost::shared_mutex> lock(m);
                bucket_iter const found = find(key);
                return found == data.end() ? def_val : found->second;
            }
            
            void add_or_mod(Key const& key, Value const& val) {
                std::unique_lock<boost::shared_mutex> lock(m);
                bucket_iter const found = find(key);
                if (found == data.end())
                    data.push_back(bucket_value(key,val));
                else
                    found->second = val;
            }
            
            void remove(Key const& key) {
                std::unique_lock<boost::shared_mutex> lock(m);
                bucket_iter const found = find(key);
                if (found != data.end())
                    data.erase(found);
            }
        };
        
        std::vector<std::unique_ptr<bucket_t>> buckets;
        Hash hash;
        
        bucket_t& get_bucket(Key const& key) {
            return *buckets[hash(key) % buckets.size()];
        }
        
    public:
        threadsafe_map(unsigned num_buckets=19, Hash const& h = Hash())
            : buckets(num_buckets), hash(h)
        {
            for (unsigned i=0; i<num_buckets; ++i)
                buckets[i].reset(new bucket_t);
        }
        
        threadsafe_map(threadsafe_map const&) = delete;
        threadsafe_map& operator=(threadsafe_map const&) = delete;
        
        Value value(Key const& key, Value const& def_val = Value()) const {
            return get_bucket(key).value(key,def_val);
        }
        
        void add_or_mod(Key const& key, Value const& val) {
            return get_bucket(key).add_or_mod(key,val);
        }
        void remove(Key const& key) {
            return get_bucket(key).remove(key);
        }
        
        std::map<Key,Value> get_map() const {
            std::vector<std::unique_lock<boost::shared_mutex>> locks;
            for (unsigned i=0; i<buckets.size(); ++i)
                locks.push_back(std::unique_lock<boost::shared_mutex>(
                    buckets[i].m));
            
            std::map<Key,Value> res;
            for (unsigned i=0; i<buckets.size(); ++i)
                for (bucket_iter it=buckets[i].data.begin();
                        it != buckets[i].data.end(); ++it)
                    res.insert(*it);
            
            return res;
        }
    };


#### threadsafe_list
example:  
node: head -> 1st -> 2nd -> null  
data: null    123    456    

    class node {
        std::mutex m;
        std::shared_ptr<T> data
        std::unique_ptr<node> next;
    };

    template<typename Func>
    void threadsafe_list::for_each(Func f) {
        node* cur = &head;
        std::unique_lock<std::mutex> cur_lk(head.m);
        while (node* const next = cur->next.get()) {
            std::unique_lock<std::mutex> next_lk(next->m);
            cur_lk.unlock();
            f(*next->data);
            cur = next;
            cur_lk = std::move(next_lk);
        }
    }


***

<a id="p6"></a>
## Part 6 - lockfree structs [home](#home)
- lockfree - every op ends in several steps
- blocking: mutex, future, condition
- spinlock_mutex - nonblocking but not lockfree
- starvation: long time while(flag.test_and_set)
- in lockfree struct some thread move forward on every step
- in waitfree - all threads move forward
- no deadlock but livelock: 2 threads try mod but all must start from begin

#### lockfree_stack
    class lockfree_stack {
        struct node {
            T data;
            node* next;
        };

        std::atomic<node*> head;
        
    public:
        void push(T const& data) {
            node* const new_node = new node(data);
            new_node->next = head.load();
            while(!head.compare_exchange_weak(new_node->next, new_node));
        }
    };

#### 1. try
- if head==null: old_head->next fails
- if operator= throws we loose data

pop:

    void pop(T& res) {
        node* old_head = head.load();
        while(!head.compare_exchange_weak(old_head, old_head->next));
        res = old_head->data;
    }

#### 2. std::shared_ptr<T> data;
with mem leaks: node* next; node* old_head;

    struct node {
        std::shared_ptr<T> data;
        node* next;
        node(T const& d): data(std::make_shared<T>(data)) {}
    };
        
    std::shared_ptr<T> pop() {
        node* old_head = head.load();
        while(old_head && !head.compare_exchange_weak(
            old_head, old_head->next));
        return old_head ? old_head->data : std::shared_ptr<T>();
    }


#### 3. fix mem leaks: sort of garbage-collector
if high load to_del list will grow to inf  
count num threads in pop

    class lockfree_stack {
        std::atomic<int> threads_in_pop;
        std::atomic<node*> to_del;
        
        static void del_nodes(node* nodes) {
            while (nodes) {
                node* next = nodes->next;
                delete nodes;
                nodes = next;
            }
        }
        
        void try_reclaim(node* old_head) {
            if (threads_in_pop==1) { // last thread - can del nodes
                // if here other thread call pop(): --threads_in_pop != 0
                node* nodes_to_del = to_del.exchange(nullptr);
                if (!--threads_in_pop)
                    del_nodes(nodes_to_del);
                else if (nodes_to_del)
                    chain_pending_many(nodes_to_del);
                delete old_head;
            } else { // add to del list
                chain_pending_one(old_head);
                --threads_in_pop;
            }
        }
        
        void chain_pending(node* first, node* last) {
            last->next = to_del;
            while (!to_del.compare_exchange_weak(last->next, first));
        }
        
        void chain_pending_many(node* nodes) {
            node* last = nodes;
            while (node* const next = last->next)
                last = next;
            chain_pending(nodes, last);
        }
        
        void chain_pending_one(node* n) {
            chain_pending(n,n);
        }
        
    public:
        std::shared_ptr<T> pop() {
            ++threads_in_pop;
            node* old_head = head.load();
            while(old_head && !head.compare_exchange_weak(
                old_head, old_head->next));
            std::shared_ptr<T> res;
            if (old_head)
                res.swap(old_head->data);
            try_reclaim(old_head); // !!!
            return res;
        }
    };


#### 4. hazard pointers
    std::shared_ptr<T> pop() {
        std::atomic<void*>& hp = get_hazard_ptr_cur_thread();
        node* old_head = head.load();
        do {
            node* tmp;
            do {
                tmp = old_head;
                hp.store(old_head);
                old_head = head.load();
            } while (old_head != tmp);
        } while(old_head && !head.compare_exchange_strong(
                old_head, old_head->next));
        hp.store(nullptr);
        std::shared_ptr<T> res;
        if (old_head) {
            res.swap(old_head->data);
            if (outstanding_hazard_ptr(old_head))
                reclaim_later(old_head);
            else
                delete old_head;
            del_nodes_no_hazard();
        }
        return res;
    }


    unsigned const max_hazard_ptrs = 100;
    struct hazard_ptr {
        std::atomic<std::thread::id> id;
        std::atomic<void*> ptr;
    };
    hazard_ptr hazard_ptrs[max_hazard_ptrs];

    class hp_owner {
        hazard_ptr* hp;
    public:
        hp_owner(hp_owner const&) = delete;
        hp_owner& operator=(hp_owner const&) = delete;
        
        hp_owner(): hp(nullptr) {
            for (unsigned i=0; i<max_hazard_ptrs; ++i) {
                std::thread::id old_id;
                if (hazard_pointers[i].id.compare_exchange_strong(
                        old_id, std::this_thread::get_id())) {
                    hp = &hazard_pointers[i];
                    break;
                }
            }
            if (!hp)
                throw std::runtime_error("no hazard pointers avail");
        }
        
        std::atomic<void*>& get_pointer() {
            return hp->ptr;
        }
        
        ~hp_owner() {
            hp->ptr.store(nullptr);
            hp->id.store(std::thread::id());
        }
    };

    std::atomic<void*>& get_hazard_ptr_cur_thread() {
        thread_local static hp_owner hazard;
        return hazard->get_pointer();
    }

    bool outstanding_hazard_ptr(void* p) {
        for (unsigned i=0; i<max_hazard_ptrs; ++i) {
            if (hazard_pointers[i].ptr.load() == p)
                return true;
        }
        return false;
    }


#### 5. ref counted
if std::atomic_is_lock_free(&some_shared_ptr)

    template<typename T>
    class lockfree_stack {
        struct node {
            std::shared_ptr<T> data;
            std::shared_ptr<node> next;
            node(T const& d): data(std::make_shared<T>(d)) {}
        };
        
        std::shared_ptr<node> head;
        
    public:
        void push(T const& data) {
            std::shared_ptr<node> const new_node 
                = std::make_shared<node>(data);
            new_node->next = head.load();
            while(!std::atomic_compare_exchange_weak(&head
                , &new_node->next, new_node));
        }
        
        std::shared_ptr<T> pop() {
            std::shared_ptr<node> old_head = std::atomic_load(&head);
            while(old_head && !std::atomic_compare_exchange_weak(
                &old_head, old_head->next));
            return old_head ? old_head->data : std::shared_ptr<T>();
        }
    };

#### 6. if shared_ptr not lockfree
using internal and external counters by hand (if type too large atomic uses mutex)

    template<typename T>
    class lockfree_stack {
        struct node;
        
        struct counted_node_ptr {
            int external_count;
            node* ptr;
        };
        
        struct node {
            std::shared_ptr<T> data;
            std::atomic<int> internal_count;
            counted_node_ptr next;
            node(T const& d): data(std::make_shared<T>(d))
                , internal_count(0) {}
        };
        
        void inc_head_count(counted_node_ptr& old_cnt) {
            counted_node_ptr new_cnt;
            do {
                new_cnt = old_cnt;
                ++new_cnt.external_count;
            } while (!head.compare_exchange_strong(
                old_cnt, new_cnt
                , _acquire, _relaxed)); // RELAX
            old_cnt.external_count = new_cnt.external_count;
        }
        
        std::atomic<counted_node_ptr> head;
        
    public:
        ~lockfree_stack() {
            while (pop());
        }
        
        void push(T const& data) {
            counted_node_ptr new_node;
            new_node.ptr = new node(data);
            new_node.external_count = 1;
            new_node.ptr->next = head.load(_relaxed); // RELAX
            while(!head.compare_exchange_weak(
                new_node.ptr->next, new_node
                , _release, _relaxed)); // RELAX
        }
        
        std::shared_ptr<T> pop() {
            counted_node_ptr old_head = head.load(_relaxed); // RELAX
            for (;;) {
                inc_head_count(old_head);
                node* const ptr = old_head.ptr;
                if (!ptr)
                    return std::shared_ptr<T>();
                    
                if (head.compare_exchange_strong(old_head, ptr->next
                        , _relaxed)) { // RELAX
                    std::shared_ptr<T> res;
                    res.swap(ptr->data);
                    int const count_inc = old_head.external_count - 2;
                    if (ptr->internal_count.fetch_add(count_inc
                            , _release) // RELAX
                            == -count_inc)
                        delete ptr;
                    
                    return res;
                    
                } else if (ptr->internal_count.fetch_add(-1
                        , _relaxed) // RELAX
                        == 1) {
                    ptr->internal_count.load(_acquire) // RELAX
                    delete ptr;
                }
            }
        }
    };


#### lockfree_queue
    template<typename T>
    class lockfree_queue {
        struct node {
            std::shared_ptr<T> data;
            node* next = nullptr;
        };
        
        std::atomic<node*> head;
        std::atomic<node*> tail;
        
        //node* get_tail() {
        //    std::lock_guard<std::mutex> tail_lock(tail_mutex);
        //    return tail;
        //}
        
        node* pop_head() {
            node* const old_head = head.load();
            if (old_head == get_tail())
                return nullptr;
            head.store(old_head->next);
            return old_head;
        }
        
    public:
        lockfree_queue(const lockfree_queue&) = delete;
        lockfree_queue& operator=(const lockfree_queue&) = delete;
        
        lockfree_queue(): head(new node), tail(head.load()) {}
        
        ~lockfree_queue() {
            while (node* const old_head = head.load()) {
                head.store(old_head->next);
                delete old_head;
            }
        }
        
        std::shared_ptr<T> try_pop() {
            node* old_head = pop_head();
            if (!old_head)
                return std::shared_ptr<T>();
            std::shared_ptr<T> res(old_head->data);
            delete old_head;
            return res;
        }
        
        void push(T val) {
            //std::shared_ptr<T> new_data(std::make_shared<T>(val));
            std::unique_ptr<T> new_data(new T(val));
            
            //node* p = new node;
            counted_node_ptr new_next;
            new_next.ptr = new node;
            new_next.external_count = 1;
            
            for (;;) {
                node* const old_tail = tail.load();     // race 1
                T* old_data = nullptr;
                
                //old_tail->data.swap(new_data);
                if (old_tail->data.compare_exchange_strong(
                        old_data, new_data.get())) {    // race 2
                    
                    //old_tail->next = p;
                    //tail.store(p);
                    old_tail->next = new_next;
                    tail.store(new_next.ptr); // other between 1 and 2
                    new_data.release();
                    break;
                }
            }
        }
    };

#### rework with AtomicPtrPlus with counters (too much) ...


#### lockfree recomendations:
+ use _seq_cst for first prototype
+ peek suitable mem release alg
    - del list
    - hazard_pointers
    - ref counters
+ ABA problem
+ identify active wait loops and help other threads


***

<a id="p7"></a>
## Part 7 - parallel program design [home](#home)
paral quicksort: using thread_pool

    template<typename T>
    struct sorter {
        struct chunk {
            std::list<T> data;
            std::promise<std::list<T>> promise;
        };
        
        threadsafe_stack<chunk> chunks;
        std::vector<std::thread> threads;
        unsigned const max_threads;
        std::atomic<bool> end;
        
        sorter()
            : max_threads(std::thread::hardware_concurency() - 1)
            , end(false)
        {}
        
        ~sorter() {
            end = true;
            for (unsigned i=0; i<threads.size(); ++i)
                threads[i].join();
        }
        
        void try_sort_chunk() {
            boost::shared_ptr<chunk> ch = chunks.pop();
            if (ch)
                sort_chunk(ch);
        }
        
        void sort_chunk(boost::shared_ptr<chunk> const& ch) {
            ch->promise.set_value(do_sort(ch->data));
        }
        
        void sort_thread() {
            while (!end) {
                try_sort_chunk();
                std::this_thread::yield();
            }
        }
        
        std::list<T> do_sort(std::list<T>& chunk_data) {
            if (chunk_data.empty())
                return chunk_data;
            
            std::list<T> res;
            res.splice(res.begin(), chunk_data, chunk_data.begin());
            T const& partition_val = *res.begin();
            
            typename std::list<T>::iterator divider
                = std::partition(chunk_data.begin(), chunk_data.end()
                , [&](T const& val){ return val < partition_val; });
            
            chunk new_lower_chunk;
            new_lower_chunk.data.splice(new_lower_chunk.data.end()
                , chunk_data, chunk_data.begin(), divider);
            std::future<std::list<T>> new_lower 
                = new_lower_chunk.promise.get_future();
            chunks.push(std::move(new_lower_chunk));
            if (threads.size() < max_threads)
                threads.push_back(std::thread(&sorter<T>::sort_thread
                                              , this));
                                              
            std::list<T> new_higher(do_sort(chunk_data));
            
            res.splice(res.end(), new_higher);
            
            while(new_lower.wait_for(std::chrono::seconds(0))
                    != std::future_status::ready)
                try_sort_chunk();
            
            res.splice(res.begin(), new_lower.get());
            
            return res;
        }
    };

    template<typename T>
    std::list<T> parall_quick_sort(std::list<T> in) {
        if (in.empty())
            return in;
        sorter<T> s;
        return s.do_sort(in);
    }

#### data partition
- before process
- recursive
- by task type

#### performance issues
- hardware_concurency
- cache ping-pong: cache update between procesors
- false sharing: try keep data for one thread close, so it apears in one cache block (32 or 64 bytes)
- limit-excess (ready threads >> waiting), too much context-switch

issues: concurency, false sharing, locality

matrix mult: 1000 x 1000; 100 procesors:
1. 1 proc calc 10 rows: 10000 el
 - read 1st mat: all 1000000;
   read 2nd mat: 10 rows - 10000 el
 - total 1010000 read
2. 1 proc calc 100 x 100 block: 10000 el
 - read 1st mat: 10 cols - 10000 el
   read 2nd mat: 10 rows - 10000 el
 - total 20000 read: less than in [a]

optimizm:
- try keep data for one thread close
- try keep data for diff thread far
- minimize amount of data read/write

test struct for false sharing:

    struct Data {
        std::mutex m;
        char padding[65536]; // few times more than cache block
        my_data data;
    };


#### exception safety
not safe: can throw, Iter and T - user types  
all ok until create threads

    template<typename Iter, typename T>
    struct accum_block {
        //void operator()(Iter first, Iter last, T& res) {
        //    res = std::accumulate(first, last, res);            // <--
        //}
        
        T operator()(Iter first, Iter last) {
            return std::accumulate(first, last, T());
        }
    };

    template<typename Iter, typename T>
    T parall_accum(Iter first, Iter last, T init) {
        ulong const len = std::distance(first,last);    // <-- ok
        if (!len)
            return init;
        ulong const min_per_thread = 25;
        ulong const max_threads 
            = (len + min_per_thread - 1)/min_per_thread;
        ulong const hw_threads
            = std::thread::hardware_concurency();
        ulong const num_threads = std::min(
            hw_threads ? hw_threads : 2, max_threads);
        ulong const block_size = len/num_threads;
        
        //std::vector<T> res(num_threads);                        // <-- ok
        std::vector<std::future<T>> futures(num_threads - 1);
        std::vector<std::thread> threads(num_threads - 1);      // <-- ok
        
        join_threads joiner(threads);
        
        Iter block_start = first;                               // <-- ok
        for (ulong i=0; i<(num_threads - 1); ++i) {
            Iter block_end = block_start;                       // <--
            std::advance(block_end, block_size);
            
            //threads[i] = std::thread(accum_block<Iter,T>()
            //    , block_start, block_end, std::ref(res[i]));    // <--
            
            std::packaged_task<T(Iter,Iter)> task(accum_block<Iter,T>());
            futures[i] = task.get_future();
            threads[i] = std::thread(std::move(task)
                , block_start, block_end);
            
            block_start = block_end;                            // <--
        }
        
        //accum_block()(block_start, last, res[num_threads - 1]); // <--
        T last_res = accum_block()(block_start, last);
        
        // join_threads joiner instead
        //std::for_each(threads.begin(), threads.end()
        //    , std::mem_fn(&std::thread::join));
        
        //return std::accumulate(res.begin(), res.end(), init);   // <--
        T res = init;
        for (ulong i=0; i<(num_threads - 1); ++i)
            res += futures[i].get();
        res += last_res;
        return res;
    }

    class join_threads {
        std::vector<std::thread>& threads;
        
    public:
        explicit join_threads(std::vector<std::thread>& threads_)
            : threads(threads_)
        {}
        
        ~join_threads() {
            for (ulong i=0; i<threads.size(); ++i)
                if (threads[i].joinable())
                    threads[i].join();
        }
    };

using std::async recursive distrib: exception safe, between threads and main thread

    template<typename Iter, typename T>
    T parall_accum(Iter first, Iter last, T init) {
        ulong const len = std::distance(first,last);
        ulong const max_chunk_size = 25;
        
        if (len <= max_chunk_size) {
            return std::accumulate(first, last, init);
        } else {
            Iter mid = first;
            std::advance(mid, len/2);
            std::future<T> first_half = std::async(parall_accum<Iter,T>
                , first, mid, init);
            T second_half = parall_accum(mid, last, T());
            return first_half.get() + second_half;
        }
    }


#### scaling
Amdal law: P = 1/(Fs + (1 - Fs)/N)  
Fs - % of serial code; P - performance inc  
when thread waiting other threads, end of in/output  
and no other thread ready to work on procesor  
then this procesor is doing nothing useful  

event based responsive GUI:

    std::thread task_thread;
    std::atomic<bool> task_cancel(false);

    void gui_thread() {
        while (true) {
            event_data event = get_event();
            if (event.type == quit)
                break;
            process(event);
        }
    }

    void task() {
        while (!task_complete() && !task_cancel)
            do_next_op();
        if (task_cancel)
            cleanup();
        else
            post_gui_event(task_complete);
    }

    void process(event_data const& event) {
        switch (event.type) {
        case start_task:
            task_cancel = false;
            task_thread = std::thread(task);
            break;
        case stop_task:
            task_cancel = true;
            task_thread.join();
            break;
        case complete_task:
            task_thread.join();
            display_data();
            break;
        }
    }


#### std::for_each
    template<typename Iter, typename Func>
    void parall_for_each(Iter first, Iter last, Func func) {
        ulong const len = std::distance(first,last);
        if (!len) return init;
        ulong const min_perth = 25;
        ulong const max_th = (len + min_perth - 1)/min_perth;
        ulong const hwc = std::thread::hardware_concurency();
        ulong const num = std::min(hwc ? hwc : 2, max_th);
        ulong const block_size = len/num;
        
        std::vector<std::future<void>> futures(num - 1);
        std::vector<std::thread> threads(num - 1);
        join_threads joiner(threads);
        
        Iter block_start = first;
        for (ulong i=0; i<(num - 1); ++i) {
            Iter block_end = block_start;
            std::advance(block_end, block_size);
            std::packaged_task<void(void)> task([=](){
                std::for_each(block_start,block_end, func) });
            futures[i] = task.get_future();
            threads[i] = std::thread(std::move(task));
            block_start = block_end;
        }
        
        std::for_each(block_start,last, func);
        for (ulong i=0; i<(num - 1); ++i)
            futures[i].get();
    }

std::for_each async:

    template<typename Iter, typename Func>
    void parall_for_each(Iter first, Iter last, Func func) {
        ulong const len = std::distance(first,last);
        if (!len) return;
        ulong const max_chunk_size = 25;
        
        if (len < (2*max_chunk_size)) {
            std::for_each(first, last, func);
        } else {
            Iter const mid = first + len/2;
            std::future<void> first_half 
                = std::async(parall_for_each<Iter,Func>
                , first, mid, func);
            parall_for_each(mid, last, func);
            first_half.get();
        }
    }


#### std::find
- std::promise: other threads stop searching if exception
- std::packaged_task: other threads continue searching if exception,  and exception saved

code:

    template<typename Iter, typename Match>
    Iter parall_find(Iter first, Iter last, Match match) {
        struct find_el {
            void operator()(Iter begin, Iter end, Match match
                    , std::promise<Iter>* res, std::atomic<bool>* done) {
                try {
                    for (; (begin!=end) && !done->load(); ++begin) {
                        if (*begin==match) {
                            res->set_value(begin);
                            done->store(true);
                            return;
                        }
                    }
                } catch (...) {
                    try {
                        res->set_exception(std::current_exception());
                        done->store(true);
                    } catch (...) { }
                }
            }
        };
        
        ulong const len = std::distance(first,last);
        if (!len) return init;
        ulong const min_perth = 25;
        ulong const max_th = (len + min_perth - 1)/min_perth;
        ulong const hwc = std::thread::hardware_concurency();
        ulong const num = std::min(hwc ? hwc : 2, max_th);
        ulong const block_size = len/num;
        
        std::promise<Iter> res;
        std::atomic<bool> done(false);
        std::vector<std::thread> threads(num - 1);
        {
            join_threads joiner(threads);
            
            Iter block_start = first;
            for (ulong i=0; i<(num - 1); ++i) {
                Iter block_end = block_start;
                std::advance(block_end, block_size);
                threads[i] = std::thread(find_el()
                    , block_start, block_end, match, &res, &done);
                block_start = block_end;
            }
            
            find_el()(block_start, last, match, &res, &done);
        }
        
        if (!done->load())
            return last;
        return res.get_future().get();
    }

std::find async:

    template<typename Iter, typename Match>
    Iter parall_find_impl(Iter first, Iter last, Match match
            , std::atomic<bool>& done) {
        try {
            ulong const len = std::distance(first,last);
            ulong const min_chunk = 25;
            
            if (len < (2*min_chunk)) {
                for (; (first!=last) && !done.load(); ++first) {
                    if (*first == match) {
                        done = true;
                        return first;
                    }
                }
                return last;
            
            } else {
                Iter mid = first;
                std::advance(mid, len/2);
                std::future<Iter> first_half = std::async(
                    parall_find_impl<Iter,Match>
                    , mid, last, match, std::ref(done));
                Iter const second_half = parall_find_impl(
                    first, mid, match, done);
                return (second_half==mid ? first_half.get() : second_half);
            }
            
        } catch (...) {
            done = true; // if exception this will stop search
            throw;
        }
    }

    template<typename Iter, typename Match>
    Iter parall_find(Iter first, Iter last, Match match) {
        std::atomic<bool> done(false);
        return parall_find_impl(first, last, match, done);
    }


#### std::partial_sum
1 2 3 4 5 6 7 8 9  
{1 2 3} {4 5 6} {7 8 9}  
{1 3 6} {4 9 15} {7 15 24}  
{1 3 6} {10 15 21} {7 15 24}  
1 3 6 10 15 21 28 36 55

    template<typename Iter>
    void parall_partial_sum(Iter first, Iter last) {
        typedef typename Iter::value_type T;
        
        struct process_chunk {
            void operator()(Iter bedin, Iter last
                    , std::future<T>* prev_end_val
                    , std::promise<T>* end_val) {
                try {
                    Iter end = last;
                    ++end;
                    std::partial_sum(begin,end,begin);
                    
                    if (prev_end_val) {
                        T& addend = prev_end_val->get();
                        *last += addend;
                        
                        if (end_val)
                            end_val->set_value(*last);
                        std::for_each(begin,last,[addend](T& val){
                            val+=addend});

                    } else if (end_val) {
                        end_val->set_value(*last);
                    }

                } catch (...) {
                    if (end_val)
                        end_val->set_exception(std::current_exception());
                    else
                        throw;
                }
            }
        };
        
        ulong const len = std::distance(first,last);
        if (!len) return init;
        ulong const min_perth = 25;
        ulong const max_th = (len + min_perth - 1)/min_perth;
        ulong const hwc = std::thread::hardware_concurency();
        ulong const num = std::min(hwc ? hwc : 2, max_th);
        ulong const block_size = len/num;
        
        std::vector<std::promise<T>> end_vals(num - 1);
        std::vector<std::future<T>> prev_end_vals;
        prev_end_vals.reserve(num - 1);

        std::vector<std::thread> threads(num - 1);
        join_threads joiner(threads);
        
        Iter block_start = first;
        for (ulong i=0; i<(num - 1); ++i) {
            Iter block_end = block_start;
            std::advance(block_end, block_size - 1);
            threads[i] = std::thread(process_chunk()
                , block_start, block_end
                , (i!=0) ? &prev_end_vals[i-1] : 0, &end_vals[i]);
            block_start = block_end;
            ++block_start;
            prev_end_vals.push_back(end_vals[i].get_future());
        }
        
        Iter final_el = block_start;
        std::advance(final_el, std::distance(block_start,last) - 1);
        process_chunk()(block_start,final_el
            , (num > 1) ? &prev_end_vals.back() : 0, 0);
    }

because of sync between threads we can not use std::async here

2nd alg:  
1  2  3  4  5  6  7  8  9  
1  3| 5  7  9 11 13 15 17  
1  3  6 10|14 18 22 26 30  
1  3  6 10 15 21 28 36|44  
1  3  6 10 15 21 28 36 55  

in case of massive parallel machine with many many hw threads - 1st alg will be not effective  
but 2nd alg not exception safe

barrier waits for N threads  
if count can be changed (decreased) from any thread

    class barrier {
        //unsigned const count;
        std::atomic<unsigned> count;
        std::atomic<unsigned> spaces;
        std::atomic<unsigned> generation;
        
    public:
        explicit barrier(unsigned n)
            : count(n), spaces(count), generation(0)
        {}
        
        void wait() {
            unsigned const gen = generation.load();
            if (!--spaces) {
                spaces = count.load();
                ++generation;
            } else {
                while (generation.load() == gen)    // spinlock
                    std::this_thread::yield();      // free procesor
            }
        }
        
        void done_waiting() {
            --count;
            if (!--spaces) {
                spaces = count.load();
                ++generation;
            }
        }
    };

    template<typename Iter>
    void parall_partial_sum(Iter first, Iter last) {
        typedef typename Iter::value_type T;
        
        struct process_el {
            void operator()(Iter first, Iter last
                    , std::vector<T>& buff, unsigned i, barrier& b) {
                T& ith = *(first + i);
                bool update = false;
                
                for (unsigned step=0,stride=1;stride<=i;++step,stride*=2) {
                    T const& src = (step%2) ? buff[i] : ith;
                    T& dst = (step%2) ? ith : buff[i];
                    T const& addend = (step%2) 
                        ? buff[i-stride] : *(first+i-stride);
                        
                    dst = src + addend;
                    update = !(step%2);
                    b.wait();
                }
                
                if (update)
                    ith = buff[i];
                b.done_waiting();
            }
        };
        
        ulong const len = std::distance(first,last);
        if (len <= 1) 
            return;
        
        std::vector<T> buff(len);
        barrier b(len);
        std::vector<std::thread> threads(len - 1);
        join_threads joiner(threads);
        
        Iter block_start = first;
        for (ulong i=0; i<(len - 1); ++i) {
            threads[i] = std::thread(process_el(), first,last
                , std::ref(buff), i, std::ref(b));
        }
        
        process_el()(first,last, buff, len-1, b);
    }


***

<a id="p8"></a>
## Part 8 - thread_pool [home](#home)

#### 1st try - the most basic
    class thread_pool {
        std::atomic<bool> done;
        threadsafe_queue<std::function<void()>> work_que;
        std::vector<std::thread> threads;
        join_threads joiner;
        
        void worker_thread() {
            while (!done) {
                std::function<void()> task;
                if (work_que.try_pop(task))
                    task();
                else
                    std::this_thread::yield();
            }
        }
        
    public:
        thread_pool(): done(false), joiner(threads) {
            unsigned const num = std::thread::hardware_concurency();
            
            try {
                for (unsigned i=0; i<num; ++i)
                    threads.push_back(std::thread(
                        &thread_pool::worker_thread, this));
            } catch {
                done = true;
                throw;
            }
        }
        
        ~thread_pool() {
            done = true;
        }
        
        template<typename Func>
        void submit(Func f) {
            work_que.push(std::function<void()>(f));
        }
    };

#### when need return value, works for independent tasks
    class func_wrap {
        struct impl_base {
            virtual void call() = 0;
            virtual ~impl_base() {}
        };
        
        std::unique_ptr<impl_base> impl;
        
        template<typename F>
        struct impl_type: impl_base {
            F f;
            impl_type(F&& f_): f(std::move(f_)) {}
            void call() { f(); }
        };
        
    public:
        template<typename F>
        func_wrap(F&& f): impl(new impl_type<F>(std::move(f))) {}
        void operator()() { impl->call(); }
        func_wrap() = default;
        func_wrap(func_wrap&& other): impl(std::move(other.impl)) {}
        func_wrap& operator=(func_wrap&& other) {
            impl = std::move(other.impl);
            return *this;
        }
        
        func_wrap(func_wrap&) = delete;
        func_wrap(const func_wrap&) = delete;
        func_wrap& operator=(const func_wrap&) = delete;
    };

    class thread_pool {

        threadsafe_queue<func_wrap> work_que;

        template<typename Func>
        std::future<typename std::result_of<Func()>::type>
        submit(Func f) {
            typedef typename std::result_of<Func()>::type res_t;
            std::packaged_task<res_t()> task(std::move(f));
            std::future<res_t> res(task.get_future());
            work_que.push(std::move(task));
            return res;
        }
        
        void run_pending_task() {
            func_wrap task;
            if (work_que.try_pop(task))
                task();
            else
                std::this_thread::yield();
        }
        
        ...
    };

use in parall_accum:

    template<typename Iter, typename T>
    T parall_accum(Iter first, Iter last, T init) {
        ulong const len = std::distance(first,last);    // <-- ok
        if (!len)
            return init;
        ulong const block_size = 25;
        ulong const num_blocks = (len + block_size - 1)/block_size;
        
        std::vector<std::future<T>> futures(num_blocks - 1);
        thread_pool pool;
        
        Iter block_start = first;
        for (ulong i=0; i<(num_threads - 1); ++i) {
            Iter block_end = block_start;
            std::advance(block_end, block_size);
            futures[i] = pool.submit(accum_block<Iter,T>());
            block_start = block_end;
        }
        
        T last_res = accum_block<Iter,T>()(block_start, last);
        T res = init;
        for (ulong i=0; i<(num_threads - 1); ++i)
            res += futures[i].get();
        res += last_res;
        return res;
    }

#### thread_local based quicksort
but there is concurency on work_que

    template<typename T>
    struct sorter {

        thread_pool pool;
        
        std::list<T> do_sort(std::list<T>& chunk_data) {
            if (chunk_data.empty())
                return chunk_data;
            
            std::list<T> res;
            res.splice(res.begin(), chunk_data, chunk_data.begin());
            T const& partition_val = *res.begin();
            
            typename std::list<T>::iterator divider
                = std::partition(chunk_data.begin(), chunk_data.end()
                , [&](T const& val){ return val < partition_val; });
            
            std::list<T> new_lower_chunk;
            new_lower_chunk.splice(new_lower_chunk.end()
                , chunk_data, chunk_data.begin(), divider);
            std::future<std::list<T>> new_lower
                = pool.submit(std::bind(&sorter::do_sort, this
                , std::move(new_lower_chunk)));
                                              
            std::list<T> new_higher(do_sort(chunk_data));

            res.splice(res.end(), new_higher);
            
            while(new_lower.wait_for(std::chrono::seconds(0))
                    == std::future_status::timeout)
                pool.run_pending_task();
            
            res.splice(res.begin(), new_lower.get());
            
            return res;
        }
    };

    template<typename T>
    std::list<T> parall_quick_sort(std::list<T> in) {
        if (in.empty())
            return in;
        sorter<T> s;
        return s.do_sort(in);
    }

#### local queue per thread to fix race on work_que
    class thread_pool {
        threadsafe_queue<func_wrap> pool_work_que;
        
        typedef std::queue<func_wrap> local_queue_t;
        static thread_local std::unique_ptr<local_queue_t> local_work_que;
        
        void worker_thread() {
            local_work_que.reset(new local_queue_t);
            while (!done)
                run_pending_task
        }
        
    public:
        template<typename Func>
        std::future<typename std::result_of<Func()>::type>
        submit(Func f) {
            typedef typename std::result_of<Func()>::type res_t;
            std::packaged_task<res_t()> task(f); //std::move?
            std::future<res_t> res(task.get_future());
            
            if (local_work_que)
                local_work_que.push(std::move(task));
            else
                pool_work_que.push(std::move(task));
            return res;
        }
        
        void run_pending_task() {
            func_wrap task;
            if (local_work_que && !local_work_que->empty()) {
                task = std::move(local_work_que->front());
                local_work_que->pop();
                task();
            
            } else if (pool_work_que.try_pop(task)) {
                task()
            
            } else {
                std::this_thread::yield();
            }
        }
    };

#### stealing tasks from other threads
when own que is empty it stealing tasks  
protected by mutex as stealing is rare event  
thread get own tasks from front, stealing from back

    class work_stealing_queue {
        typedef func_wrap T;
        std::deque<T> que;
        mutable std::mutex m;
        
    public:
        work_stealing_queue() {}
        work_stealing_queue(const work_stealing_queue&) = delete;
        work_stealing_queue& operator=(const work_stealing_queue&) = delete;
        
        void push(T data) {
            std::lock_guard<std::mutex> g(m);
            que.push_front(std::move(data));
        }
            
        bool empty() {
            std::lock_guard<std::mutex> g(m);
            return que.empty();
        }
        
        bool try_pop(T& res) {
            std::lock_guard<std::mutex> g(m);
            if (que.empty())
                return false;
            res = std::move(que.front());
            que.pop_front();
            return true;
        }
        
        bool try_steal(T& res) {
            std::lock_guard<std::mutex> g(m);
            if (que.empty())
                return false;
            res = std::move(que.back());
            que.pop_back();
            return true;
        }
    };


#### thread_pool with stealing
    class thread_pool {
        typedef func_wrap task_t;

        std::atomic<bool> done;
        threadsafe_queue<task_t> pool_work_que;
        std::vector<std::unique_ptr<work_stealing_queue>> ques;
        std::vector<std::thread> threads;
        join_threads joiner;
        
        static thread_local work_stealing_queue* local_work_que;
        static thread_local unsigned id;
        
        void worker_thread(unsigned my_index) {
            id = my_index;
            local_work_que = ques[id].get();
            while (!done)
                run_pending_task();
        }
        
        bool pop_local(task_t& task) {
            return local_work_que && local_work_que->try_pop(task);
        }
        
        bool pop_pool(task_t& task) {
            return pool_work_que && pool_work_que.try_pop(task);
        }
        
        bool pop_steal(task_t& task) {
            for (unsigned i=0; i<ques.size(); ++i) {
                // start from next pos
                unsigned const index = (id+i+1)%ques.size();
                if (ques[index]->try_steal(task))
                    return true;
            }
            return false;
        }
        
    public:
        thread_pool(): done(false), joiner(threads) {
            unsigned const num = std::thread::hardware_concurency();
            
            try {
                for (unsigned i=0; i<num; ++i) {
                    ques.push_back(std::unique_ptr<work_stealing_queue>(
                        new work_stealing_queue));
                    threads.push_back(std::thread(
                        &thread_pool::worker_thread, this, i));
                }
            } catch (...) {
                done = true;
                throw;
            }
        }
        
        ~thread_pool() {
            done = true;
        }
        
        template<typename Func>
        std::future<typename std::result_of<Func()>::type>
        submit(Func f) {
            typedef typename std::result_of<Func()>::type res_t;
            std::packaged_task<res_t()> task(f); //std::move?
            std::future<res_t> res(task.get_future());
            
            if (local_work_que)
                local_work_que->push(std::move(task));
            else
                pool_work_que.push(std::move(task));
            return res;
        }
        
        void run_pending_task() {
            task_t task;
            if (pop_local(task) || pop_pool(task) || pop_steal(task)) {
                task();
            } else {
                std::this_thread::yield();
            }
        }
    };


#### thread interrupt
    class interrupt_flag {
    public:
        void set();
        bool is_set() const;
    };

    thread_local interrupt_flag this_thread_interrupt_flag;

    class interrupt_thread {
        std::thread internal_thread;
        interrupt_flag* flag;
        
    public:
        template<typename Func>
        interrupt_thread(Func f) {
            std::promise<interrupt_flag*> p;
            internal_thread = std::thread([f,&p](){
                p.set_value(&this_thread_interrupt_flag);
                f();
            });
            flag = p.get_future().get();
        }
        
        void interrupt() {
            if (flag)
                flag->set();
        }
    };

    void interrupt_point() {
        if (this_thread_interrupt_flag.is_set())
            throw thread_interrupt_excpt();
    }

    void foo() {
        while (!done) {
            interrupt_point();
            process();
        }
    }

#### interrupt waiting condition
beter interrupt thread when it blocked and waiting for smth,  
but in this moment thread not runing  
so it can not call interrupt_point  

bad:

    void interrupt_wait(std::condition_variable& cv
            , std::unique_lock<std::mutex>& lk) {
        interrupt_point();
        this_thread_interrupt_flag.set_condition(cv);
        // here race
        cv.wait(lk); // can throw
        this_thread_interrupt_flag.clear_condition();
        interrupt_point();
    }

with timeout:

    class interrupt_flag {
        std::atomic<bool> flag;
        std::condition_variable* cond;
        std::condition_variable_any* cond_any;
        std::mutex m;
        
        template<typename Lock>
        struct custom_lock {
            interrupt_flag* self;
            Lock& lk;
            
            custom_lock(interrupt_flag* self_
                    , std::condition_variable_any& cond_
                    , Lock& lk_)
                : self(self_), lk(lk_)
            {
                self->m.lock();
                self->cond_any = cond_;
            }
            
            void lock() {
                std::lock(self->m, lk);
            }
            
            void unlock() {
                lk.unlock();
                self->m.unlock();
            }
            
            ~custom_lock() {
                self->cond_any = 0;
                self->m.unlock();
            }
        };
            
    public:
        interrupt_flag(): cond(0), cond_any(0) {}
        
        void set() {
            flag.store(true, _relaxed);
            std::lock_guard<std::mutex> g(m);
            if (cond)
                cond->notify_all();
            else if (cond_any)
                cond_any->notify_all();
        }
        
        template<typename Lock>
        void wait(std::condition_variable_any& cv, Lock& lk) {
            custom_lock<Lock> cl(this, cv, lk);
            interrupt_point();
            cv.wait(cl);
            interrupt_point();
        }
        
        bool is_set() const {
            return flag.load(_relaxed);
        }
        
        void set_condition(std::condition_variable& cv) {
            std::lock_guard<std::mutex> g(m);
            cond = &cv;
        }
        
        void clear_condition() {
            std::lock_guard<std::mutex> g(m);
            cond = 0;
        }
        
        struct destr {
            ~destr() { 
                this_thread_interrupt_flag.clear_condition(); 
            }
        };
    };

    template<typename Pred>
    void interrupt_wait(std::condition_variable& cv
            , std::unique_lock<std::mutex>& lk
            , Pred pred) {
        interrupt_point();
        this_thread_interrupt_flag.set_condition(cv);
        interrupt_flag::destr guard;
        while (!this_thread_interrupt_flag.is_set() && !pred())
            cv.wait_for(lk, std::chrono::milliseconds(1));
        interrupt_point();
    }

for any:

    template<typename Lock>
    void interrupt_wait(std::condition_variable_any& cv
            , Lock& lk) {
        this_thread_interrupt_flag.wait(cv,lk);
    }

future interrupt:

    template<typename T>
    void interrupt_wait(std::future<T>& f) {
        while (!this_thread_interrupt_flag.is_set()) {
            if (f.wait_for(lk, std::chrono::milliseconds(1))
                    == std::future::ready)
                break;
        }
        interrupt_point();
    }

interrupt handle:

    internal_thread = std::thread([f,&p](){
        p.set_value(&this_thread_interrupt_flag);
        try { f(); } catch (thread_interrupt_excpt const&) {}
    });


#### interrupt bg threads
    std::vector<interrupt_thread> bg_threads;

    void bg_thread(int disk_id) {
        while (true) {
            interrupt_point();
            fs_change fsc = get_fs_changes(disk_id);
            if (fsc.has_chages())
                update_index(fsc);
        }
    }

    void start_bg_process() {
        bg_threads.push_back(interrupt_thread(bg_thread, disk_1));
        bg_threads.push_back(interrupt_thread(bg_thread, disk_2));
    }

    int main() {
        start_bg_process();
        process_gui_until_exit();
        std::unique_lock<std::mutex> lk(m);
        for (unsigned i=0; i<bg_threads.size(); ++i)
            bg_threads[i].interrupt(); // mb interrupt_threads
        for (unsigned i=0; i<bg_threads.size(); ++i)
            bg_threads[i].join(); // mb join_threads
    }


***

<a id="p9"></a>
## Part 9 - test and debug [home](#home)
errors:
1. unwanted blocking
 - deadlock
 - active blocking
 - block waiting for in/output
2. races
 - data race
 - invariant break
 - wrong life time

error searching:
 - what data need to protect from simult access
 - how it is protected
 - where other threads can be runing
 - what mutexes are locked in this thread
 - what mutexes are locked in other threads
 - what order restrictions exist in this and other threads, how this restrictions are suported
 - does data loaded by this thread still valid, could some other thread change this data
 - what happens if other thread can change your data, how to guarantee this never happens

write testable code:
 - all func and class responsibility stricrlt defined
 - func is short and solve one task
 - test can control environment of tested code
 - tested code located in one place
 - before writing code think how it will be tested

test cases for threadsafe_queue:
 - 1 call push or pop
 - 1st call push for empty que, 2nd call pop
 - several call push for empty que
 - several call push for non empty que
 - several call pop for empty que
 - several call pop for non empty que
 - several call pop for non empty que, but not enough for all
 - several call push for empty que, 1 call pop
 - several call push for non empty que, 1 call pop
 - several call push for empty que, several call pop
 - several call push for non empty que, several call pop

additional:
 - several: 3,4,1024
 - is it enough procesors for each thread
 - what pocessor architecture
 - how do planing for loops in tests

test method - if can divide on:
 - per thread code - test as simple serial code
 - sync code - guarantee only 1 thread access data

highload tests: many times, long time
 - test on 1 proc machine
 - test on diff architecture: can support diff memory_order_, is_lock_free

special sw:
 - combinatoric imitation tests
 - special test lib

performance
 - scaling
 - test on 1 procesor machine, to as many as you can
 - can differ depend on alg: on 4 proc and large scale proc

tests struct:
 - global setup
 - per thread setup
 - run code
 - finish code (check state)

example:

    void test_push_pop_empty_que() {
        threadsafe_queue<int> que;
        
        std::promise<void> go, push_ready, pop_ready;
        std::shared_future<void> ready(go.get_future());
        std::future<void> push_done;
        std::future<int> pop_done;
        
        try {
            push_done = std::async(std::launch::async
                , [&que, ready, &push_ready](){
                    push_ready.set_value();
                    ready.wait();
                    que.push(42);
                });
            
            pop_done = std::async(std::launch::async
                , [&que, ready, &pop_ready](){
                    pop_ready.set_value();
                    ready.wait();
                    return que.pop();
                });
                
            push_ready.get_future().wait();
            pop_ready.get_future().wait();
            go.set_value();
            
            push_done.get();
            assert(pop_done.get() == 42);
            assert(que.empty());
        } catch (...) {
            go.set_value();
            throw;
        }
    }
