<h2>《Effective STL》 :books: </h2> 

> Scott Meyers 著    电子工业出版社

```c++
31.了解各种与排序相关的选择。
   (1) 如果需要对 vector、string、deque 或者数组中的元素执行一次完全排序，可以使用 sort 或者 stable_sort。
   (2) 如果有一个 vector、string、deque 或者数组，并且只需要对等价性最前面的 n 个元素进行排序，
可以使用 partial_sort。
       bool qualityCompare(const Widget& lhs, const Widget& rhs) {
           // 返回 lhs 的质量是否好于 rhs 的质量的结果
       }
       ...
       // 将质量最好的 20 个元素顺序放在 widgets 的前 20 个位置上
       partial_sort(widgets.begin(), widgets.begin() + 20, widgets.end(), qualityCompare);
    (3) 如果有一个 vector、string、deque 或者数组，并且需要找到第 n 个位置上的元素，或者需要找到等价性最前面
的 n 个元素进行排序， 可以使用 nth_element。
       vector<Widget>::iterator begin(widgets.begin()); 
       vector<Widget>::iterator end(widgets.end()); 
       vector<Widget>::iterator goalPosition;       // 定位感兴趣的元素
       goalPosition = begin + widgets.size() / 2;   // 找到具有中间质量级别的 Widget
       nth_element(begin, goalPosition, end, qualityCompare);
    (4) 如果需要将一个标准序列容器中的元素按照是否满足某个特定的条件区分开来，
可以使用 partition 和 stable_partition。
       bool hasAcceptableQuality(const Widget& w) {
          // 判断 w 的质量值是否为 2 或者更好
       }
       // 将满足 hasAcceptableQuality 的所有元素移到前部，然后返回一个迭代器，指向第一个不满足条件的 Widget
       vector<Widget>::iterator goodEnd = partition(widgets.begin(), widgets.end(), hasAcceptableQuality);
    (5) 如果你的数据在一个 list 中，仍然可以直接调用 partition 和 stable_partition 算法；可以用 list::sort 来替代
sort 和 stable_sort 算法。如果你需要获得 partial_sort 或 nth_element 算法的效果，可以通过间接途径完成。

32.如果确实需要删除元素，则需要在 remove 这一类算法之后调用 erase。
    vector<int> v;
    ...
    v.erase(remove(v.begin(), v.end(), 99), v.end());     // 真正删除所有值等于 99 的元素
    count << v.size();

33.对包含指针的容器使用 remove 这一类算法时要特别小心。

34.了解哪些算法要求使用排序的区间作为参数。
   (1) 用于查找的算法 binary_search、lower_bound、upper_bound 和 equal_range 
要求排序的区间，因为它们用二分法查找数据。
   (2) set_union、set_intersection、set_difference 和 set_symmetric_difference 
这 4 个算法提供了线性时间效率的集合操作。
   (3) merge 和 inplace_merge 实现了合并和排序的联合操作。
   (4) unique 和 unique_copy 删除区间中的重复值。
  
35.通过 mismatch 或 lexicographical_compare 实现简单的忽略大小写的字符串比较。
   bool ciCharLess(char c1, char c2) {   // 返回忽略大小写的情况下，c1 是否在 c2 之前
      return tolower(static_cast<unsigned char>(c1)) < tolower(static_cast<unsigned char>(c2));
   }
   bool ciStringCompare(const string &s1, const string &s2) {
      return lexicographical_compare(s1.begin(), s1.end(), s1.begin(), s1.end(), ciCharLess);
   }

36.理解 copy_if 算法的正确实现。
   template<typename InputIterator, typename OutputIterator, typename Predicate>
   OutputIterator copy_if(InputIterator begin, InputIterator end, OutputIterator destBegin, Predicate p) {
      while (begin != end) {
         if (p(*begin)) *destBegin++ = *begin;
         ++begin;
      }
      return destBegin;
   }

37.使用 accumulate 或者 for_each 进行区间统计。
   (1) 计算一个区间中数值的乘积：
     vector<float> vf;      // 创建一个 float 容器并加入一些数据
     ...
     // 对 vf 中的每个元素调用 multiplies<float>，并把结果赋给 product，初始值为 1.0f
     float product = accumulate(vf.begin(), vf.end(), 1.0f, multiplies<float>()); 
   (2) 计算出一个区间中的所有点的平均值：
     struct Point {
        Ponit(double initX, duoble initY): x(initX), y(initY) {}
        double x, y;
     };
     class PointAverage: public unary_function<Point, void> {
        public:
           PointAverage():xSum(0), ySum(0), numPoints(0) {}
           void operator()(const Point& p) {
              ++numPoints;
              xSum += p.x;
              ySum += p.y;
           }
           Point result() const {
              return Point(xSum/numPoints, ySum/numPoints);
           }
        private:
           size_t numPoints;
           double xSum;
           double ySum;
     };
     list<Point> lp;
     ...
     Point avg = for_each(lp.begin(), lp.end(), PointAverage()).result();
     
38.遵循按值传递的原则来设计函数子类。
   "Bridge Pattern" 或 "Pimpl Idiom":
      template<typename T>
      class BPFCImpl: public unary_function<T, void> {  // BPFC (Big Polymorphic Functor Class) 实现类
         private:
            Widget w;      // 原来 BPFC 中的数据
            int x;
            ...
            virtual ~BPFCImpl();   // 多态类析构函数
            virtual void operator() (const T& val) const;
            friend class BPFC<T>;
      };
      template<typename T>
      class BPFC: public unary_function<T, void> {  // 新的 BPFC 类：短小，单态
         private:
            BPFCImpl<T> *pImpl;      // BPFC 唯一的数据成员
         public:
            void operator() (const T& val) const {   // 非虚函数将调用转到 BPFCImpl 中
               pImpl->operator()(val);
            }
            ...
      };
      
39.确保判别式是 "纯函数"。
   (1) 判别式 (predicate)：返回值为 bool 类型 (或者可以隐式地转换为 bool 类型) 的函数.
   (2) 纯函数 (pure function)：返回值仅仅依赖于其参数的函数。
   (3) 判别式类 (predicate class)：函数子类，它的 operator() 函数是一个判别式，返回 true 或 false。 

40.若一个类是函数子，则应使它可配接。
   4 个标准的函数配接器 (not1、not2、bind1st 和 bind2nd) 都要求一些特殊的类型定义。这些特殊的类型定义是：
argument_type、first_argument_type、second_argument_type 及 result_type。提供这些类型定义的最简便方法是
让函数子从一个基结构继承。如果函数子类的 operator() 只有一个实参，它应该从 std::unary_function 继承；
如果函数子类的 operator() 有两个实参，它应该从 std::binary_function 继承。
     template<typename T>
     class MeetsThreshold: public std::unary_function<Widget, bool> {
        private:
           const T threshold;
        public:
           MeetsThreshold(const T& threshold);
           bool operator()(const Widget&) const;
           ...
     };
     struct WidgetNameCompare: public std::binary_function<Widget, Widget, bool> {
        bool operator()(const Widget& lhs, const Widget& rhs) const;
     }
     list<Widget> widgets;
     ...
     list<Widget>::reverse_iterator i1 =         // 找到最后一个不符合阈值 10 的 Widget
          find_if(widgets.rbegin(), widgets.rend(), not1(MeetsThreshold<int>(10)));
     Widget w(...);
     list<Widget>::iterator i2 =  // 找到按 WidgetNameCompare 定义的规则排序时，在 w 之前的第一个 Widget 对象
          find_if(widgets.begin(), widgets.end(), bind2nd(WidgetNameCompare(), w));
```
